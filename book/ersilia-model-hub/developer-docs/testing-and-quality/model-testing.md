# Model testing

`ersilia test` checks that a model's files, metadata, and outputs are what Ersilia expects, runs it, and measures its performance. Contributors use it before opening a pull request, and Ersilia's CI uses it to test every model change and every Docker image before it's published.

## Run a test

Testing needs Ersilia installed with its test dependencies:

```bash
pip install "ersilia[test]"
```

Choose a level and where to get the model from:

```bash
ersilia test eos3b5e --shallow --from_dir ~/models/eos3b5e
```

| Source              | Tests                                                                           |
| ------------------- | ------------------------------------------------------------------------------- |
| `--from_dir <PATH>` | A local copy of the model repository. Use this while you're working on a model. |
| `--from_github`     | The model's code on GitHub.                                                     |
| `--from_s3`         | The model's archive in Ersilia's S3 storage.                                    |
| `--from_dockerhub`  | The model's Docker image. Add `--version` to test a specific image tag.         |

When the tests finish, Ersilia saves a report and deletes the model from your machine.

## Test levels

Choose at least one level. Each level runs the checks of the levels before it.

### Inspect (`--inspect`)

Checks the repository without running the model:

* **Metadata.** The identifier matches the repository, and the slug, status, title, description, task, input, output, interpretation, tags, license, publication, source code, contributor, Docker Hub and S3 URLs, Docker architecture, biomedical area, and target organism are valid.
* **Required files.** `install.yml`, `metadata.yml`, `README.md`, `LICENSE`, `model/framework/run.sh`, `model/framework/columns/run_columns.csv`, and the example files `run_input.csv` and `run_output.csv`.
* **Output columns.** The number of columns in `run_columns.csv` matches the metadata's output dimension, column names use only lowercase letters, digits, and underscores, and the example output has the same columns as `run_columns.csv`.
* **Dependencies.** Every dependency in `install.yml` has a pinned version. If not, testing stops here.
* **Size.** The size of the repository.

### Surface (`--surface`)

Fetches and runs the model once:

* **Fetch.** The model fetches correctly, and Ersilia records the size of its environment or Docker image.
* **Example run.** The model runs on `run_input.csv`. For models with fixed output, the result must match `run_output.csv`. For models with variable output, it must not be empty. In both cases, the columns must match `run_columns.csv`.
* **Background jobs.** The same input also runs through the server's job API.

### Shallow (`--shallow`)

Checks the content and consistency of the outputs:

* **Output content.** The output file has the expected content for its inputs.
* **Robustness.** The model runs on three randomly chosen molecules without returning only empty values. For models with variable output, Ersilia retries with five and then ten molecules before failing.
* **Consistency.** For models with fixed output, running the same input twice gives the same result, within a small tolerance for numerical outputs.
* **Ersilia and `run.sh` agree.** For models with fixed output, Ersilia's output matches the output of running `run.sh` directly. If it doesn't, testing stops. With `--from_dockerhub`, add `--permissive` to turn this failure into a warning.

Consistency checks are skipped for models with variable output and for models with an `Online` source.

### Deep (`--deep`)

Measures how long the model takes to run on 1, 10, 100, 1,000, and 10,000 inputs. These are the computational performance values recorded in the model's metadata.

## The report

Ersilia saves a report named `<MODEL_ID>-test.json` in the current folder, or in the folder given with `--report_path`. Add `--clean` to remove temporary files after testing.

If a check stops the testing, Ersilia still saves a report with the results so far, and deletes the model.

## Where it runs automatically

Ersilia's CI runs `ersilia test` on every model, through the workflows in [ersilia-model-workflows](https://github.com/ersilia-os/ersilia-model-workflows):

| When                                                    | Command                                                                            |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| On every pull request to a model                        | `ersilia test <MODEL_ID> --shallow --from_dir <MODEL_ID>`                          |
| When the model's code on GitHub changes                 | `ersilia test <MODEL_ID> --shallow --from_github`                                  |
| Before a Docker image is published, for AMD64 and ARM64 | `ersilia test <MODEL_ID> --deep --from_dockerhub --version dev-amd64 --permissive` |

For how these workflows fit together, see [CI/CD workflows](/broken/pages/f858c75afc06c02a4bb36dbfaa72c23bb9c84f67).
