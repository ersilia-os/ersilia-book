# Command Line Interface

The `ersilia` command line interface (CLI) fetches, serves, and runs models from the Ersilia Model Hub. This page describes every command and its options.

{% hint style="info" %}
Run `ersilia --help` for the updated list of commands, or `ersilia <command> --help` for a command's options.
{% endhint %}

## Run models

Running a model always follows the same cycle: `fetch`, `serve`, `run`, `close`.

A served model belongs to the terminal where you served it: `run`, `info`, `example`, and `close` act on that model.

To show detailed logs for any command, add `-v` before the command name:

```bash
ersilia -v fetch eos3b5e
```

### fetch

Downloads a model and sets it up on your machine. By default, Ersilia fetches the model's Docker image from Docker Hub, so Docker must be running. Fallback is `--from_github`. Use the `--from_*` options to fetch from another source. Fetching from source creates a conda environment for the model.

```bash
ersilia fetch <MODEL> [OPTIONS]
```

<table data-search="false"><thead><tr><th>Option or argument</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td><code>MODEL</code></td><td>Required</td><td>The model's identifier, such as <code>eos3b5e</code>, or its slug.</td></tr><tr><td><code>--from_dockerhub</code></td><td>Used if no other source is given</td><td>Fetch the model's Docker image from Docker Hub.</td></tr><tr><td><code>--version</code></td><td>Latest</td><td>The Docker image version to fetch from Docker Hub.</td></tr><tr><td><code>--from_github</code></td><td>Off</td><td>Fetch from source, by cloning the model's GitHub repository.</td></tr><tr><td><code>--from_s3</code></td><td>Off</td><td>Fetch from source, using the model's archive in Ersilia's S3 storage.</td></tr><tr><td><code>--from_dir</code></td><td>None</td><td>Fetch from source, from a local copy of the model repository. Takes a relative or absolute path.</td></tr><tr><td><code>--from_hosted</code></td><td>None</td><td>Connect to a model hosted elsewhere, given its URL. Ersilia only creates the local folder structure and doesn't download the model.</td></tr><tr><td><code>--from_apptainer</code></td><td>Off</td><td>Fetching from an apptainer image (under development=</td></tr></tbody></table>

```bash
# examples
ersilia fetch eos3b5e
ersilia fetch eos3b5e --from_github
ersilia fetch eos3b5e --from_dir ~/models/eos3b5e
```

{% hint style="info" %}
If the model is already on your machine, `fetch` doesn’t download it again. To fetch it again, delete it first with `ersilia delete`
{% endhint %}

### serve

Starts a fetched model as a local API, ready to receive inputs from `ersilia run`. If the model isn't on your machine yet, Ersilia asks wether to fetch it first. If you serve a model on a terminal that is already serving another model, Ersilia asks whether to close it to serve the new one instead

```bash
ersilia serve <MODEL> [OPTIONS]
```

<table data-search="false"><thead><tr><th>Option or argument</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td><code>MODEL</code></td><td>Required</td><td>The model's identifier or slug.</td></tr><tr><td><code>-p</code>, <code>--port</code></td><td>Chosen automatically</td><td>The port for the local API.</td></tr><tr><td><code>--enable_cache</code> / <code>--disable_cache</code></td><td>Disabled</td><td>Cache predictions in Redis, so repeated inputs return instantly (needs Docker). See <a href="../running-models/results-caching.md">Results caching</a>.</td></tr><tr><td><code>--max_cache_memory_frac</code></td><td><code>0.3</code></td><td>The maximum fraction of system memory Redis can use for the cache, between 0 and 1.</td></tr><tr><td><code>-rs</code>, <code>--read_store</code></td><td>Off</td><td>Read previously computed predictions from the <code>isaura</code> precalculation store before running the model. Requires <code>isaura</code>. See <a href="../data-storage/precalculation-store.md">Precalculation Store</a>.</td></tr><tr><td><code>-ws</code>, <code>--write_store</code></td><td>Off</td><td>Write predictions to the <code>isaura</code> precalculation store as they're computed. Requires <code>isaura</code> and <code>--access</code>.</td></tr><tr><td><code>-a</code>, <code>--access</code></td><td>None</td><td>The visibility of predictions written to the store: <code>public</code> or <code>private</code>. Required with <code>--write_store</code>.</td></tr><tr><td><code>-t</code>, <code>--track</code></td><td>Off</td><td>Track the model's runs, such as input and output statistics, errors, and timing, and send them to Ersilia's S3 tracking bucket. Requires Ersilia's credentials to upload.</td></tr><tr><td><code>--tracking_use_case</code></td><td><code>local</code></td><td>The use case to record when tracking: <code>local</code>, <code>self-service</code>, <code>hosted</code>, or <code>test</code>.</td></tr></tbody></table>

```bash
# examples
ersilia serve eos3b5e
ersilia serve eos3b5e --port 12450 --enable_cache --max_cache_memory_frac 0.5
```

### run

A comma-separated CSV file with a header and one input per line. If the file has several columns, Ersilia uses the one named `smiles` or `input`.

```bash
ersilia run -i <INPUT> -o <OUTPUT> [OPTIONS]
```

| Option               | Default  | Description                                              |
| -------------------- | -------- | -------------------------------------------------------- |
| `-i`, `--input`      | Required | A CSV file with a single column and a header.            |
| `-o`, `--output`     | Required | The file to save the predictions to, as `.csv` or `.h5`. |
| `-b`, `--batch_size` | `100`    | The number of inputs sent to the model in each batch.    |

<pre class="language-bash"><code class="lang-bash"><strong>ersilia run -i input.csv -o output.csv --batch_size 1000
</strong></code></pre>

### close

Stops the model served in this terminal and frees its resources. It has no options.

```bash
ersilia close
```

## Explore models

### catalog

Lists models, either those fetched on your machine (the default) or all models in the Ersilia Model Hub. You can also browse all models in the [Ersilia catalog](https://catalog.ersilia.io/).

```bash
ersilia catalog [MODEL] [OPTIONS]
```

| Option or argument  | Default   | Description                                                                                                                |
| ------------------- | --------- | -------------------------------------------------------------------------------------------------------------------------- |
| `--local` / `--hub` | `--local` | List the models on your machine, or all models in the Hub.                                                                 |
| `--more` / `--less` | `--less`  | Show more metadata for each model, or only its identifier.                                                                 |
| `--task`            | None      | Show only models of one task: `Annotation`, `Representation`, or `Sampling`.                                               |
| `--card`            | Off       | Show the full model card of the model given as `MODEL`. The model doesn't need to be on your machine. Can be saved with -o |
| `-o`, `--output`    | None      | Save the catalog to a `.csv` or `.json` file.                                                                              |

```bash
# examples
ersilia catalog
ersilia catalog --hub --more -o catalog.json
ersilia catalog --card eos3b5e
```

### info

Shows information about the served model, such as its title, description, identifiers, and its GitHub, S3, and Docker Hub links.

```bash
ersilia info [OPTIONS]
```

| Option           | Default | Description                                       |
| ---------------- | ------- | ------------------------------------------------- |
| `-o`, `--output` | None    | Save the information to a `.json` or `.csv` file. |

### example

Generates example inputs for a model: the served model, or the one you name.

```bash
ersilia example [MODEL] [OPTIONS]
```

| Option or argument    | Default          | Description                                                                                                                                                                                                  |
| --------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `MODEL`               | The served model | The model to generate examples for, if none is served.                                                                                                                                                       |
| `-n`, `--n_samples`   | 5                | The number of examples to generate. Ignored in `curated` mode.                                                                                                                                               |
| `-m`, `--mode`        | `random`         | How examples are chosen: `random`, from Ersilia's list of maintained inputs; `deterministic`, always the same examples, for any model; or `curated`, the examples from the model's own `run_input.csv` file. |
| `-o`, `--output_file` | Required         | The file to save the examples to.                                                                                                                                                                            |

```bash
ersilia example eos3b5e -n 10 -o examples.csv
```

## Manage your installation

### delete

Removes a model from your machine, including its files, conda environment, Docker image and containers, and database entries.

```bash
ersilia delete [MODEL] [OPTIONS]
```

| Option or argument | Default | Description                        |
| ------------------ | ------- | ---------------------------------- |
| `MODEL`            | None    | The model to delete.               |
| `--all`            | Off     | Delete all models on your machine. |

If the model is being served in your terminal, Ersilia asks before closing and deleting it.

### uninstall

Removes Ersilia completely from your machine, including the Ersilia folder with all fetched models, all Ersilia Docker images and conda environments, and the `ersilia` Python package. It has no options.

```bash
ersilia uninstall
```

## Develop models

### test

Tests a model and writes a report on its behavior and performance. To learn what each level checks, see [Model Testing](../testing-and-quality/model-testing.md). Testing needs Ersilia installed with its test dependencies: `pip install "ersilia[test]"`. You don't need to fetch a model before testing it.

```bash
ersilia test <MODEL> [OPTIONS]
```

<table data-search="false"><thead><tr><th>Option or argument</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td><code>MODEL</code></td><td>Required</td><td>The model's identifier or slug.</td></tr><tr><td><code>--inspect</code></td><td>Off</td><td>Check the file structure and metadata, without running the model.</td></tr><tr><td><code>--surface</code></td><td>Off</td><td>Also fetch the model and run it once.</td></tr><tr><td><code>--shallow</code></td><td>Off</td><td>Also check inputs, outputs, and the consistency of results.</td></tr><tr><td><code>--deep</code></td><td>Off</td><td>Also measure computational performance.</td></tr><tr><td><code>--from_dir</code></td><td>None</td><td>Test a local copy of the model repository.</td></tr><tr><td><code>--from_github</code></td><td>Off</td><td>Test the model from its GitHub repository.</td></tr><tr><td><code>--from_s3</code></td><td>Off</td><td>Test the model from Ersilia's S3 storage.</td></tr><tr><td><code>--from_dockerhub</code></td><td>Off</td><td>Test the model's Docker image from Docker Hub.</td></tr><tr><td><code>--version</code></td><td>Latest</td><td>The Docker image version to test.</td></tr><tr><td><code>--permissive</code></td><td>Off</td><td>Turn failures of the consistency check between Ersilia and <code>run.sh</code> into warnings. Only with <code>--from_dockerhub</code>.</td></tr><tr><td><code>--report_path</code></td><td>The current folder</td><td>Where to save the report, <code>&#x3C;MODEL>-test.json</code>.</td></tr><tr><td><code>--clean</code></td><td>Off</td><td>Remove the temporary folder after testing.</td></tr></tbody></table>

```bash
# examples
ersilia test eos3b5e --shallow --from_dir ~/models/eos3b5e
ersilia test eos3b5e --deep --from_dockerhub
```

## Sessions and logs

Ersilia stores the logs of each terminal in a session folder under `~/eos/sessions`. All commands run in the same terminal belong to the same session, and serving a model shows the path to its session folder. Sessions are temporary: after a terminal is closed, Ersilia removes its session folder the next time it starts a new session.
