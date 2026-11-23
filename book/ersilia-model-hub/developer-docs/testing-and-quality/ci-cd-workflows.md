# CI CD workflows

Ersilia uses GitHub Actions to test and publish its code automatically. Workflows run in three places:

* **The** [**Ersilia repository**](https://github.com/ersilia-os/ersilia), for the CLI and Python package.
* **Every model repository.** Each model has short caller workflows, copied from [eos-template](https://github.com/ersilia-os/eos-template), which run reusable workflows from [ersilia-model-workflows](https://github.com/ersilia-os/ersilia-model-workflows). Changing a reusable workflow updates every model at once.
* **The** [**ersilia-pack repository**](https://github.com/ersilia-os/ersilia-pack), which builds the base images that model images are built from.

The workflows that monitor models already in the Hub are described in [Model Hub maintenance](../maintenance/model-hub-maintenance.md).

## Ersilia

### Test the codebase

`tests_and_cleanup.yml` runs on every pull request and push to `master`. Its jobs run in this order:

1. **`ruff-format`** checks the code with Ruff.
2. **`run-pytest`** runs the unit tests.
3. **`run-default-cli-test`** runs the [CLI testing playground](cli-testing.md) with its defaults, in Python 3.12.
4. **`run-cli-test-multiple`** fetches several models from Docker Hub and serves them.
5. **`run-cli-test-parallel`** fetches, serves, and runs a model on Ubuntu and macOS, in every Python version the playground supports, from GitHub, S3, and Docker Hub. Docker Hub is tested on Ubuntu only.
6. **`update-model-request-template`** runs only when the list of model tags changes. It sorts the tags and updates the [model request form](https://github.com/ersilia-os/ersilia/blob/master/.github/ISSUE_TEMPLATE/model_request.yml) to match.

### Release Ersilia

`publish.yaml` runs on the first day of every month at 03:00 UTC, and can also be started by hand:

1. **`version`** increases the version number, creates a release tag, and updates `CITATION.cff` and `codemeta.json`.
2. **`gh-release`** creates the GitHub release, with notes generated from the commit history.
3. **`pypi-release`** publishes the new version to PyPI.

The [conda-forge package](https://github.com/conda-forge/ersilia-feedstock) is updated separately: a new PyPI release opens a pull request in the feedstock, which an Ersilia maintainer reviews and merges.

### Approve a model request

When an Ersilia maintainer comments `/approve` on a model request issue, `approve-trigger.yml` checks that the issue has the `new-model` label and starts `approve-dispatch.yml`, which:

1. Reads the model request and creates a new model repository from eos-template, with a new `eos` identifier.
2. Fills in `metadata.yml` from the request, including the contributor.
3. Adds the model to Airtable and to the catalog's S3 JSON file.
4. Comments on the issue, so the contributor can start working.

### Other workflows

| Workflow                                           | Runs                                                                     | What it does                                                                                                        |
| -------------------------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| `docs.yml`                                         | On pushes to `master`, and on pull requests that change the code or docs | Builds the [API documentation](https://ersilia-os.github.io/ersilia/) with Sphinx and publishes it to GitHub Pages. |
| `json-check.yml`                                   | On pushes to `master` and on pull requests                               | Checks that JSON and YAML files are valid.                                                                          |
| `airtable_to_json.yml`                             | Every day at 02:00 UTC                                                   | Exports the model metadata from Airtable to the JSON file in S3 that the catalog reads.                             |
| `update-model-actions-from-eos-template-input.yml` | By hand                                                                  | Updates a model's workflows to the current version in eos-template.                                                 |
| `write_secrets.yml`                                | On new tags                                                              | Copies the secrets Ersilia needs to a private repository.                                                           |

## Models

Each model repository has five caller workflows. Together, they take a model from a pull request to a published release.

{% stepper %}
{% step %}
#### Pull request

`test-model-pr.yml` runs on every pull request to `main`. It tests the code in the pull request with `ersilia test --shallow --from_dir`, so a change can't be merged if it breaks the model.
{% endstep %}

{% step %}
#### Push to main

`upload-model.yml` runs on every push to `main`, including merged pull requests, and can also be started by hand. Its jobs run one after another, each only if the previous one succeeded:

1. **`test-model-source`** tests the model from GitHub with `ersilia test --shallow`, then updates its metadata, Airtable, and README.
2. **`upload-model-to-s3`** uploads the model to S3 and records its size and S3 URL.
3. **`upload-ersilia-pack`** builds the model's Docker images with [Ersilia Pack](../packaging-and-distribution/ersilia-pack.md), for AMD64 and ARM64, and tags them `dev`.
{% endstep %}

{% step %}
#### Test the images

`test-model-image.yml` runs when `upload-model.yml` succeeds:

1. **`test-image`** tests the `dev` images with `ersilia test --deep --from_dockerhub --permissive`. If at least one architecture passes, it tags the image as `latest` and with the date.
2. **`post-upload`** creates the first release, `v1.0.0`, if the model has none yet, and tags the image with it. It then sets the model's status to `Ready` and records its Docker Hub URL, architectures, image size, computational performance, and release in the metadata, Airtable, the catalog's S3 JSON file, the README, and the Docker Hub description.
3. **`build-sif`** runs only when `post-upload` created the first release. It builds and publishes the model's [Apptainer image](../packaging-and-distribution/apptainer-singularity-images.md).
{% endstep %}

{% step %}
#### Release

`retag-release-docker.yml` runs when a new release is published. It tags the `latest` image with the release's version, updates the metadata and Airtable, and then rebuilds the Apptainer image for that major version.
{% endstep %}
{% endstepper %}

The fifth caller, `post2slack.yml`, posts a message on Slack when an issue is opened in the model repository.

{% hint style="info" %}
A model's workflows are copied from eos-template when its repository is created, so older models may have older versions. To update them, run `update-model-actions-from-eos-template-input.yml` in the Ersilia repository.
{% endhint %}

## Ersilia Pack

`upload-ersiliapack-to-dockerhub.yml` runs on every pull request and push to `main` in the ersilia-pack repository:

1. **`ruff-format`** checks the code with Ruff.
2. **`run-pytest`** runs the unit tests.
3. **`test-pipeline`** packs and serves a test model in each Python version from 3.8 to 3.12.
4. **`upload_ersiliapack_to_dockerhub`** runs only on pushes to `main`. It builds the base images, such as `ersiliaos/ersiliapack-py312` and the conda variants, for AMD64 and ARM64, and pushes them to Docker Hub. Every model's Docker image starts from one of these.
