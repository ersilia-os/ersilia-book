---
description: Run Ersilia models where Docker is not available, from one tested .sif file per model.
---

# Apptainer (Singularity) images

## Why Apptainer

[Apptainer](https://apptainer.org/), formerly **Singularity**, runs a container from a single file, a `.sif`, without a background service and without root.

* **HPC clusters** almost never allow Docker, because it needs root. Apptainer is the standard there and works with job schedulers.
* **Google Colab and Kaggle** have no Docker either, but they offer free GPUs.
* **A single file** can be copied to a machine without internet access.

Converting a Docker image yourself needs about three times the image size in free disk. We build each `.sif` once, centrally, and test it before publishing it.

## Where the images are

There is one file per model and **major** version, in the public bucket `models-sif`:

```
https://models-sif.s3.eu-north-1.amazonaws.com/<model_id>_v<major>.sif
```

For example, `eos4e40_v1.sif` serves any release `v1.x.y` of eos4e40. Minor and patch releases replace the file. The URL is stored in the model's `Apptainer` metadata field and shown in its README, next to Docker Hub and S3.

## Using a model from Apptainer

### With the Ersilia CLI

{% hint style="info" %}
`ersilia fetch --from_apptainer` is in review in [ersilia#1935](https://github.com/ersilia-os/ersilia/pull/1935) and will be available in the next Ersilia release.
{% endhint %}

```bash
ersilia fetch eos4e40 --from_apptainer
ersilia serve eos4e40
ersilia run -i input.csv -o output.csv
ersilia close
```

The image is downloaded to `~/eos/sifs`. Set `ERSILIA_SIF_DIR` to store images elsewhere, for example where home quotas are small. An image already on disk is reused, and `ersilia delete` removes it.

Ersilia does not install Apptainer. If Apptainer is missing or cannot run images on your machine, Ersilia tells you so before downloading anything. On a cluster, Apptainer is usually a module: `module load apptainer` or `module load singularity`.

### With ersilia-apptainer

[ersilia-apptainer](https://github.com/ersilia-os/ersilia-apptainer) runs a `.sif` directly on a file, without serving the model:

```bash
pip install git+https://github.com/ersilia-os/ersilia-apptainer.git

# run on a CSV whose first column is SMILES
ersilia_apptainer run --sif eos4e40_v1.sif --input molecules.csv --output predictions.csv

# build a .sif yourself from the model's Docker image (only if you do not download the published one)
ersilia_apptainer create --model eos4e40 --version v1.0.0     # → eos4e40_v1.sif
```

The output has the same format as `ersilia run`: `key`, `input`, then the model's columns.

### GPU

On a GPU host, add `--nv` so the container sees the driver.

On Google Colab, `--nv` works once the driver library (`libcuda.so.1`) is registered with `ldconfig`; repeat this in every new runtime. Kaggle blocks Apptainer; there, unpack the `.sif` and run it with PRoot.

## How the images are built

The images are built by `build-upload-sif.yml`, a reusable workflow in [ersilia-model-workflows](https://github.com/ersilia-os/ersilia-model-workflows). It runs after the Docker image has passed its tests, on the first release of a model and on every new release (see [CI/CD workflows](ci-cd-workflows.md#ersilia-models)).

1. **Check.** Read `metadata.yml` and continue only if `Docker Architecture` contains `AMD64`, because `:latest` can hold an image that failed its tests.
2. **Build.** Install Apptainer and ersilia-apptainer, and run `ersilia_apptainer create` from the model's Docker image. The `.sif` is built from the image that already passed its tests, so there is no second installation path that could drift.
3. **Smoke test.** Run the `.sif` as a normal user on `examples/run_input.csv`, and check that the rows and columns match `examples/run_output.csv`.
4. **Upload** to `s3://models-sif/<model_id>_v<major>.sif`, then check that the public URL answers.
5. **Record.** Write the URL in the `Apptainer` field of `metadata.yml`, update Airtable and the S3 JSON, and regenerate the README.

Design decisions:

* The build runs as root, in `$RUNNER_TEMP` for disk space. The smoke test runs as a normal user, like on a cluster; GitHub's Ubuntu 24.04 runners block unprivileged user namespaces through AppArmor, so the workflow lifts that restriction first.
* Ersilia validates the `Apptainer` field: empty, `none` or `null` mean there is no image; any other value must start with the `models-sif` bucket URL and end in `.sif`. Broken or third-party links never reach the Hub.
* Secrets: `AWS_ACCESS_KEY`, `AWS_SECRET_ACCESS_KEY`, `AIRTABLE_API_KEY`.

Coverage (which Ready models have a `.sif`) is tracked by the `model-monitoring` skill.
