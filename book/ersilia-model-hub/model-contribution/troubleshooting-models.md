# Troubleshooting models

This page covers technical problems, such as installation errors, missing dependencies, or failing tests. It doesn't cover whether a model's predictions are accurate for your use case.

When Ersilia fetches a model, it runs it on the first three molecules of the model's example input, `run_input.csv`. If the model can't produce an output, the fetch stops with an error like this:

```
Error occurred while fetching model: eos3b5e
```

## Run the basic checks

Many fetch errors come from the environment rather than the model. Before you investigate the model itself, check that:

* **Ersilia is up to date.** Upgrade it in your Ersilia environment with `pip install --upgrade ersilia`. If you installed it from a clone with `pip install -e .`, pull the latest changes in your clone instead.
* **You have enough disk space.** We recommend at least 10 GB free. A `No space left on device` message in the log means you need to free some up.
* **Your connection works.** Run the fetch in verbose mode, `ersilia -v fetch eos3b5e`, and look for errors such as `TimeoutError` or `NewConnectionError`, usually from the `urllib3` library. If you see them, try another network, and turn off any VPN or firewall.
* **Ersilia can create its folders.** Ersilia stores models in an `eos` folder in your home directory, with each model in `eos/dest`. If the folder isn't created, check that your user has permission to create folders there.

If the fetch still fails, and you're a user rather than a contributor, [open an issue on GitHub](https://github.com/ersilia-os/ersilia/issues) with the model identifier, your operating system, and the output of the verbose fetch.

## Diagnose a model you're incorporating

If you're incorporating or fixing a model, work from a local copy of its repository, so you can change the code and fetch it again without network issues.

{% hint style="warning" %}
When Ersilia deletes a model, which also happens when you fetch it again or run `ersilia test`, it removes every conda environment whose name starts with the model identifier, and runs `docker system prune -f`, which removes all stopped containers and unused images on your machine. Don't name your own environments after the model, for example `eos3b5e-dev`.
{% endhint %}

{% stepper %}
{% step %}
#### Fetch from your local copy

Fetch the model from your clone in verbose mode, and save the log to a file:

```bash
ersilia -v fetch eos3b5e --from_dir eos3b5e > out.log 2>&1
```

The folder must be named exactly like the model identifier. If you renamed your clone, Ersilia can't find the model and fails with "Bundle path not found". Read the whole log, not just the last lines. Look for error messages, dependency conflicts, and memory or connection problems.
{% endstep %}

{% step %}
#### Run the model without Ersilia

If the fetch fails, check whether the model itself runs. Create a fresh conda environment with the Python version and packages from `install.yml`, installing the packages one at a time so you can see which one fails. Then run the model directly:

```bash
cd eos3b5e/model/framework
bash run.sh . examples/run_input.csv output.csv
```

Compare `output.csv` with `examples/run_output.csv`. The most common causes of failure are:

* **Imports and paths**: `main.py` runs as a script, from another folder. Don't use relative imports such as `from . import helper`: helper files next to `main.py` can be imported directly. Build file paths from the location of the file, for example with `os.path.dirname(os.path.abspath(__file__))`, as the template does, and don't write paths that only exist on your machine.
* **Input and output formats**: the model reads or writes data in a different format from the one Ersilia expects. See [The `main.py` file](model-template.md#the-main.py-file).
* **CPU and GPU settings**: some models, for example many built with PyTorch, assume a GPU is available.
* **Dependency versions**: a pinned version is no longer available to install, or a package that isn't pinned has changed. Pin every package in `install.yml` to a version that is still available.
{% endstep %}

{% step %}
#### Fetch again

Once the model runs on its own, remove any temporary changes, such as `print` statements, update `install.yml` with the package versions that work, and fetch the model from your local copy again. Then run the test again, as described in the [Test step](model-incorporation-workflow.md#incorporate-the-model).
{% endstep %}
{% endstepper %}

## Fix a failing test

`ersilia test` saves a report, `eosXXXX-test.json`, and prints every check. Ignore the checks that say "not present": Ersilia's workflows fill them in after the merge. Fix the first failing check first, because a model that doesn't run makes the others fail too.

| Failing check | Likely cause | Fix |
|---|---|---|
| `simple_model_run` | The model doesn't run: an import or path error, a missing dependency, or the input isn't read correctly | Run the model without Ersilia, as in the steps above, and read the error |
| `model_description`, `metadata_*` | A value that isn't in the accepted list, a list written as a single value, placeholder text, or a description shorter than 200 or longer than 600 characters | Compare with [The `metadata.yml` file](model-template.md#the-metadata.yml-file) |
| `columns`, `metadata_dim_and_run_column_file_dim_check` | The names or the number of rows of `run_columns.csv` differ from what `main.py` writes or from `Output Dimension` | Make `run_columns.csv` match the output columns, and `Output Dimension` its number of rows |
| `install_yaml_check` | A command isn't written as a list, or a package has no pinned version | Write `["pip", "package", "version"]` and pin every version |
| `check_consistency_of_model_output` | The output changes between runs, or differs between Ersilia and `bash run.sh`: for example dropout is still active, a random seed is missing, or temporary files have unstable names | Set the seed, call `model.eval()` for PyTorch models, and use stable file names. If the model is random by design, set `Output Consistency` to `Variable` |
| Empty values for some molecules | The model can't process them | Return an empty value and keep the row. Don't drop the molecule and don't invent a value |

## Fix a failing check on your pull request

The same test runs on your pull request with `ersilia -v test eosXXXX --shallow --from_dir eosXXXX`. Open the failed run from the pull request's Checks tab, read the first error, and reproduce it locally with that command. The first fetch of a model with heavy dependencies can take more than 10 minutes, so if the run fails while the environment is being built and the model works on your machine, say so in the pull request. If the version label check fails, an Ersilia maintainer has to add the label, which is `semver:major` for a new model. See [Model versioning](model-versioning.md).

## Use the skills

Two of Ersilia's skills for Claude Code automate this work:

* `/ersilia-model-test` tests a model the way Ersilia will run it in the Hub, and reports where it fails. For example: `/ersilia-model-test eos3b5e ~/models/eos3b5e`.
* `/model-fixing` proposes fixes for the checks that failed, applies them once you agree, and runs the test again.

To set them up, see [Ersilia Skills](../../agentic-ai/ersilia-skills.md). If you work without the skills, the steps above and the detailed guides in the archive cover the same ground.

## Go deeper

For problems the steps above don't solve, the [archived Troubleshooting models page](https://ersilia.gitbook.io/ersilia-book/ersilia-book/ersilia-book-archive/ersilia-model-hub/troubleshooting-models) has detailed guides on:

* Packaging a model locally with [ersilia-pack](https://github.com/ersilia-os/ersilia-pack) to find where packaging fails.
* Inspecting a model's Docker image.
* Models that work on AMD64 but fail on ARM64, such as Apple silicon Macs.
* Building a model from Ersilia's base Docker images when the image wasn't pushed to Docker Hub.

## Ask for help

If you can't fix the model, comment on your model request issue or on your pull request, and an Ersilia maintainer will help. Include the model identifier, your operating system, the full `out.log` file, and what you've already tried. If you think the problem is in Ersilia itself, open a bug report on [ersilia-os/ersilia](https://github.com/ersilia-os/ersilia/issues) with the same information.
