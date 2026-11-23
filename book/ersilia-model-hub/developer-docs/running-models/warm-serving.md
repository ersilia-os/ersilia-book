---
description: >-
  Keep a served model loaded in memory between batches, instead of starting it
  again for every batch.
---

# Warm serving

## Why warm serving

`ersilia run` splits the input into batches and sends them to the served model one at a time. Without **warm serving**, every batch starts `run.sh`: a new Python process that imports the libraries and loads the checkpoints before it predicts anything. For many models, this start-up takes longer than the prediction itself. <mark style="color:$warning;">A model that needs 3 seconds to load, run on 100 batches, spends 5 minutes only loading.</mark>

With warm serving, the model is loaded **once**, when the server starts, in a long-lived worker process. Every batch goes to that worker.

## Usage

The model's `main.py` defines two functions, as in the current [model template](../../model-contribution/model-template.md):

```python
def load_tool():
    # slow, one-time setup: imports, checkpoints, building the model
    return model

def run_tool(model, input_file, output_file):
    # one batch: read input_file, predict, write output_file
    ...

if __name__ == "__main__":
    run_tool(load_tool(), sys.argv[1], sys.argv[2])   # run.sh keeps working
```

The server uses the worker only when both conditions are met:

* `main.py` defines both `load_tool` and `run_tool`. If those functions are imported from another file or defined inside a class the server won't see them.
* `run.sh` is the default one-liner: `python $1/code/main.py $2 $3`.

Any other model keeps the previous behavior: `run.sh` once per batch. No model changes behavior until its `main.py` is adapted.

### Rules for `run_tool`

* Do not load anything again inside `run_tool`.
* Do not keep state between batches: each batch must give the same result, whatever ran before it.
* Keep code that reads `sys.argv` inside the `if __name__ == "__main__":` block. Top-level code runs only once, when the worker loads the model.
* Check that `examples/run_output.csv` is unchanged after the migration.

## How it works

Everything lives in [ersilia-pack](https://github.com/ersilia-os/ersilia-pack):

* `templates/warm.py` decides, for each batch, whether the worker can be used, and starts the worker when the server starts. Loading runs in the background, so the server is up straight away; batches wait until the model is ready.
* `templates/warm_worker.py` is the worker. It runs in the model's own Python environment, imports `main.py`, calls `load_tool()` once, and then calls `run_tool()` for each batch. The two processes exchange one JSON message per line; anything the model prints goes to the log.

## Failure handling

A worker problem never loses a batch or changes a result:

| What happens                              | What Ersilia does                                                     |
| ----------------------------------------- | --------------------------------------------------------------------- |
| `load_tool()` raises an error             | The model stays on `run.sh` for the rest of the server's life         |
| The worker dies between or during a batch | That batch is rerun with `run.sh`; the next batch starts a new worker |
| `run_tool()` raises an error              | The batch fails, exactly as a failing `run.sh` does                   |
| `ersilia close`                           | The worker is stopped; no process is left behind                      |

Set `ERSILIA_WARM=0` to turn warm serving off. Every batch then goes through `run.sh`.

### Monitoring

`/models/status` reports a `warm` block with `active`, `loaded`, `loadSeconds`, `pid` and the worker's memory (`memoryAfterLoadMb`, `memoryMb`, `peakMemoryMb`). The server's own `memoryUsage` does not include the worker.

## Limitations

A warm model keeps its memory while idle, roughly what one batch used at its peak, so serving many models on the same machine **needs more memory** than before: for example, eos4n4d stays at about 2.2 GB instead of 120 MB. Each served model has a single worker, so batches are processed one at a time.

{% hint style="info" %}
In our tests on nine models, warm serving made runs up to 9.5 times faster, with identical outputs. The gain grows with the time a model takes to load, so light models gain little
{% endhint %}
