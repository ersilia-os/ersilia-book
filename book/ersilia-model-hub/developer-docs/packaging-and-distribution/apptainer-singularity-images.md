---
description: >-
  Run Ersilia models where Docker is not available, from one tested .sif file
  per model.
---

# Apptainer (Singularity) images

## Why Apptainer

[Apptainer](https://apptainer.org/), formerly **Singularity**, runs a container from a single file, a `.sif`, without a background service and without root.

* **HPC clusters** almost never allow Docker, because it needs root. Apptainer is the standard there and works with job schedulers.
* **Google Colab and Kaggle** have no Docker either, but they offer free GPUs.
* **A single file** can be copied to a machine without internet access.

Converting a Docker image yourself needs about three times the image size in free disk. We build each `.sif` once, centrally, and test it before publishing it.

## Image storage

There is one file per model and **major** version, in the public bucket `models-sif`:

```
https://models-sif.s3.eu-north-1.amazonaws.com/<model_id>_v<major>.sif
```

For example, `eos4e40_v1.sif` serves any release `v1.x.y` of eos4e40. Minor releases replace the file. The URL is stored in the model's `Apptainer` metadata field.

The images are built by `build-upload-sif.yml`, a reusable workflow in [ersilia-model-workflows](https://github.com/ersilia-os/ersilia-model-workflows). It runs after the Docker image has passed its tests, on the first release of a model and on every new major and minor release (see [CI/CD workflows](../testing-and-quality/ci-cd-workflows.md#models)).

## Usage

### With the Ersilia CLI

```bash
ersilia fetch eos4e40 --from_apptainer
ersilia serve eos4e40
ersilia run -i input.csv -o output.csv
ersilia close
```

The image is downloaded to `~/eos/sifs`. Set `ERSILIA_SIF_DIR` to store images elsewhere, for example where home quotas are small. An image already on disk is reused, and `ersilia delete` removes it.

Ersilia does not install Apptainer. If Apptainer is missing or cannot run images on your machine, Ersilia tells you so before downloading anything. On a cluster, Apptainer is usually a module: `module load apptainer` or `module load singularity`.

### With ersilia-apptainer

[ersilia-apptainer](https://github.com/ersilia-os/ersilia-apptainer) runs a `.sif` directly on a file, without serving the model. ersilia-apptainer needs Python 3.9 or later and calls the `singularity` command. If your system only has `apptainer`, create an alias or symlink named `singularity`.

```bash
pip install git+https://github.com/ersilia-os/ersilia-apptainer.git

# run on the first column of a CSV
ersilia_apptainer run --sif eos4e40_v1.sif --input molecules.csv --output predictions.csv

# build a .sif yourself from the model's Docker image (only if you do not download the published one)
ersilia_apptainer create --model eos4e40 --version v1.0.0     # → eos4e40_v1.sif
```

The output has the same format as `ersilia run`: `key`, `input`, then the model's columns.

### GPU

On a GPU host, add `--nv` so the container sees the driver.

On Google Colab, `--nv` works once the driver library (`libcuda.so.1`) is registered with `ldconfig`; repeat this in every new runtime. Kaggle blocks Apptainer; there, unpack the `.sif` and run it with PRoot.

{% hint style="info" %}
ersilia-apptainer runs models on CPU only. To use a GPU you have to use Apptainer directly adding the `--nv` flag.
{% endhint %}

