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

Add the model's code and checkpoints to the [model template](model-template.md), and adapt it so the model reads Ersilia's input and writes output in the expected format. By the end of this step, the repository contains:

* the model's checkpoints in `model/checkpoints/` and its code in `model/framework/code/`, with `main.py` running the model;
* `install.yml`, listing the dependencies with pinned versions;
* `run_columns.csv`, describing each output column;
* `run_input.csv` and `run_output.csv`, an example input and the output the model gives for it.

The model must not download weights at run time.

Skill: `/model-incorporation-code`
{% endstep %}

{% step %}
#### Test the model

Test the model the way Ersilia will run it in the Hub:

```bash
ersilia test eosXXXX --shallow --from_dir path/to/eosXXXX
```

The test installs the model from your clone, runs it with every input and output format, checks that the metadata and the output files agree, and checks that repeated runs give the same results. The same test runs automatically on your pull request. If a check fails, see [Troubleshooting models](troubleshooting-models.md).

Skills: `/ersilia-model-test`, then `/model-fixing` if a check fails
{% endstep %}

{% step %}
#### Reproduce the original results

Check that the model, as incorporated, gives the same outputs as the authors' code on the same molecules. If it doesn't, the incorporation has a bug: fix it before you continue. Then compare its performance with what the paper reports, where the data allow. If a result can't be reproduced or differs from the paper, say so in the pull request and explain why, for example that the data isn't public or that the released weights differ from the paper's.

Skill: `/model-incorporation-reproduce`
{% endstep %}

{% step %}
#### Open the pull request

Commit your work to your fork and open a pull request to the `main` branch of the model's repository. Two automated checks run on every pull request: the same test you ran in the Test step, and a check that the pull request has a version label, which an Ersilia maintainer adds; see [Model versioning](model-versioning.md). Make sure they pass. An Ersilia maintainer then reviews the pull request and merges it.

Skill: `/model-incorporation-pr`
{% endstep %}

{% step %}
#### Release the model

After the merge, Ersilia's workflows test the model again, publish it to Docker Hub and as an SIF image, and release it as `v1.0.0`. Ersilia maintainers then check that every workflow passed, file the model's paper and close the model request.

Skill: `/model-incorporation-release`
{% endstep %}
{% endstepper %}

Once a month, Ersilia maintainers use the [model-incorporation-digest](https://github.com/ersilia-os/ersilia-skills/tree/main/skills/model-incorporation-digest) skill to summarise all the models incorporated into the Hub during that month, crediting their original authors and flagging any metadata that needs fixing.

## Example: incorporating eos3xhm

[HADES](https://github.com/Narek-Petros-yan/HADES) ([*J. Chem. Inf. Model.*, 2026](https://doi.org/10.1021/acs.jcim.5c02953)) scores how closely a compound resembles an approved oral drug. This example walks through its incorporation as `eos3xhm` with the skills: for each step, what the skill does, what you should check, and what the step produced for HADES. The skills propose and you decide: only you can confirm that the model's details are correct. The files linked below are from the release `v1.0.1`.

**Choose a model.** `/paper-to-model-assessment` summarizes the paper and looks for similar models in the Hub. For HADES, it found two drug-likeness models already there. *Check:* what the new model adds. HADES is trained on approved oral drugs, unlike the other two, so we went ahead.

**Request the model.** `/model-incorporation-request` proposes every field of the request from the paper and the source code, and opens the issue once you confirm. *Check:* the name, the description, the publication and the license, against the paper. The request for HADES is [ersilia-os/ersilia#1907](https://github.com/ersilia-os/ersilia/issues/1907). Once it was approved, the repository [ersilia-os/eos3xhm](https://github.com/ersilia-os/eos3xhm) was created, and the skill forked it and cloned the fork.

**Fill in the metadata.** `/model-incorporation-metadata` proposes a value for every field of `metadata.yml`, and writes them once you confirm. *Check:* every field against the paper, including the ones you didn't write. The main fields in [`metadata.yml`](https://github.com/ersilia-os/eos3xhm/blob/v1.0.1/metadata.yml) are:

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

**Add the model code.** `/model-incorporation-code` adds the authors' code and checkpoints to the template, writes `main.py`, pins the dependencies in `install.yml`, describes the outputs in `run_columns.csv`, and runs the model to produce the example files. *Check:* that the output columns and their direction make sense, and that a molecule that can't be read gets an empty row instead of being dropped. For HADES, the checkpoints are five classifiers; the largest is 64 MB, under the 100 MiB limit, so all of them are in Git. Its [`run_columns.csv`](https://github.com/ersilia-os/eos3xhm/blob/v1.0.1/model/framework/columns/run_columns.csv) has one column:

```
name,type,direction,description
hades_score,float,high,Probability that the molecule resembles an approved oral drug with 0.63 the recommended cutoff
```

**Test the model.** `/ersilia-model-test` runs `ersilia test --shallow` and explains any check that fails; `/model-fixing` applies the fixes once you agree. For HADES, every check passed.

**Reproduce the original results.** `/model-incorporation-reproduce` compares the model with the authors' code, then recomputes the metrics the paper reports. *Check:* the reproduced numbers against the paper yourself, and that any gap is explained. For HADES, the outputs matched the authors' code on 20 molecules (Spearman correlation 1.000), and the paper's results reproduced on its 1,622-molecule test set:

| Metric | Paper | Reproduced |
|---|---|---|
| Accuracy | 0.909 | 0.9094 |
| Brier score | 0.070 | 0.0700 |
| Large-error rate (above 0.7) | 6.17% | 6.165% |

**Open the pull request.** `/model-incorporation-pr` commits the work, asks before pushing, and opens the pull request with the `semver:major` version label and a description of what was tested and reproduced. *Check:* that description, and that every automated check passes. For HADES, this is [ersilia-os/eos3xhm#1](https://github.com/ersilia-os/eos3xhm/pull/1).

**Release the model.** After the merge, the workflows release the model automatically. An Ersilia maintainer then runs `/model-incorporation-release` to check every workflow and the release, file the paper and close the request. HADES was released as `v1.0.0` the same day.

## Get help

If you get stuck, comment on your model request issue so an Ersilia maintainer can help. For problems with the skills themselves, open an issue on [ersilia-os/ersilia-skills](https://github.com/ersilia-os/ersilia-skills/issues).
