# Troubleshooting models

This page covers technical problems, such as installation errors, missing dependencies, or failing tests. It doesn't cover whether a model's predictions are accurate for your use case.

When Ersilia fetches a model, it tests it on three random molecules. If the model can't produce an output, the fetch stops with an error like this:

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

{% stepper %}
{% step %}
#### Fetch from your local copy

Fetch the model from your clone in verbose mode, and save the log to a file:

```bash
ersilia -v fetch eos3b5e --from_dir eos3b5e > out.log 2>&1
```

Read the whole log, not just the last lines. Look for error messages, dependency conflicts, and memory or connection problems.
{% endstep %}

{% step %}
#### Run the model without Ersilia

If the fetch fails, check whether the model itself runs. Create a fresh conda environment with the Python version and packages from `install.yml`, installing the packages one at a time so you can see which one fails. Then run the model directly:

```bash
cd eos3b5e/model/framework
bash run.sh . examples/run_input.csv output.csv
```

Compare `output.csv` with `examples/run_output.csv`. The most common causes of failure are:

* **Imports**: relative imports that break when Ersilia runs the code from another folder. Use absolute paths.
* **Input and output formats**: the model reads or writes data in a different format from the one Ersilia expects. See [The `main.py` file](model-template.md#the-main.py-file).
* **CPU and GPU settings**: some models, for example many built with PyTorch, assume a GPU is available.
{% endstep %}

{% step %}
#### Fetch again

Once the model runs on its own, remove any temporary changes, such as `print` statements, update `install.yml` with the package versions that work, and fetch the model from your local copy again.
{% endstep %}
{% endstepper %}

## Use the skills

Two of Ersilia's skills for Claude Code automate this work:

* `/ersilia-model-test` tests a model the way Ersilia will run it in the Hub, and reports where it fails. For example: `/ersilia-model-test eos3b5e ~/models/eos3b5e`.
* `/model-fixing` reviews where a model failed its tests and fixes it automatically.

To set them up, see [Ersilia Skills](../../agentic-ai/ersilia-skills.md). If you work without the skills, the steps above and the detailed guides in the archive cover the same ground.

## Go deeper

For problems the steps above don't solve, the [Archive](https://app.gitbook.com/s/LNLMUraHo6ztNotxIuIq/) has detailed guides on:

* Packaging a model locally with [ersilia-pack](https://github.com/ersilia-os/ersilia-pack) to find where packaging fails.
* Inspecting a model's Docker image.
* Models that work on AMD64 but fail on ARM64, such as Apple silicon Macs.
* Building a model from Ersilia's base Docker images when the image wasn't pushed to Docker Hub.

## Ask for help

If you can't fix the model, open a bug report on [ersilia-os/ersilia](https://github.com/ersilia-os/ersilia/issues). Include the model identifier, your operating system, the full `out.log` file, and what you've already tried.
