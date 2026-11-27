# Model incorporation workflow

Anyone can contribute a model to the Ersilia Model Hub. If you're a researcher who has published a model, or you've found one in the literature that would help infectious and neglected disease research, you can incorporate it so that others can run it with a single command.

Every incorporation goes through the same steps, described on this page. You can do them in two ways:

* **Manually**, following [Incorporate a model manually](https://ersilia.gitbook.io/ersilia-book/ersilia-book/ersilia-book-archive/ersilia-model-hub/model-incorporation-workflow), in the Ersilia Book Archive. This guide walks you through every step you need to do by hand to incorporate a model into the Ersilia Model Hub, with a worked example.
* **With Ersilia's skills for Claude Code**, which automate most of the work. Each step below names the skill that does it. If you work manually, your goal is to produce the same result as the skill. Each skill's `SKILL.md` file in [ersilia-skills](https://github.com/ersilia-os/ersilia-skills) describes exactly what it does, so you can also use it as a checklist.

Either way, the decisions are yours: whether a model is worth incorporating, whether its results are reproduced faithfully, and whether the final pull request is right.

## Before you begin

You need:

* A GitHub account.
* Ersilia, installed in test mode if you want to test the model locally before opening the pull request (the same test runs on your pull request either way):

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

Good candidates are published models that take a single small molecule as input (SMILES, InChI or molfile), perform a Hub task such as activity or property prediction, and address an endpoint relevant to Ersilia, such as antimicrobial resistance, neglected diseases or ADMET. Their code or weights must be open, with a license that allows redistribution. Before you start, follow the original authors' instructions to run the model on your machine, and note its dependencies.

Skill: [`/paper-to-model-assessment`](https://github.com/ersilia-os/ersilia-skills/tree/main/skills/paper-to-model-assessment) summarizes the paper from its PDF, checks whether the Hub already has a similar model, and assesses its relevance to Ersilia.

## Incorporate the model

{% stepper %}
{% step %}
#### Request the model

Open a model request issue on [ersilia-os/ersilia](https://github.com/ersilia-os/ersilia). The form asks for the model's name, a short description (up to 600 characters), a slug, tags, the publication, the source code and the license. An Ersilia maintainer reviews and approves the request, and Ersilia's bot then creates the model's repository from the model template, with its identifier, for example `eos3xhm`.

Fork that repository to your GitHub account and clone your fork. You'll do all the following steps in your clone and open the pull request from your fork.

Skill: `/model-incorporation-request` opens the issue from the paper and its code and, once the request is approved, forks and clones the repository for you.
{% endstep %}

{% step %}
#### Fill in the metadata

Complete `metadata.yml` with the information from the paper and the source repository: what the model does, its task and inputs and outputs, the publication, the source code and the license. Many fields only accept values from a fixed list, described in the [metadata.yml reference](model-template.md#the-metadata.yml-file). Check the source and license fields carefully against the original publication.

The output dimension and the interpretation must match the output columns you define in the next step, so check them again once the code is in place.

Skill: `/model-incorporation-metadata`
{% endstep %}

{% step %}
#### Add the model code

Add the model's code and checkpoints to your clone of the model repository, which Ersilia created from the [`eos-template`](https://github.com/ersilia-os/eos-template) repository, and adapt the code so the model reads Ersilia's input and writes output in the expected format. Start from the template's [`main.py`](https://github.com/ersilia-os/eos-template/blob/main/model/framework/code/main.py), a placeholder that calculates molecular weight. By the end of this step, the repository contains:

* `model/checkpoints/`, with the weights and any other data the model loads. The model must not download weights at run time. If a file is 100 MiB or larger, the folder is stored with eosvc instead of Git; see [Checkpoints and storage](model-template.md#checkpoints-and-storage).
* `model/framework/code/`, with the model's code. In `main.py`, `load_tool()` does the one-time setup (imports, reading the checkpoints, building the model) and returns the model. `run_tool(model, input_file, output_file)` reads a CSV with a `smiles` column, runs the model on that batch and writes the output CSV, with one row per input molecule in the same order. Handle molecules that can't be parsed without stopping the batch, and keep longer logic in helper modules next to `main.py`. See [The `main.py` file](model-template.md#the-main.py-file).
* `install.yml`, in the root of the repository, listing the dependencies with pinned versions. See [The `install.yml` file](model-template.md#the-install.yml-file).
* `model/framework/columns/run_columns.csv`, describing each output column with four fields: `name`, `type` (`string`, `float` or `integer`), `direction` (`high` or `low`, or empty if it doesn't apply) and `description`. It needs one row for each column that `main.py` writes, with the same name, and the number of rows must match `Output Dimension` in `metadata.yml`. See [Columns file](model-template.md#columns-file).
* `model/framework/examples/run_input.csv` and `run_output.csv`, three example molecules and the output the model gives for them. Produce `run_output.csv` by running the model, not by hand. See [Example files](model-template.md#example-files).

For example, the `run_columns.csv` of a model that returns the molecular weight:

{% code title="model/framework/columns/run_columns.csv" %}
```csv
name,type,direction,description
mol_weight,float,high,The calculated molecular weight of the molecule in g/mol
```
{% endcode %}

Skill: `/model-incorporation-code` copies the checkpoints, adapts `main.py` and writes `install.yml`, `run_columns.csv` and the example files, running the model to produce `run_output.csv`.
{% endstep %}

{% step %}
#### Test the model

Test the model the way Ersilia will run it in the Hub. From the `ersilia` environment you created in [Before you begin](#before-you-begin), run:

```bash
ersilia -v test eosXXXX --shallow --from_dir path/to/eosXXXX
```

The folder must be named exactly like the model identifier, so don't rename your clone. The first run can take several minutes, because the test builds the model's environment. It installs the model from your clone, runs it with every input and output format, checks that the metadata and the files agree, including the columns of `run_output.csv` with `run_columns.csv`, and checks that repeated runs give the same results. The same test runs automatically on your pull request.

The test prints the result of each check and saves a report, `eosXXXX-test.json`, in the folder where you run it. Don't commit it. Ignore the checks that say "not present", such as the Docker Hub and S3 links and the model size: Ersilia's workflows fill them in after the merge. If a check fails, start with the first one, because a model that doesn't run makes the others fail too, and see [Troubleshooting models](troubleshooting-models.md). The test doesn't check the values in `run_output.csv`, so regenerate that file whenever you change the code.

Skills: `/ersilia-model-test` runs the test and proposes fixes, and `/model-fixing` applies them once you confirm.
{% endstep %}

{% step %}
#### Reproduce the original results

This step is optional, but we encourage it: it's the best way to catch a bug in the incorporation before the pull request is reviewed. Check that the model, as incorporated, reproduces what the authors report. What you check depends on the model type, which is the `Task` in `metadata.yml`:

* **Annotation**, a model that gives a score or a label for each molecule. Run the same molecules through the authors' code and through the Ersilia model, and compare the predictions. Then, if the paper's test data is public, calculate the metric the paper reports, for example the AUC, and compare it with the reported value.
* **Representation**, a model that gives a vector for each molecule, such as an embedding. Run the same molecules through the authors' code and through the Ersilia model, and compare the vectors. If the paper reports how similar two named molecules are in the model's own embedding space, check that value too.
* **Sampling**, a model that generates molecules. The molecules usually differ from run to run, so you can't compare them one by one. Check instead that the generated molecules look like the paper describes: the share that are valid, unique and new is close to the reported one, and their properties, such as molecular weight or logP, are similar to those of the training set.

If the Ersilia model doesn't match the authors' code, the incorporation has a bug: fix it before you continue. If a result can't be reproduced or differs from the paper, say so in the pull request and explain why, for example that the data isn't public or that the released weights differ from the paper's.

Skill: `/model-incorporation-reproduce` detects the model type, proposes the checks for it and reports how close each result is to the original.
{% endstep %}

{% step %}
#### Optimize the model

This step is optional, and the author decides whether the model needs it. A model's image is usually much bigger than the model itself, because of the libraries it installs: packages the model never uses, packages that other packages pull in, and GPU support that a model running on a CPU doesn't need. If the dependencies in `install.yml` look heavier than they should be, you can trim them, as long as the outputs don't change. Do it before you open the pull request.

Skill: `/model-optimization` finds out which packages the model really uses when it runs, removes the ones it doesn't need from `install.yml`, and checks that the outputs stay exactly the same. It only reduces the size.
{% endstep %}

{% step %}
#### Open the pull request

Commit your work to your fork and open a pull request to the `main` branch of the model's repository. Stage only the files that belong to the model, not the test report or other generated files, and don't put `[skip ci]` or similar in a commit message, because GitHub would then skip the checks. In the description, link the model request with `Refs ersilia-os/ersilia#N` instead of `Closes`, and say what you tested, reproduced and optimized, and what you didn't.

Two automated checks run on every pull request: the same test you ran in the Test step, and a check that the pull request has a version label. A new model always takes `semver:major`; an Ersilia maintainer adds the label if you can't, see [Model versioning](model-versioning.md). Make sure both pass. An Ersilia maintainer then reviews the pull request and merges it.

Skill: `/model-incorporation-pr` commits only the intended files, asks before it pushes, opens the pull request with the `semver:major` label and a description of what was and wasn't run, and reports the checks.
{% endstep %}

{% step %}
#### Release the model

After the merge, Ersilia's workflows take over. They test the model again, upload it to S3, build and test its Docker image for AMD64 and ARM64, publish it to Docker Hub and as an SIF image, update `metadata.yml` and the Hub catalogue, and release it as `v1.0.0`. The release comes from the version label of the pull request, see [Model versioning](model-versioning.md). This usually takes a couple of hours, and longer for a slow model. We encourage you to follow the model until it is released, and then to close the model request.

Skill: `/model-incorporation-release` follows the workflows until the model is released, tells you what failed if something does, and closes the request once everything is done. It doesn't release the model itself. Ersilia maintainers also use it to file the model's paper in Ersilia's Drive, which they have to do before closing the request.
{% endstep %}
{% endstepper %}

Once a month, Ersilia maintainers use the [model-incorporation-digest](https://github.com/ersilia-os/ersilia-skills/tree/main/skills/model-incorporation-digest) skill to summarise all the models incorporated into the Hub during that month, crediting their original authors.

## Example: incorporating eos3xhm

[HADES](https://github.com/Narek-Petros-yan/HADES) ([*J. Chem. Inf. Model.*, 2026](https://doi.org/10.1021/acs.jcim.5c02953)) estimates how closely a molecule resembles an approved oral drug. This example follows its incorporation into the Hub as `eos3xhm`, step by step, and explains what each step means in chemical and in programming terms. In every step, the skill proposes the values and the contributor confirms them. The files linked below belong to the first release of the model, `v1.0.0`. The optional optimization step is not shown.

**Choose a model.** `/paper-to-model-assessment` summarizes the paper and searches the Hub for similar models. Drug-likeness models estimate how much a molecule resembles known drugs, and they are used to prioritize molecules in drug discovery. The Hub already had two of them, but HADES is trained on approved oral drugs, unlike the other two, so it adds something new and was incorporated.

**Request the model.** `/model-incorporation-request` proposes every field of the request from the paper and the source code, and opens the issue once the contributor confirms. The request for HADES is [ersilia-os/ersilia#1907](https://github.com/ersilia-os/ersilia/issues/1907). After it was approved, the repository [ersilia-os/eos3xhm](https://github.com/ersilia-os/eos3xhm) was created from the model template, and the skill forked it and cloned the fork.

**Fill in the metadata.** `/model-incorporation-metadata` proposes a value for every field of `metadata.yml` from the paper and the code, and writes them once the contributor confirms. The main fields in [`metadata.yml`](https://github.com/ersilia-os/eos3xhm/blob/v1.0.0/metadata.yml) describe what the model does and what it returns:

```yaml
Task: Annotation
Subtask: Property calculation or prediction
Output:
  - Score
Output Dimension: 1
Interpretation: Probability that a compound resembles an approved oral drug;
  values above 0.63 indicate likely oral drug-likeness.
License: MIT
```

**Add the model code.** `/model-incorporation-code` adds the authors' code and checkpoints to the template, writes `main.py`, pins the dependencies in `install.yml`, describes the output in `run_columns.csv`, and runs the model to produce the example files. For HADES, the model returns one score per molecule. Its [`run_columns.csv`](https://github.com/ersilia-os/eos3xhm/blob/v1.0.0/model/framework/columns/run_columns.csv) describes that column:

```
name,type,direction,description
hades_score,float,high,Probability that the molecule resembles an approved oral drug with 0.63 the recommended cutoff
```

The direction is `high` because a higher score means a closer resemblance to an approved oral drug. The dependencies are pinned to exact versions because a new version of a library can change the results without any error.

**Test the model.** `/ersilia-model-test` runs `ersilia test --shallow`, which runs the model on the example molecules and checks the files, the metadata and the consistency of the outputs. `/model-fixing` applies the fixes if a check fails. For HADES, every check passed.

**Reproduce the original results.** `/model-incorporation-reproduce` checks that the Ersilia model gives the same scores as the authors' code on a set of molecules, and that the metrics reported in the paper are obtained again on the paper's test set. For HADES, both checks passed. For example, the accuracy reported in the paper, 0.909, was reproduced.

**Open the pull request.** `/model-incorporation-pr` commits the work, asks before pushing, and opens the pull request with the `semver:major` version label, which marks a new model, and a description of what was tested and reproduced. For HADES, this is [ersilia-os/eos3xhm#1](https://github.com/ersilia-os/eos3xhm/pull/1).

**Release the model.** After the merge, the workflows test the model again, build its images and release it automatically. HADES was released as `v1.0.0`, the first version of every model. `/model-incorporation-release` follows the workflows until the model is released and then closes the request; Ersilia maintainers also use it to file the paper.

## Get help

If you get stuck, comment on your model request issue so an Ersilia maintainer can help. For problems with the skills themselves, open an issue on [ersilia-os/ersilia-skills](https://github.com/ersilia-os/ersilia-skills/issues).
