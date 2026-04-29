---
hidden: true
---

# Precalculation Store

**Isaura** is **Ersilia’s pre-calculation store**: it stores model outputs in **Ersilia output format**, stores them efficiently in object storage, and serves them back later via **exact** or **approximate** lookup.

* Benchmark: see [Benchmarks](https://github.com/ersilia-os/isaura/blob/main/BENCHMARK.md).
* How it works (mechanism): see [How Isaura Works](https://github.com/ersilia-os/isaura/blob/main/HOW_IT_WORKS.md).

***

## Quick start guide

Isaura uses [`uv`](https://docs.astral.sh/uv/getting-started/installation/) for fast Python dependency management.

{% stepper %}
{% step %}
### Clone and set up

```bash
git clone https://github.com/ersilia-os/isaura.git
cd isaura
uv sync
source .venv/bin/activate
```
{% endstep %}

{% step %}
### Start all services

#### Prerequisites

* [Docker](https://www.docker.com/get-started) installed and running
* Docker Compose installed
  * Ubuntu: follow Docker’s install docs
  * macOS: `brew install docker-compose`

#### Fastest way

```bash
isaura engine --start
```
{% endstep %}

{% step %}
### Optional: Install MinIO Client (mc)

The MinIO Client (`mc`) is a command-line tool to manage MinIO/S3 storage.

#### Install (Linux/macOS)

```bash
curl -O https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
sudo mv mc /usr/local/bin/
```

#### Or with Homebrew (macOS)

```bash
brew install minio/stable/mc
```

#### Configure `mc`

```bash
mc alias set local http://localhost:9000 minioadmin123 minioadmin1234
```

Example: list projects (buckets)

```bash
mc ls local
```

MinIO web console:

* [http://localhost:9001](http://localhost:9001/)
* Username: `minioadmin123`
* Password: `minioadmin1234`

More on `mc`: [https://github.com/minio/mc?tab=readme-ov-file](https://github.com/minio/mc?tab=readme-ov-file)
{% endstep %}
{% endstepper %}

***

## Cloud functionality

Export the following environment variables.

#### Public cloud bucket (read/write)

```bash
export MINIO_CLOUD_AK=<Key here>   # access key
export MINIO_CLOUD_SK=<Key here>   # secret key
```

#### Private cloud bucket (read/write)

```bash
export MINIO_PRIV_CLOUD_AK=<Key here>   # access key
export MINIO_PRIV_CLOUD_SK=<Key here>   # secret key
```

***

## How Isaura works

An overview mechanism onto how **Isaura** operates to store and fetch calculations (more details here [How Isaura Works](https://github.com/ersilia-os/isaura/blob/main/HOW_IT_WORKS.md).). It relies on four main different but related services.

#### Storage & query engines (summary)

* **DuckDB**: used as the _write_ engine (ingestion into Parquet chunks) and _query_ engine (exact retrieval from Parquet).
* **MinIO**: object storage backend for Parquet data + indexes.
* **Milvus**: used only for _approximate_ lookup (top-1 nearest input).
* **NNS**: Go-based REST API server for high-performance Milvus ingest/query: [NN-Search API](https://github.com/ersilia-os/nn-search/tree/main/api)

#### DuckDB object paths (“DuckDB URL”)

Isaura organizes artifacts in MinIO and DuckDB reads/writes those objects through S3-compatible access.

Logical paths:

* Parquet data:
  * `s3://<bucket>/<eos_id>/<version>/data/chunk_*.parquet`
* Bloom filter:
  * `s3://<bucket>/<eos_id>/<version>/bloom.pkl`
* Access metadata:
  * `s3://<bucket>/<eos_id>/<version>/access.json`

> With MinIO, DuckDB is configured with an S3 endpoint (e.g. `http://localhost:9000`) and credentials; the object keys remain under `s3://...`.

***

## Projects (buckets) and access

Isaura stores calculations in **projects** (MinIO buckets).

Default projects:

* `isaura-public`
* `isaura-private`

### Folder structure

Example:

```
isaura-public/
  eosid/
    version/
      bloom.pkl
      access.json
      data/
        chunk_{idx}.parquet
```

* Each `chunk_{idx}.parquet` contains **up to 2,000,000 rows** (2M max) for DuckDB performance (row grouping / scanning efficiency).
* `bloom.pkl` enables fast membership checks (“does this input exist?”).
* `access.json` stores **inputs and their access classification** (`public` / `private`).

### Copying from custom projects into defaults

When you **copy calculations** for a given **project + model + version**:

1. Isaura reads `<custom_project>/<eos_id>/<version>/access.json`
2. It routes each input/output into:
   * `isaura-public` if access is `public`
   * `isaura-private` if access is `private`
3. Bloom filter(s) are updated accordingly.
4. **Inputs are registered into Milvus** at copy time (so they become available for approximate search).

> Important: Milvus registration happens **when copying from a custom project to the default projects**, not necessarily at initial write into the custom project.

***

### Approximate search (Milvus)

Approximate search is enabled when you request ANN/nearest-neighbor behavior.

Current behavior:

* For each query input, Milvus returns the **top-1 most similar stored input**
* Similarity metric: **Jaccard similarity**
* Current input type supported for ANN: **CPD inputs**
* Representation: **1024-bit Morgan fingerprints**
* Collection name:
  * `{ersilia_eos_id}_{version}`

Milvus stores _input representations_ used for matching (not full model outputs). After the nearest stored input is found, Isaura fetches the corresponding cached outputs via **DuckDB + MinIO** and returns results in **Ersilia output format**.

***

## Commands at a glance

> **Buckets** are MinIO projects (storage directories) that hold model calculations.

| Command   | Alias | Required options                                                      | Optional options                                                            | What it does                                                                                                                            |
| --------- | ----- | --------------------------------------------------------------------- | --------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `write`   | —     | `-i/--input-file`, `-m/--model`                                       | `-pn/--project-name`, \`--access \[public                                   | private                                                                                                                                 |
| `read`    | —     | `-i/--input-file`, `-m/--model`                                       | `-pn/--project-name`, `--access`, `-v/--version`, `-o/--output-file`, `-nn` | Read/download results for inputs in a CSV and optionally save as CSV/HDF5. Use `-nn` for approximate search (ANN).                      |
| `copy`    | `cp`  | `-m/--model`, `-v/--version`, `-pn/--project-name`, `-o/--output-dir` | —                                                                           | Copy all artifacts for a model/version from a project to a local directory. If `-o` is omitted, logs counts; with `-o` it writes files. |
| `move`    | `mv`  | `-m/--model`, `-v/--version`, `-pn/--project-name`                    | —                                                                           | Move/relocate server-side artifacts for a model/version within the project space.                                                       |
| `remove`  | `rm`  | `-m/--model`, `-v/--version`, `-pn/--project-name`, `-y/--yes`        | —                                                                           | Permanently delete artifacts for a model/version from a project. Safety-guarded by `--yes`.                                             |
| `inspect` | —     | `-m/--model`, `-v/--version`, `-o/--output-file`                      | `-pn/--project-name`, `--access`, `-i/--input-file`, `--cloud`              | Inspect available items or validate inputs. With `-i`, validates inputs and writes a report; without `-i`, lists available entries.     |
| `catalog` | —     | `-pn/--project-name`                                                  | `--cloud`                                                                   | List models present in a project (bucket).                                                                                              |

***

## Brief CLI usage examples

| Example                         | Command                                                                                                 | Description                                                                                     |
| ------------------------------- | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| 🧾 Write results calculation    | `isaura write -i data/ersilia_output.csv -m eos8a4x -v v2 -pn myproject --access public`                | Upload/write outputs (input column must be `input`) for a model + version using a CSV as input. |
| 📥 Read results (exact)         | `isaura read -i data/inputs.csv -m eos8a4x -v v2 -pn myproject -o data/outputs.csv`                     | Read results for inputs and save to an output CSV file.                                         |
| 🔁 Read results (approximate)   | `isaura read -i data/inputs.csv -m eos8a4x -v v2 -pn myproject -o data/outputs.csv -nn`                 | Fetch results using approximate search (Milvus top-1 similar input).                            |
| 📂 Copy buckets                 | `isaura copy -m eos8a4x -v v1 -pn myproject-private -o ~/Documents/files/`                              | Copy all model artifacts from a project to a local directory.                                   |
| 🚚 Move buckets                 | `isaura move -m eos9876 -v v1 -pn myproject-private`                                                    | Move or relocate artifacts for a model/version within the project.                              |
| 🗑️ Remove buckets              | `isaura remove -m eos8a4x -v v1 -pn myproject-private --yes`                                            | Permanently delete artifacts for a model/version from a project.                                |
| 🔍 Inspect inputs (validate)    | `isaura inspect inputs -m eos8a4x -v v1 -pn myproject -i data/inputs.csv -o reports/inspect_report.csv` | Validate input data and output a report.                                                        |
| 📋 List available model results | `isaura inspect -m eos8a4x -v v1 -o reports/available.csv`                                              | List all available inputs/files for a model/version.                                            |
| 📚 Catalog project models       | `isaura catalog -pn myproject`                                                                          | Display all models within a project.                                                            |

***

## API usage examples

```python
from isaura.manage import (
    IsauraWriter,
    IsauraReader,
    IsauraMover,
    IsauraCopy,
    IsauraRemover,
    IsauraInspect,
    IsauraPull,
    IsauraPush,
)

writer = IsauraWriter(
    input_csv="data/input.csv",
    model_id="eos8a4x",
    model_version="v1",
    bucket="my-project",
    access="public",  # 'public', 'private', or 'both'
)
writer.write()

reader = IsauraReader(
    model_id="eos8a4x",
    model_version="v1",
    bucket="my-project",
    input_csv="data/query.csv",
    approximate=False,  # ANN if True
)
reader.read(output_csv="results.csv")

puller = IsauraPull(
    model_id="eos8a4x",
    model_version="v1",
    bucket="my-project",
    input_csv="data/ids.csv",
)
puller.pull()

pusher = IsauraPush(
    model_id="eos8a4x",
    model_version="v1",
    bucket="my-project",
)
pusher.push()

copier = IsauraCopy(
    model_id="eos8a4x",
    model_version="v1",
    bucket="my-project",
    output_dir="backups/",
)
copier.copy()

mover = IsauraMover(
    model_id="eos8a4x",
    model_version="v1",
    bucket="my-project",
)
mover.move()

remover = IsauraRemover(
    model_id="eos8a4x",
    model_version="v1",
    bucket="my-project",
)
remover.remove()

inspector = IsauraInspect(
    model_id="eos8a4x",
    model_version="v1",
    project_name="my-project",
    access="public",
    cloud=False,
)

df_inputs = inspector.list_available(output_file="inputs.csv")
df_inspected = inspector.inspect_inputs("data/input.csv", "inspected_results.csv")
df_models = inspector.inspect_models("my-project")
```

## Ersilia ↔ Isaura Integration: Initial Setup Failure Troubleshooting

This documents a practical cleanup-and-verify workflow for cases where the **initial setup stage fails** during the **Ersilia + Isaura integration**, or when retrieval stays slow after setup.

***

#### What this helps with

Typical causes:

* Setup fails early / won’t complete cleanly
* NN server failed for some reason
* Old model artifacts keep getting reused unexpectedly
* Milvus index/collection looks wrong or stale or not index built in Milvus
* Retrieval remains slow even after a successful run

***

#### Components involved (local)

* **MinIO console** (buckets): `http://localhost:9000/`
* **Milvus** (index/collections): `http://localhost:8080/`

***

#### ⚠️ Important warning

These steps delete local data (buckets, containers/images, volumes). Only proceed if you’re okay wiping local state and rebuilding.

***

### Step-by-step recovery workflow

#### 1) Clean model artifacts from MinIO buckets

1. Open: `http://localhost:9000/`
2. In each of these buckets:
   * `isaura-public`
   * `isaura-private`
   * `ersilia`
3. Find the model you’re troubleshooting (e.g., `eosxxxx`) and **remove its stored artifacts** (select → delete).

This helps ensure you’re not reusing stale or corrupted objects.

***

#### 2) Remove old Milvus volumes (local storage reset)

```bash
sudo rm -rf ~/isaura
```

This resets local persisted Milvus/Isaura state (if your deployment uses that path).

***

#### 3) Remove Isaura-related Docker containers/images

Check what’s running / installed:

```bash
docker ps
docker images
```

Remove problematic containers:

```bash
docker rm -f <container_id>
```

Remove images (examples):

```bash
docker rmi ersiliaos/nns
docker rmi milvus
```

If you suspect the container is just stuck, you can restart instead:

```bash
docker restart <container_id>
```

***

#### 4) Inspect Milvus collection/index contents

To inspect the contents/metadata for a Milvus collection:

```bash
curl -X POST "http://localhost:8080/info?collection=eosxxxx_v1" | jq
```

Replace `eosxxxx_v1` with your real collection name.

***

#### 5) Restart the Milvus container

If Milvus is running but behaving inconsistently, restart it:

```bash
docker restart <milvus_container_id>
```

(Use `docker ps` to locate the container ID/name.)

***

#### 6) Reinstall Isaura and re-run engine setup

If anything changed (or state is inconsistent), reinstall Isaura in ersilia venv or its own venv, then run:

```bash
pip uninstall isaura
pip install git+https://github.com/ersilia-os/isaura.git
```

```bash
isaura engine -s
```

***

#### 7) Serve the model and run a batch inference test

Serve the model:

```bash
ersilia serve eosxxxx -rs -ws -a public
```

Run inference:

```bash
ersilia run -i input.csv -o output.csv -b 10000
```

***

#### 8) Run it twice to validate retrieval performance

Run the same inference command **two times**.

* The **second run should be fast** (i.e., retrieval/caching kicks in).
* If the **second run is still slow**, the issue is likely not resolved.

***

**Expected outcome**

After cleanup + rebuild, you should see:

* Clean model artifacts in buckets
* Milvus collection present and consistent
* First run: slower (index/build/warm-up)
* Second run: **fast retrieval**

***

**If it’s still broken**

If the second run is not fast:

* Contact the admin
* Open an issue with:
  * What model (`eosxxxx`) and collection name (`eosxxxx_v1`)
  * Output of:
    * `docker ps`
    * `curl -X POST "http://localhost:8080/info?collection=..." | jq`
  * Any errors from:
    * `isaura engine -s`
    * `ersilia serve ...`
    * `ersilia run ...`
