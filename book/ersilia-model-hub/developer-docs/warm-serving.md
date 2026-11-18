---
description: Keep a served model loaded in memory between batches, instead of starting it again for every batch.
---

# Warm serving

## Why

`ersilia run` splits the input into batches and sends them to the served model one at a time. Without **warm serving**, every batch starts `run.sh`: a new Python process that imports the libraries and loads the checkpoints before it predicts anything. For many models, this start-up takes longer than the prediction itself. A model that needs 3 seconds to load, run on 100 batches, spends 5 minutes only loading.

With warm serving, the model is loaded **once**, when the server starts, in a long-lived worker process. Every batch goes to that worker.

## How a model opts in

The model's `main.py` defines two functions, as in the current [model template](../model-contribution/model-template.md#the-model-folder):

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

* `main.py` defines both `load_tool` and `run_tool`.
* `run.sh` is the default one-liner: `python $1/code/main.py $2 $3`.

Any other model keeps the previous behavior: `run.sh` once per batch. No model changes behavior until its `main.py` is adapted.

### Example: eos9o72

Before, the script ran top to bottom on every batch and loaded the CheMeleon checkpoint each time:

```python
input_file = sys.argv[1]
output_file = sys.argv[2]
_, smiles_list = read_smiles(input_file)
chemeleon_fingerprint = CheMeleonFingerprint()
outputs = chemeleon_fingerprint(smiles_list)
header = [f"feat_{str(i).zfill(4)}" for i in range(2048)]
write_out(outputs, header, output_file, np.float32)
```

After, the same code is split in two. Imports, `run.sh` and outputs do not change:

```python
def load_tool():
    return CheMeleonFingerprint()

def run_tool(model, input_file, output_file):
    _, smiles_list = read_smiles(input_file)
    outputs = model(smiles_list)
    header = [f"feat_{str(i).zfill(4)}" for i in range(2048)]
    write_out(outputs, header, output_file, np.float32)

if __name__ == "__main__":
    run_tool(load_tool(), sys.argv[1], sys.argv[2])
```

With 100 molecules in batches of 20, the run went from 14.5 s to 1.9 s, with identical output.

### Rules for `run_tool`

* Do not load anything again inside `run_tool`.
* Do not keep state between batches: each batch must give the same result, whatever ran before it.
* Check that `examples/run_output.csv` is unchanged after the migration.

## How it works

Everything lives in [ersilia-pack](https://github.com/ersilia-os/ersilia-pack):

* `templates/warm.py` decides, for each batch, whether the worker can be used, and starts the worker when the server starts. Loading runs in the background, so the server is up straight away; batches wait until the model is ready.
* `templates/warm_worker.py` is the worker. It runs in the model's own Python environment, imports `main.py`, calls `load_tool()` once, and then calls `run_tool()` for each batch. The two processes exchange one JSON message per line; anything the model prints goes to the log.

## Failure handling

A worker problem never loses a batch or changes a result:

| What happens | What Ersilia does |
| --- | --- |
| `load_tool()` raises an error | The model stays on `run.sh` for the rest of the server's life |
| The worker dies between or during a batch | That batch is rerun with `run.sh`; the next batch starts a new worker |
| `run_tool()` raises an error | The batch fails, exactly as a failing `run.sh` does |
| `ersilia close` | The worker is stopped; no process is left behind |

Set `ERSILIA_WARM=0` to turn warm serving off. Every batch then goes through `run.sh`.

## Monitoring

`/models/status` reports a `warm` block with `active`, `loaded`, `loadSeconds`, `pid` and the worker's memory (`memoryAfterLoadMb`, `memoryMb`, `peakMemoryMb`). The server's own `memoryUsage` does not include the worker.

## Results

We measured end to end with the Ersilia CLI (`fetch → serve → run ×3 → close`) on nine models of different task types. Values are the median of three runs, 100 molecules in batches of 20 (eos57bx: 6 molecules in batches of 2).

| Model | Before (s) | Warm (s) | Speed-up | Load time (once) |
| --- | --- | --- | --- | --- |
| eos4n4d | 23.0 | 2.4 | 9.5× | 3.14 s |
| eos9o72 | 14.5 | 1.9 | 7.8× | 2.24 s |
| eos96f4 | 9.5 | 2.1 | 4.6× | 1.05 s |
| eos57bx | 568.5 | 203.3 | 2.8× | 3.37 s |
| eos45di | 16.9 | 6.2 | 2.7× | 1.79 s |
| eos18ie | 43.2 | 18.4 | 2.4× | 4.48 s |
| eos12x7 | 1.5 | 0.8 | 1.9× | 0.09 s |
| eos7jio | 1.7 | 1.0 | 1.7× | 0.09 s |
| eos2mrz | 0.9 | 0.8 | 1.1× | 0.02 s |

On 1,000 molecules, eos4n4d went from 47.5 s to 6.8 s. Outputs were identical for all nine models, and models that were not adapted were unaffected.

The gain grows with the load time: light models gain little.

## Trade-off: memory

A served model now keeps its memory while idle. For eos4n4d, idle memory goes from about 120 MB to 2.2 GB, which is about what one batch used at its peak before. Peak memory during a run is about the same. Serving many models on the same machine therefore needs more memory than before.

## Current limitations

* One worker per served model: batches are processed one at a time.
* New models get the `load_tool` / `run_tool` structure from the template. Existing models are adapted one at a time.
