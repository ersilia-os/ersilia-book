# Model incorporation workflow

Anyone can contribute a model to the Ersilia Model Hub. If you're a researcher who has published a model, or you've found one in the literature that would help infectious and neglected disease research, you can incorporate it so that others can run it with a single command.

Every incorporation goes through the same steps, described on this page. You can do them in two ways:

* **Manually**, following the detailed instructions in the [archive](https://ersilia.gitbook.io/ersilia-book/archive) .
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

* Optionally, [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) with the `ersilia-skills` repository set up, if you want to use the skills. See [Ersilia Skills](/broken/pages/f558538cd70b037f277348c75c5d46289ad8f2bd).

{% hint style="info" %}
If the model's checkpoints include a file of 100 MiB or more, or if you include training code or data, those files are stored in S3 with `eosvc`, which requires AWS credentials. External contributors don't have these, so an Ersilia maintainer will help you with this step. Mention it in your model request. See [Checkpoints and storage](/broken/pages/d13966e6015d8cbed272b3de577e7f62dd036e97#checkpoints-and-storage).
{% endhint %}

## Choose a model

Good candidates are published, open source models relevant to infectious or neglected disease research, with code and trained weights available and a license that allows redistribution. Before you start, follow the original authors' instructions to run the model on your machine, and note its dependencies.

## Incorporate the model

{% stepper %}
{% step %}
#### Request the model

Open a model request issue on [ersilia-os/ersilia](https://github.com/ersilia-os/ersilia), describing the model and its source. An Ersilia maintainer reviews it and, once it's approved, creates the model's repository with its identifier, for example `eos2r5a`.

Skill: `/model-incorporation-request`
{% endstep %}

{% step %}
#### Fill in the metadata

Complete `metadata.yml` with the information from the paper and the source repository. Check the source and license fields carefully against the original publication.

Skill: `/model-incorporation-metadata`
{% endstep %}

{% step %}
#### Add the model code

Add the model's code and checkpoints to the [model template](/broken/pages/d13966e6015d8cbed272b3de577e7f62dd036e97), and adapt it so the model reads Ersilia's input and writes output in the expected format. The model must not download weights at run time.

Skill: `/model-incorporation-code`
{% endstep %}

{% step %}
#### Reproduce the original results

Run the model and check that its outputs and performance match the original work. This is the most important check: don't continue if the results don't match the paper.

Skill: `/model-incorporation-reproduce`
{% endstep %}

{% step %}
#### Test the model

Test the model the way Ersilia will run it in the Hub. If a test fails, see [Troubleshooting models](/broken/pages/f63e9e4283e461deb51e3818472d0b7b0b65ee8d).

Skill: `/ersilia-model-test`
{% endstep %}

{% step %}
#### Open the pull request

Commit your work and open a pull request to the model's repository. Automated checks run on every pull request, so make sure they all pass.

Skill: `/model-incorporation-pr`
{% endstep %}

{% step %}
#### Release the model

After an Ersilia maintainer merges the pull request, the model is released and published to GitHub, Docker Hub, and as an SIF image. Ersilia maintainers run this step.

Skill: `/model-incorporation-release`
{% endstep %}
{% endstepper %}

## Example: incorporating eos2r5a

## Get help

If you get stuck, comment on your model request issue so an Ersilia maintainer can help. For problems with the skills themselves, open an issue on [ersilia-os/ersilia-skills](https://github.com/ersilia-os/ersilia-skills/issues).
