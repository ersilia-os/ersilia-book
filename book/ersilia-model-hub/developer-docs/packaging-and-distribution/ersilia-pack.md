# Ersilia Pack

[Ersilia Pack](https://github.com/ersilia-os/ersilia-pack) turns a model repository into a bundle: a self-contained FastAPI server around the model's `run.sh` file. Every Ersilia model runs through it, whether it comes from a Docker image or from source.

## Usage

* **Docker images.** When a model is released, Ersilia's CI builds its Docker image by running Ersilia Pack inside a base image that matches the model's Python version. It uses a pip-only or a conda image depending on whether `install.yml` has conda packages.
* **Fetching from source.** `ersilia fetch --from_github`, `--from_s3`, or `--from_dir` create a conda environment named after the model, install Ersilia Pack in it, and pack the model into `~/eos/repository`.
* **Serving.** `ersilia serve` starts the bundle's server with `ersilia_model_serve`.

## Install

```bash
pip install git+https://github.com/ersilia-os/ersilia-pack.git
```

Ersilia Pack needs Python 3.8 or later.

## Commands

### ersilia\_model\_lint

```bash
ersilia_model_lint --repo_path <REPO_PATH>
```

Checks that a model repository has what Ersilia Pack needs:

* A valid `eos` identifier.
* A `metadata.yml` or `metadata.json` file.
* An `install.yml` file, or a legacy `Dockerfile`.
* `LICENSE` and `README.md` files.
* A `model/framework` folder.
* An example input and output for each API, such as `run_input.csv` and `run_output.csv` for `run.sh`.

### ersilia\_model\_pack

```bash
ersilia_model_pack --repo_path <REPO_PATH> --bundles_repo_path <BUNDLES_PATH> [--conda_env_name <ENV>]
```

| Option                | Default  | Description                                                   |
| --------------------- | -------- | ------------------------------------------------------------- |
| `--repo_path`         | Required | The model repository to pack.                                 |
| `--bundles_repo_path` | Required | The folder to write bundles to.                               |
| `--conda_env_name`    | Not set  | The conda environment to install the model's dependencies in. |

Packing copies the `model` folder, adds the server's code, turns `install.yml` into an install script and runs it, points `run.sh` at the right Python, and writes the model's information and API schema. If a bundle for the model already exists, it's replaced.

Each `.sh` file in `model/framework` becomes an API. For most models, that's `run.sh`, served at `/run`.

The bundle is written to `<BUNDLES_PATH>/<MODEL_ID>/<DATE>-<ID>/`:

```
<MODEL_ID>/<DATE>-<ID>/
├── app/                  # the FastAPI server
├── installs/install.sh   # generated from install.yml
├── model/                # copy of the model folder
├── static/
├── information.json      # model card, identifier, slug, and APIs
├── api_schema.json
├── status.json           # {"done": true} once packing finished
└── run_uvicorn.py
```

### ersilia\_model\_serve

```bash
ersilia_model_serve --bundle_path <BUNDLES_PATH>/<MODEL_ID> [--host <HOST>] [--port <PORT>]
```

| Option          | Default     | Description                |
| --------------- | ----------- | -------------------------- |
| `--bundle_path` | Required    | The model's bundle folder. |
| `--host`        | `0.0.0.0`   | The host to serve on.      |
| `--port`        | A free port | The port to serve on.      |

## The server's API

A served bundle has interactive documentation at `/docs`. Its main endpoints are:

| Endpoint                                            | Description                                                                                                                                                                                                |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `POST /run`                                         | Runs the model on a JSON list of inputs, such as `["CCO", "c1ccccc1"]`, and returns the results. Add `output_type=heavy` to exchange data with the model in binary format, which is faster for large runs. |
| `POST /job/submit`                                  | Submits a job to run in the background and returns its ID. Check it with `GET /job/status/{job_id}` and get the results with `GET /job/result/{job_id}`.                                                   |
| `GET /run/example/input`, `GET /run/example/output` | The model's example input and output.                                                                                                                                                                      |
| `GET /run/columns/input`, `GET /run/columns/output` | The input and output column names.                                                                                                                                                                         |
| `GET /card`                                         | The model's metadata.                                                                                                                                                                                      |
| `GET /healthz`                                      | The server's health, including CPU and memory.                                                                                                                                                             |

Results are returned as JSON, in any of the pandas `orient` formats, set with the `orient` query parameter. The default is `records`.

## Reading inputs and writing outputs in model code

Model code reads its inputs and writes its outputs with two functions from [ersilia-pack-utils](https://pypi.org/project/ersilia-pack-utils/), as the [model template](../../model-contribution/model-template.md) shows:

* `read_smiles(in_file)` returns the input file's header and the list of inputs from its first column.
* `write_out(results, header, file, dtype=np.float32)` writes the results with one column per value.

Both work with CSV files and with Ersilia Pack's binary format. The server uses the binary format when `/run` is called with `output_type=heavy`, which is faster for large runs. Use these functions rather than reading or writing files yourself, so your model works in both cases.

In the binary format, outputs must be integers or numbers, stored as `int32` or `float32`. Numbers too large for `float32` are capped at its maximum.

## Pack and serve a model locally

To find where packing fails, run Ersilia Pack on a local copy of the model, outside Ersilia:

{% stepper %}
{% step %}
#### Set up an environment

Create a conda environment with the Python version in the model's `install.yml`, and install Ersilia Pack in it.
{% endstep %}

{% step %}
#### Check the repository

```bash
ersilia_model_lint --repo_path eos3b5e
```
{% endstep %}

{% step %}
#### Pack it

```bash
ersilia_model_pack --repo_path eos3b5e --bundles_repo_path bundles
```
{% endstep %}

{% step %}
#### Serve it

```bash
ersilia_model_serve --bundle_path bundles/eos3b5e --port 8000
```

Open `http://localhost:8000/docs` and try the `/run` endpoint.
{% endstep %}
{% endstepper %}
