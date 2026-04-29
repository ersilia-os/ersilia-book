---
description: >-
  Ersilia relies on GitHub Actions for CI/CD. Here is a high-level summary of
  workflows involved in maintaining the Ersilia CLI and code-bases from all the
  models available via the Hub.
---

# CI/CD workflows

## Ersilia CLI

These workflows ensure that we push quality code to [Ersilia](https://github.com/ersilia-os/ersilia), incorporating testing by individual components as well as end-to-end.&#x20;

### Testing

#### Testing Ersilia on a Pull Request/Push

* Objective: Ensure Ersilia continues to work with any new code merging from contributors and maintainers to the master branch. Comprehensive tests include several Python versions, supported platforms and testing several models. This workflow also runs the unit and integration test suite for the repository.
* Workflow name: Test Ersilia codebase
* Workflow file: [tests\_and\_cleanup.yml](https://github.com/ersilia-os/ersilia/blob/master/.github/workflows/tests_and_cleanup.yml)
* Runs on: automatically when a PR is open on the Ersilia master branch or a push is made directly to master.
* Jobs:
  * `install-ersilia`, installs Ersilia from Python 3.8 to Python 3.12 on an Ubuntu base.
  * `test-docker`  is the next step to follow after we confirm that Ersilia can be installed in environments created with different Python versions. This step tests whether dockerized models can be fetched using the ersilia CLI, ie the happy flow works without issues.
  * `test-os`  Since Ersilia is supported only on Linux distributions, and MacOS, this job ensures that Ersilia can be installed and run on both of these platforms, presently only utilizing a Python 3.10 environment.&#x20;
  * `run-pytest`  This is the main step that carries out unit testing within Ersilia ensuring that different components continue to work without issues.
  * `run-cli-test-single` This an integration level test that ensures different components within Ersilia interact cohesively in running a single model using the Ersilia CLI. More details on how this is implemented can be found [here](cli-testing.md).
  * `run-cli-test-multiple` Similar to the job described above, this job tests the functionality in Ersilia to simultaneously run multiple models. More details around implementation of the specific tests and the testing frameworks used can be found [here](cli-testing.md).
  * `test-colab-notebook` This test ensures that Ersilia can run in IPython notebooks, such as Jupyter or Google Colab. Specifically, this test checks whether a notebook containing Ersilia code can be executed successfully.
  * `update-model-request-template` This job within the workflow only runs when the PR or the commit on `master` branch change the file containing [model tags](https://github.com/ersilia-os/ersilia/blob/master/ersilia/hub/content/metadata/tag.txt). We expect model tags to be completely sorted because they are used to populate the [Model Request Issue](https://github.com/ersilia-os/ersilia/issues/new?assignees=\&labels=new-model\&projects=\&template=model_request.yml\&title=%F0%9F%A6%A0+Model+Request%3A+%3Cname%3E) (more on this below). The job sorts the tags file if it is changed, and then updates the Model Request Issue [template](https://github.com/ersilia-os/ersilia/blob/master/.github/ISSUE_TEMPLATE/model_request.yml) with the changes, and commits them back to the master branch.

### Packaging

#### Ersilia Release

* Objective: Distribute Ersilia as source, as a Docker image, as a PyPI package, and as a conda-forge package.&#x20;
* Workflow name: Ersilia Release
* Workflow file: [publish.yml](https://github.com/ersilia-os/ersilia/blob/master/.github/workflows/publish.yaml)
* Runs on: At the beginning of each month 3:00 AM UTC
* Jobs:
  * `version` increments the version of the codebase and creates a new release tag, while also updating this information in the [CITATION.cff](https://github.com/ersilia-os/ersilia/blob/master/CITATION.cff), and [CODEMETA.json](https://github.com/ersilia-os/ersilia/blob/master/codemeta.json) files.&#x20;
  * `gh-release` Utilizing the tag created in the previous job, a GitHub release is created. The GitHub CLI is used to automatically create this release and utilize the commit history from the `master` branch to generate release notes.
  * `pypi-release` Finally, we utilize the release artifacts created from the GitHub release job of the workflow to publish a new version of Ersilia on the Python Packaging Index (PyPI).
  * The completion of this release workflow triggers the Docker build workflow mentioned above.
* `Conda-Forge Release` Ersilia is released on conda-forge using by way of updating its [feedstock](https://github.com/conda-forge/ersilia-feedstock) maintained within the conda-forge organization on GitHub. Upon the creation of a new PyPi release, an automated pull request is created in this feedstock repository which is then manually reviewed and merged by an Ersilia maintainer. Currently an update from `Redis` to `Redis-py` in the `meta.yaml` is needed for conda to install the package properly.

### Community

#### New Model Request

* Objective: Contribute new models to the Ersilia Model Hub
* Workflow name: Approve Command Dispatch
* Workflow file: [approve-dispatch.yml](https://github.com/ersilia-os/ersilia/blob/master/.github/workflows/approve-dispatch.yml)
* Runs on: When an Ersilia maintainer comments `/approve`  on a [Model Request](https://github.com/ersilia-os/ersilia/issues/new?assignees=\&labels=new-model\&projects=\&template=model_request.yml\&title=%F0%9F%A6%A0+Model+Request%3A+%3Cname%3E) issue submitted by a model contributor. A small action named `approve-trigger.yml` first checks if we are effectively on a Model request issue.
* Jobs:&#x20;
  * `approve-command-dispatch`  The Model Request issue is parsed to ensure all required fields are filled in. If the metadata is correct, a new model repository is created from the [eos-template](https://github.com/ersilia-os/eos-template) and a record is generated in Airtable. The repository includes all the files in eos-template and a mock Git LFS object to allow for uploading Git LFS objects in the repository if required. The issue creator is notified to prompt them to start contributing to the model. Read on more about model contributions in our Model Contribution [guidelines](../model-contribution/example-of-the-model-incorporation-workflow.md).&#x20;

## Ersilia Models

Ersilia models include a series of workflow callers for the reusable workflows available in the [ersilia-model-workflows](https://github.com/ersilia-os/ersilia-model-workflows) and [eos-template](https://github.com/ersilia-os/eos-template/tree/main/.github/workflows) repository:

### Test and upload model

* Objective: Test a model using its source code and, if correct, update it across all the platforms (GitHub, S3 and Dockerhub)
* Workflow file: [upload-model.yml](https://github.com/ersilia-os/eos-template/blob/main/.github/workflows/upload-model.yml)
* Runs on: whenever code is pushed to the main branch of the repository, either directly by a maintainer or by way of merging a pull request. This workflow can also be triggered manually by a maintainer.
* Jobs:
  * `test-model-source`: uses the reusable workflow of the same name, which installs Ersilia and tests the source code of the model using the `test --shallow` command. It captures and updates the relevant metadata to Airtable
  * `upload-model-to-s3`: uses the reusable workflow of the same name and uploads the model to S3 if the test of the source code has passes
  * `upload-ersilia-pack`: uses the reusable workflow of the same name and is triggered if the upload to S3 is successful. A Docker image is built for both AMD64 and ARM64 architectures and tagged as dev. By default we first try to build a FastAPI packaged model

### Test model image

* Objective: test the dev version of a model image
* Workflow file: [test-model-image.yml](https://github.com/ersilia-os/eos-template/blob/main/.github/workflows/test-model-image.yml)
* Runs on: whenever the test and upload model has successfully completed one of the three Docker build options
* Jobs:
  * `test-image`: using the reusable workflow test-model-image, it tests the dev versions of the image for both architectures if available using the `test --deep` command. If successful, retags the image(s) as latest.
  * `post-upload`: the post-model-upload workflow is triggered if the image testing is successful. It reads the output of the test command (.json) and captures the relevant metadata (image size, computational performance etc). It also changes the status of the model to Ready. Finally it updates Airtable and the metadata and readme files in the model repository.

### Test model PR

* Objective: test that the model incorporation is complete
* Workflow file: [test-model-pr.yml](https://github.com/ersilia-os/eos-template/blob/main/.github/workflows/test-model-pr.yml)
* Runs on: whenever a PR is opened to the model repository
* Jobs:&#x20;
  * `test-model-pr`: using the workflow of the same name, the workflow installs ersilia, clones the code in the PR and performs a `test --shallow` to check the model is able to work within Ersilia.

### Retag release docker:

* Objective: tag the Docker image corresponding to the release
* Workflow file: [retag-release-docker.yml](https://github.com/ersilia-os/eos-template/blob/main/.github/workflows/retag-release-docker.yml)
* Runs on: whenever there is a new release (not triggered inside a GitHub action)
* Jobs:
  * `retag`: using the retag-release-docker reusable workflow, it checks the latest tag available and tags the "latest" Docker image with it. It also updates the corresponding metadata file and Airtable.

## Ersilia Pack

* Objective: Check code formatting, run unit and pipeline tests across multiple Python versions, and, if successful, build and upload multi-architecture Ersilia-Pack Docker images to DockerHub.
* Workflow file: [upload-ersiliapack-to-dockerhub.yml](https://github.com/ersilia-os/ersilia-pack/blob/main/.github/workflows/upload-ersiliapack-to-dockerhub.yml)
* Runs on: Whenever code is pushed to the `main` branch, or when a pull request is opened against the `main` branch.&#x20;
* Jobs:
  * `ruff-format`: checks the codebase for proper formatting and linting errors using Ruff on Python 3.12.
  * `run-pytest`: depends on `ruff-format`. It sets up a Miniconda environment, installs required dependencies, and runs standard unit tests using `pytest`.
  * `test-pipeline`: depends on `run-pytest`. It runs a testing matrix across multiple Python versions (3.8 through 3.12) and their corresponding test models using `nox`. It also provisions a Redis container specifically to test caching and multiprocessing capabilities (currently targeted at Python 3.10).
  * `upload_ersiliapack_to_dockerhub`: depends on `test-pipeline` and is strictly triggered only on a `push` event. It uses a matrix strategy to fetch base configuration files and dynamically generate Dockerfiles. It then builds Docker images for both AMD64 and ARM64 architectures across various Conda and Pip Python environments, finally tagging and pushing them to DockerHub.

## Ersilia Maintenance

The primary goal of this combined suite of workflows is to ensure the continuous reliability, accuracy, and accessibility of the models hosted on the platform. You can reed more about ersilia maintenance [here](../model-hub-maintenance.md).

### Hub health status

* Objective: Automatically generate a monthly health status report (including historical data and plots) and commit it to the repository to track the Hub's performance metrics over time.
* Workflow file: [hub-health-status.yml](https://github.com/ersilia-os/ersilia-maintenance/blob/main/.github/workflows/hub_health_status.yml)
* Runs on: A scheduled cron job on the first day of every month at 01:00 UTC.
* Jobs:
  * `monthly-health-report:` Sets up a Python 3.10 environment and installs required dependencies (including `matplotlib` for generating plots). It executes the health report generation script (`src/health_report.py`) and saves the resulting Markdown report, JSON history data, and PNG plots as downloadable workflow artifacts. Finally, if the workflow is running on the `main` or `master` branch, it automatically commits and pushes these updated files directly back into the repository using the `ersilia-bot` account.

### Repo data sync

* Objective: Automatically fetch and sync repository metadata, refresh open issues, and update the model report markdown file to keep the centralized repository tracking data current.
* Workflow file: [repo\_data\_sync.yml](https://github.com/ersilia-os/ersilia-maintenance/blob/main/.github/workflows/repo_data_sync.yml)
* Runs on: A scheduled cron job every 10 days at 00:00 UTC.
* Jobs:
  * `repository-update`: Sets up a Python 3.10 environment and installs necessary API and parsing dependencies (`requests`, `pyyaml`). It sequentially runs a series of Python scripts to fetch the repository index, update metadata, refresh open issues, and generate the updated model report. Finally, if it is running on the `main` or `master` branch, it automatically commits and pushes the updated JSON tracking files and Markdown reports back to the repository using the `ersilia-bot` account.

### Single Model test

* Objective: Run an isolated test on a specific, user-provided Ersilia model and update the repository with the resulting test reports and metadata.
* Workflow file: [single\_model\_test.yml](https://github.com/ersilia-os/ersilia-maintenance/blob/main/.github/workflows/single_model_test.yml)
* Runs on: Triggered manually by a maintainer via the GitHub Actions interface (`workflow_dispatch`). It requires the user to input a specific `model_id` (e.g., "eos4e40") before running.
* Jobs:
  * `test-model`: Prepares the Ubuntu runner by actively freeing up disk space, checks out the code, and sets up a Python 3.12 Miniconda environment. It installs the `ersilia[test]` suite directly from the source repository, along with standard CLI and Python dependencies (`git-lfs`, `gh`, `jq`, `requests`, `pyyaml`). It then executes a dedicated bash script (`scripts/test_single_model.sh`) using the provided model ID. Finally, if the workflow is on the `main` or `master` branch, it securely commits and pushes the updated JSON tracking file, Markdown report, and text summary back to the repository using the `ersilia-bot` account.

### Test models

* Objective: Automate the weekly selection and testing of a batch of Ersilia models, updating the repository with the chosen models and their corresponding test results.
* Workflow file: [test\_models.yml](https://github.com/ersilia-os/ersilia-maintenance/blob/main/.github/workflows/test_models.yml)
* Runs on: A scheduled cron job every Monday at 10:00 UTC.
* Jobs:
  * `pick-models`: Sets up a Python 3.10 environment and installs necessary dependencies (`requests`, `pyyaml`). It executes a script (`src/pick_repo.py`) to determine which models should be tested for the week. It then commits and pushes this selection to the repository as a JSON file (`files/picked_weekly.json`) using the `ersilia-bot` account.
  * `test-model`: Depends on the successful completion of `pick-models`. It prepares the runner by freeing up disk space and checking out the _latest_ `main` branch (ensuring it pulls the JSON file generated in the previous job). It then sets up a Python 3.12 Miniconda environment, installs the `ersilia[test]` suite alongside necessary CLI and Python dependencies, and runs the main testing bash script (`scripts/test_models.sh`). Finally, if running on `main` or `master`, it securely commits and pushes the updated repository tracking files, text summaries, and Markdown reports back to the repository using the `ersilia-bot` account.

### Update Metadata

* Objective:
* Workflow file: [update\_metadata.yml](https://github.com/ersilia-os/ersilia-maintenance/blob/main/.github/workflows/update_metadata.yml)
* Runs on:
* Jobs:

