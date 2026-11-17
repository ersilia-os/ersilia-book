---
description: >-
  A guide to Ersilia's Claude Code skills repository: what it contains, how to
  install it, and how to contribute new skills.
---

# Ersilia Skills

`ersilia-skills` is Ersilia's curated library of Claude Code skills (reusable AI workflows built on top of Claude Code that encode our institutional knowledge into slash commands). Instead of re-explaining context to Claude each time you start a task, you invoke a skill and Claude already knows the process, the Ersilia conventions, and the expected outputs.

## What's in this repository

* `skills/` : The heart of the repository. Each subdirectory is one skill (a self-contained folder with a `SKILL.md` file that defines the workflow Claude follows when you invoke the skill). Most skills also include a `references/` folder with supporting documents (guidelines, templates, real examples, troubleshooting notes) that Claude reads as context during execution.
* `setup.sh`: A one-time installation script. When you run it, it creates symlinks from every skill directory in this repository into `~/.claude/skills/`, which is the folder Claude Code reads to discover locally installed skills. It also installs the post-merge git hook
* `.git-hooks/`: Contains a `post-merge` hook that runs automatically every time you do a `git pull`. It re-runs `setup.sh` in the background, so any new skills added to the repository since your last pull are immediately linked into your local Claude Code installation.

## Setup

{% hint style="info" %}
Prerequisites:

* Claude Code
* Git
{% endhint %}

{% stepper %}
{% step %}
### Clone ersilia-skills repo

```
git clone https://github.com/ersilia-os/ersilia-skills.git
cd ersilia-skills
```
{% endstep %}

{% step %}
### Run setup script

```
bash setup.sh
```
{% endstep %}

{% step %}
### Verify installation

Verify the installation by opening a Claude Code session and running:

```
/test-skill
```

Claude should confirm that the symlink is in place and that the setup is working correctly.
{% endstep %}
{% endstepper %}

#### Keeping skills up to date

Once setup is done, you only need to `git pull` to stay current. The post-merge hook that was installed by `setup.sh` runs `setup.sh` automatically after every pull, so new skills added to the repository will be linked into your `~/.claude/skills/` directory without any manual steps.

#### Using a skill

In any Claude Code session, type the skill name as a slash command:

```
/ersilia-model-test eos4ywv ~/models/eos4ywv
/molecule-auditing results.csv --context malaria
/event-discovery
```

Claude will load the skill's workflow and follow it. Some skills accept arguments (like a model ID or a file path); the skill's description shown in Claude Code will tell you what to provide.

## Skill Catalogue

At Ersilia, we have identified three key goals we want to make progress on, and for each one we have mapped specific skills we believe can help us get there: better public reach, Ersilia Model Hub growth, and improving scientific literacy.

The skills are still being developed and tested by the Ersilia team; you can check the ones that are ready-to-use and available on [GitHub](https://github.com/ersilia-os/ersilia-skills).

#### 1. Better public reach

| Name            | Skill                                                                                                                                         |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| stylia-plotting | How to create Python plots using the stylia package — Ersilia's matplotlib wrapper for publication-ready figures.                             |
| event-discovery | Discover interesting events for Ersilia and write a summarised report. Classify between categories (local/global, science/philanthropy, etc.) |
| html-formatting | Style any HTML into the recognisable Ersilia look and feel, and improve its UX at the same time.                                              |
| org-context     | Keeps the agent-context files short, true and consistent.                                                                                     |

#### 2. Ersilia Model Hub growth

Several of these skills are designed to be chained together as part of a larger workflow rather than run in isolation:

* **Model incorporation pipeline** — `model-incorporation-request`, `model-incorporation-metadata`, `model-incorporation-code`, `model-incorporation-reproduce`, `model-incorporation-pr`, and `model-incorporation-release` cover the full lifecycle of bringing a new model into the Hub, from opening the initial request, through verifying it reproduces the original paper’s results and submitting the pull request, to releasing it once the PR is merged. They are meant to be run in sequence, with `model-incorporation-digest` summarising the month’s completed incorporations afterwards.
* **Hub maintenance workflow** — `ersilia-model-test`, `model-monitoring`, `model-fixing`, and `github-digest` work together as a recurring maintenance loop: testing models before incorporation, monitoring the state of models and stored data, fixing what fails, and digesting GitHub activity to keep track of it all. We recommend running these as a bundled workflow rather than as standalone skills.

| Name                          | Skill                                                                                                                                                                                                                       |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| model-incorporation-request   | Open a model request issue on ersilia-os/ersilia.                                                                                                                                                                           |
| model-incorporation-metadata  | Fill in metadata.yml from the paper and source repo.                                                                                                                                                                        |
| model-incorporation-code      | Wire the model code into the Ersilia template.                                                                                                                                                                              |
| model-incorporation-reproduce | Verifies model outcomes/performance matches the original work.                                                                                                                                                              |
| model-incorporation-pr        | Commits a finished model incorporation and opens the pull request to ersilia-os/\<model\_id>. Asks for approval before pushing, reports the PR checks and merges once authorized.                                           |
| model-incorporation-release   | Last step of the incorporation pipeline. It runs after the model's PR is merged into ersilia-os/eosXXXX.                                                                                                                    |
| model-incorporation-digest    | Monthly digest of the models incorporated into the Hub, crediting each model’s original authors.                                                                                                                            |
| ersilia-model-test            | Tests an Ersilia Model Hub model before hub incorporation.                                                                                                                                                                  |
| model-fixing                  | When a model fails a test, reviews where it failed and fixes it automatically.                                                                                                                                              |
| github-digest                 | Tracks open issues and produces summaries for tech-tracking meetings.                                                                                                                                                       |
| repository-auditing           | Audits a repository to make sure it abides by Ersilia's standards.                                                                                                                                                          |
| model-monitoring              | Track pending models, stored data, etc.                                                                                                                                                                                     |
| model-optimization            | Makes an Ersilia Model Hub model smaller without changing its outputs, verifies it with exact output gates, a served-model run and ersilia test --shallow, and opens a PR on the model repository with the size comparison. |
| model-gpu                     | Adapts a model to also run on GPU: template run.sh switching on ERSILIA\_DEVICE, main\_gpu.py, and a gpu: section in install.yml.                                                                                           |
| model-speedup                 | Makes an Ersilia Model Hub model faster, and stops its memory from growing with the input, without changing its outputs; checks outputs stay identical, verifies the served model and runs ersilia test --shallow.          |

#### 3. Improve scientific literacy

| Name                      | Skill                                                                                                                                                                                                        |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| literature-review         | Given a topic, offer a structured review from the literature. This review will provide you with relevant research/review papers, alongside potential ML models and datasets that can be included in Ersilia. |
| literature-digest         | Produces a weekly literature digest for Ersilia.                                                                                                                                                             |
| molecule-auditing         | Audits small molecules suggested in Ersilia's screening and scores them according to parameters of interest.                                                                                                 |
| paper-to-model-assessment | Summarize a given paper and put it in context of Ersilia's interests.                                                                                                                                        |
| peer-reviewing            | Emulate a peer review and suggest how to address changes.                                                                                                                                                    |

## Contributing

To add a new skill:

1. Create a branch from `main`.
2. Add a new directory under `skills/` named after your skill (use lowercase and hyphens).
3. Write a `SKILL.md` file. The frontmatter must include at minimum `name` and `description`. Add `argument-hint` if the skill takes arguments, and `allowed-tools` to restrict which tools Claude can use.

{% hint style="info" %}
To make the process easier, use Anthropic's built-in `skill-creator` skill. Once you have Claude Code set up, run `/skill-creator` in your session and Claude will guide you through designing and writing a new skill from scratch, helping you structure the workflow, write the frontmatter, and identify what reference documents to include. It is the recommended starting point for anyone contributing their first skill to this repository.
{% endhint %}

4. Add a `references/` folder with any supporting documents Claude should read — guidelines, examples, templates, troubleshooting notes.
5. Open a pull request. The post-merge hook will ensure collaborators get the skill automatically after they pull
