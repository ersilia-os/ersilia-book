# Model incorporation workflow

Anyone can contribute a model to the Ersilia Model Hub. If you're a researcher who has published a model, or you've found one in the literature that would help infectious and neglected disease research, you can incorporate it so that others can run it with a single command.

Every incorporation goes through the same steps, described on this page. You can do them in two ways:

* **Manually**, following the steps below. The [Model template](model-template.md) page describes every file you have to fill in.
* **With Ersilia's skills for Claude Code**, which automate most of the work. Each step below names the skill that does it. If you work manually, your goal is to produce the same result as the skill. Each skill's `SKILL.md` file in [ersilia-skills](https://github.com/ersilia-os/ersilia-skills) describes exactly what it does, so you can also use it as a checklist.

Either way, the decisions are yours: whether a model is worth incorporating, whether its results are reproduced faithfully, and whether the final pull request is right.

## Before you begin

You need:

* A GitHub account.
* Docker, installed and running.
* Ersilia, installed in test mode so you can test the model locally:

```bash
conda create -n ersilia python=3.12
conda activate ersilia
pip install "ersilia[test]"
```

* Optionally, [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) with the `ersilia-skills` repository set up, if you want to use the skills. See [Claude at Ersilia](../../agentic-ai/ersilia-skills.md).

{% hint style="info" %}
If the model's checkpoints include a file of 100 MiB or more, or if you include training code or data, those files are stored in S3 with `eosvc`, which requires AWS credentials. External contributors don't have these, so an Ersilia maintainer will help you with this step. Mention it in your model request. See [Checkpoints and storage](model-template.md#checkpoints-and-storage).
{% endhint %}

## Choose a model

Good candidates are published, open source models relevant to infectious or neglected disease research, with code and trained weights available and a license that allows redistribution. Before you start, follow the original authors' instructions to run the model on your machine, and note its dependencies.

Skill: `/paper-to-model-assessment` summarizes the paper and looks for similar models already in the Hub.

## Incorporate the model

{% stepper %}
{% step %}

#### Request the model

Open a model request issue on [ersilia-os/ersilia](https://github.com/ersilia-os/ersilia), describing the model and its source. An Ersilia maintainer reviews it and approves it by commenting `/approve`. This creates the model's repository under `ersilia-os` from the model template, with its identifier, for example `eos2r5a`.

Skill: `/model-incorporation-request`
{% endstep %}

{% step %}

#### Fork and clone the repository

Fork the model's repository to your GitHub account and clone your fork. Commit all your work to your fork's `main` branch, not to a feature branch: the pull request checks test the `main` branch of your fork.

The `/model-incorporation-request` skill does this for you once the request is approved.
{% endstep %}

{% step %}

#### Fill in the metadata

Complete `metadata.yml` with the information from the paper and the source repository. Check the source and license fields carefully against the original publication. The output dimension and the interpretation must match the output columns you define in the next step, so check them again once the code is in place.

Skill: `/model-incorporation-metadata`
{% endstep %}

{% step %}

#### Add the model code

Add the model's code and checkpoints to the [model template](model-template.md), and adapt it so the model reads Ersilia's input and writes output in the expected format. This means describing the output columns in `run_columns.csv`, pinning the dependencies in `install.yml`, and running the model to produce the example files `run_input.csv` and `run_output.csv`. The model must not download weights at run time.

Skill: `/model-incorporation-code`
{% endstep %}

{% step %}

#### Test the model

Test the model the way Ersilia will run it in the Hub:

```bash
ersilia test eosXXXX --shallow --from_dir path/to/eosXXXX
```

The same test runs automatically on your pull request. If a check fails, see [Troubleshooting models](troubleshooting-models.md).

Skills: `/ersilia-model-test` runs the test and diagnoses failures, and `/model-fixing` applies the fixes.
{% endstep %}

{% step %}

#### Reproduce the original results

Run the model as incorporated and check that it gives the same results as the original work. First compare its outputs with those of the authors' code on the same molecules. Then, for a model that predicts a property or an activity, check the performance reported in the paper; for a model that produces embeddings, check the vectors match the original code; for a generative model, check the generated molecules resemble the training set.

This is the most important check. Don't open the pull request if the results don't match the original work: comment on your model request issue instead.

Skill: `/model-incorporation-reproduce`
{% endstep %}

{% step %}

#### Open the pull request

Commit your work to your fork's `main` and open a pull request from it to the `main` branch of the model's repository. Say in the pull request how you tested the model and reproduced its results. Automated checks run on every pull request, so make sure they all pass.

Skill: `/model-incorporation-pr`. Before pushing, it offers to run `/model-optimization`, which removes dependencies the model doesn't use without changing its outputs.
{% endstep %}

{% step %}

#### Release the model

After an Ersilia maintainer merges the pull request, the model is released and published to GitHub, Docker Hub, and as an SIF image. Ersilia maintainers run this step: they check that every workflow passed, file the model's paper and close the model request.

Skill: `/model-incorporation-release`
{% endstep %}
{% endstepper %}

## Example: incorporating eosXXXX

<!-- TODO: walkthrough of one real incorporation, one block per step with its real output (issue, metadata excerpt, run_columns.csv, test result, reproduction table, pull request, release). Model to be chosen; eos2r5a can't be used because it predates the current template. -->

## Get help

If you get stuck, comment on your model request issue so an Ersilia maintainer can help. For problems with the skills themselves, open an issue on [ersilia-os/ersilia-skills](https://github.com/ersilia-os/ersilia-skills/issues).
