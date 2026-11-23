# Python API

The Python API lets you use the Ersilia Model Hub from scripts and notebooks. It has two classes, `Model` and `Catalog`, which mirror the [command line interface](command-line-interface.md) (CLI) with the same options, defaults, and rules.

## Quick start

```python
from ersilia.api import Model

model = Model("eos3b5e")
model.fetch()
model.serve()
df = model.run(["CCO", "c1ccccc1"])
model.close()
```

Once a model is fetched, a `with` block serves it at the start and closes it at the end, even if an error happens inside:

```python
with Model("eos3b5e") as model:
    df = model.run("input.csv")
```

## Model

```python
Model(model, verbose=False)
```

<table><thead><tr><th width="120.25">Parameter</th><th width="147.3046875">Default</th><th>Description</th></tr></thead><tbody><tr><td><code>model</code></td><td>Required</td><td>The model's identifier, such as <code>"eos3b5e"</code>, or its slug.</td></tr><tr><td><code>verbose</code></td><td><code>False</code></td><td>Print the same progress messages as the CLI.</td></tr></tbody></table>

A `Model` has two attributes, `model_id` and `slug`. If the model isn't in the Ersilia Model Hub, the API raises `InvalidModelIdentifierError`, with suggestions if you made a typo.

### fetch

```python
model.fetch(from_dir=None, from_github=False, from_s3=False, from_hosted=None, version=None)
```

Downloads the model and sets it up on your machine, like `ersilia fetch`. By default, it fetches the model's Docker image from Docker Hub, so Docker must be installed and running. Specify another source if preferred.

<table><thead><tr><th width="144.7421875">Parameter</th><th width="131.1015625">Default</th><th>Description</th></tr></thead><tbody><tr><td><code>from_dir</code></td><td>Not set</td><td>Fetch from a local copy of the model repository, given its path.</td></tr><tr><td><code>from_github</code></td><td><code>False</code></td><td>Fetch from the model's GitHub repository.</td></tr><tr><td><code>from_s3</code></td><td><code>False</code></td><td>Fetch from Ersilia's S3 storage.</td></tr><tr><td><code>from_hosted</code></td><td>Not set</td><td>Connect to a model hosted elsewhere, given its URL.</td></tr><tr><td><code>version</code></td><td>Latest</td><td>The Docker image version to fetch from Docker Hub. Ignored for other sources.</td></tr></tbody></table>

Returns `True` if the model was fetched now, or `False` if it was already on your machine. Raises `ModelFetchError` if the fetch fails.

To check whether a model is already fetched, use `model.is_fetched()`, which returns `True` or `False`.

### serve

```python
model.serve(port=None, enable_cache=False, max_cache_memory_frac=None, read_store=False, write_store=False, access=None, track=False, tracking_use_case="local")
```

Starts the model as a local API, like `ersilia serve`. The model must be fetched first.

<table data-search="false"><thead><tr><th width="213.19140625">Parameter</th><th width="187.91015625">Default</th><th>Description</th></tr></thead><tbody><tr><td><code>port</code></td><td>Chosen automatically</td><td>The port for the local API.</td></tr><tr><td><code>enable_cache</code></td><td><code>False</code></td><td>Cache predictions in Redis, which needs Docker running. See <a href="../running-models/results-caching.md">Results caching</a>.</td></tr><tr><td><code>max_cache_memory_frac</code></td><td><code>0.3</code></td><td>The maximum fraction of system memory the cache can use, between 0 and 1.</td></tr><tr><td><code>read_store</code></td><td><code>False</code></td><td>Read previously computed predictions from the <code>isaura</code> precalculation store. Requires <code>isaura</code>.</td></tr><tr><td><code>write_store</code></td><td><code>False</code></td><td>Write predictions to the <code>isaura</code> store. Requires <code>isaura</code> and <code>access</code>.</td></tr><tr><td><code>access</code></td><td>Not set</td><td>The visibility of predictions written to the store: <code>"public"</code> or <code>"private"</code>.</td></tr><tr><td><code>track</code></td><td><code>False</code></td><td>Track the model's runs and send them to Ersilia's S3 tracking bucket. Uploading needs Ersilia's AWS credentials.</td></tr><tr><td><code>tracking_use_case</code></td><td><code>"local"</code></td><td>With <code>track</code>: <code>"local"</code>, <code>"self-service"</code>, <code>"hosted"</code>, or <code>"test"</code>.</td></tr></tbody></table>

Returns a dictionary describing how the model is served, including its `url`, the `docs` URL of its API, and its `apis`. If the model is already served, `serve` doesn't start it again and returns the same details.

Raises `ModelNotAvailableLocallyError` if the model isn't fetched, and `SessionBusyError` if another model is already served in the same session. See [Sessions](python-api.md#sessions).

### run

```python
model.run(input, output=None, batch_size=100)
```

Runs the served model, like `ersilia run`.

<table><thead><tr><th width="164.4375">Parameter</th><th width="163.65625">Default</th><th>Description</th></tr></thead><tbody><tr><td><code>input</code></td><td>Required</td><td>A list of inputs, such as SMILES strings, or the path to a CSV file with a header and one input per line. If the file has several columns, Ersilia uses the one named <code>smiles</code> or <code>input</code>. Dataframes are not accepted, convert their columns to list.</td></tr><tr><td><code>output</code></td><td>Not set</td><td>A <code>.csv</code> or <code>.h5</code> file to save the predictions to.</td></tr><tr><td><code>batch_size</code></td><td><code>100</code></td><td>The number of inputs sent to the model in each batch.</td></tr></tbody></table>

Returns the predictions as a pandas DataFrame or, if you give `output`, the path to the output file. Raises `ModelNotServedError` if the model isn't served, and `InvalidOptionError` if there's a problem with the input or output file.

### info

```python
model.info(output=None)
```

Returns the model's information as a dictionary, like `ersilia info`. The model must be fetched, but unlike the CLI, it doesn't need to be served. To also save the information, give `output` a `.json` or `.csv` path.

### example

```python
model.example(n_samples=5, mode="random", output=None)
```

Returns example inputs for the model as a list, like `ersilia example`. The model must be fetched.

<table><thead><tr><th width="139.46875">Parameter</th><th width="141.3671875">Default</th><th>Description</th></tr></thead><tbody><tr><td><code>n_samples</code></td><td><code>5</code></td><td>The number of examples. Ignored in <code>"curated"</code> mode.</td></tr><tr><td><code>mode</code></td><td><code>"random"</code></td><td><code>"random"</code>, from a list of molecules included with Ersilia; <code>"deterministic"</code>, always the same molecules, for any model; or <code>"curated"</code>, the model's own example inputs.</td></tr><tr><td><code>output</code></td><td>Not set</td><td>A <code>.csv</code> file to also save the examples to.</td></tr></tbody></table>

### close

```python
model.close()
```

Stops the served model, like `ersilia close`. Returns `True` if the model was closed, or `False` if no model was served, so there was nothing to close. Raises `ModelNotServedError` if a different model is served in the session.

### delete

```python
model.delete()
```

Removes the model from your machine, like `ersilia delete`. Unlike the CLI, it never closes a served model for you: if the model is served, it raises `ModelNotDeletableError`, so call `close()` first. Raises `ModelNotAvailableLocallyError` if the model isn't fetched.

## Catalog

```python
from ersilia.api import Catalog

catalog = Catalog()
hub = catalog.hub(task="Annotation")
local = catalog.local()
card = catalog.card("eos3b5e")
```

`Catalog(verbose=False)` lists models, like `ersilia catalog`. It has three methods:

| Method                                      | Returns    | Description                                                               |
| ------------------------------------------- | ---------- | ------------------------------------------------------------------------- |
| `hub(more=False, task=None, output=None)`   | DataFrame  | All models in the Ersilia Model Hub, one row per model.                   |
| `local(more=False, task=None, output=None)` | DataFrame  | The models fetched on your machine. Empty if there are none.              |
| `card(model, output=None)`                  | Dictionary | The full model card of a model, which doesn't need to be on your machine. |

In `hub` and `local`, `more=True` adds columns, `task` keeps only `"Annotation"`, `"Representation"`, or `"Sampling"` models, and `output` also saves the table to a `.csv` or `.json` file. In `card`, `output` saves the card to a `.json` or `.csv` file.

## Handle errors

Every error the Python API raises is a subclass of `ErsiliaError`, so you can catch them all at once:

```python
from ersilia.api import ErsiliaError, Model

try:
    with Model("eos3b5e") as model:
        df = model.run("input.csv")
except ErsiliaError as e:
    print(f"Ersilia failed: {e}")
```

The errors you're most likely to see are:

<table data-search="false"><thead><tr><th>Error</th><th>When</th></tr></thead><tbody><tr><td><code>InvalidModelIdentifierError</code></td><td>The model isn't in the Ersilia Model Hub.</td></tr><tr><td><code>InvalidOptionError</code></td><td>An option has an invalid value, or there's a problem with an input or output file.</td></tr><tr><td><code>ModelFetchError</code></td><td>The model couldn't be fetched.</td></tr><tr><td><code>ModelNotAvailableLocallyError</code></td><td>The model isn't fetched.</td></tr><tr><td><code>ModelServeError</code></td><td>The model couldn't be started.</td></tr><tr><td><code>SessionBusyError</code></td><td>Another model is already served in this session.</td></tr><tr><td><code>ModelNotServedError</code></td><td>The model isn't served, or its server stopped.</td></tr><tr><td><code>ModelNotDeletableError</code></td><td>The model can't be deleted, for example because it's served.</td></tr><tr><td><code>EmptyRunOutputError</code></td><td>The model produced no output.</td></tr></tbody></table>

To catch a specific error, import it from `ersilia.utils.exceptions_utils.exceptions` (`InvalidModelIdentifierError` and `ModelNotAvailableLocallyError`) or `ersilia.utils.exceptions_utils.api_exceptions` (all the others).

## Sessions

Like the CLI, the Python API serves one model at a time per session. By default, a session belongs to the process that started Python, such as your terminal or your notebook server. To serve a second model, close the first one, or the second `serve` raises `SessionBusyError`.

If your code runs in an environment where the parent process changes between steps, such as `conda run`, `make`, or a CI pipeline, name the session with the `ERSILIA_SESSION` environment variable, so all steps share it:

```bash
export ERSILIA_SESSION=myproject
```

Names can use letters, digits, dots, hyphens, and underscores, up to 64 characters, and can't be only digits. A named session is removed when you close its model.

## API reference

For the full reference, generated from the code, see the [Ersilia API documentation](https://ersilia-os.github.io/ersilia/source/autodoc/ersilia.api.html).
