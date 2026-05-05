---
description: >-
  A guide to Ersilia's Claude Code skills repository: what it contains, how to
  install it, and how to contribute new skills.
---

# Ersilia Skills

`ersilia-skills` is Ersilia's curated library of Claude Code skills (reusable AI workflows built on top of Claude Code that encode our institutional knowledge into slash commands). Instead of re-explaining context to Claude each time you start a task, you invoke a skill and Claude already knows the process, the Ersilia conventions, and the expected outputs.

## What's in this repository

* `skills/` : The heart of the repository. Each subdirectory is one skill (a self-contained folder with a `SKILL.md` file that defines the workflow Claude follows when you invoke the skill). Most skills also include a `references/` folder with supporting documents (guidelines, templates, real examples, troubleshooting notes) that Claude reads as context during execution.&#x20;
* `setup.sh`: A one-time installation script. When you run it, it creates symlinks from every skill directory in this repository into `~/.claude/skills/`, which is the folder Claude Code reads to discover locally installed skills. It also installs the post-merge git hook
* `.git-hooks/`: Contains a `post-merge` hook that runs automatically every time you do a `git pull`. It re-runs `setup.sh` in the background, so any new skills added to the repository since your last pull are immediately linked into your local Claude Code installation.

## Setup

{% hint style="info" %}
Prerequisites:

* Claude Code&#x20;
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
### Verify installation&#x20;

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
/newsletter-drafting
```

Claude will load the skill's workflow and follow it. Some skills accept arguments (like a model ID or a file path); the skill's description shown in Claude Code will tell you what to provide.

## Skill Catalogue

<table><thead><tr><th width="235.3671875">Skill name</th><th>Description</th></tr></thead><tbody><tr><td>ersilia-metadata</td><td>Fills in the fields of a model's <code>metadata.yml</code> file. Given a publication PDF and a source code repository, it extracts the relevant information and populates fields like Task, Output, Biomedical Area, and Target Organism. It never touches fields it is not supposed to modify.</td></tr><tr><td>ersilia-model-test<br></td><td>Runs <code>ersilia test &#x3C;model_id> --shallow</code> on a locally cloned model repository, reads the JSON output, groups failures by category (run errors, metadata issues, file structure, consistency), and proposes debugging strategies. It reports findings without auto-applying fixes</td></tr><tr><td>model-incorporation-code<br></td><td>Guides the full process of integrating a source model's code into the Ersilia model template. This is a multi-phase workflow: understanding the source model, handling checkpoints and large files with git-lfs, adapting <code>main.py</code> to use <code>ersilia_pack_utils</code>, creating <code>run_columns.csv</code>, pinning dependencies in <code>install.yml</code>, and generating example input/output files by actually running the model</td></tr><tr><td>molecule-auditing</td><td>Audits a CSV of molecules from an Ersilia screening run. It fetches model metadata from GitHub, scores molecules by activity, drug-likeness, and ADMET properties, and produces a structured Markdown report with a prioritized candidate list (Promising / Borderline / Deprioritise). Designed to handle large CSVs efficiently without loading the full file into memory.</td></tr><tr><td>newsletter-drafting</td><td>Writes Ersilia's monthly newsletter content from a summary of the month's events. It produces six ready-to-paste content blocks: opening paragraph, long story, short story, embedded story, where-to-find-us section, and footer tagline. It follows Ersilia's tone and style guidelines</td></tr><tr><td>social-media-post-drafting</td><td>Drafts LinkedIn posts and newsletter content. At the start of the month it takes a content calendar and produces a posting schedule with drafted posts. At the end of the month it generates the newsletter content blocks from an event summary. It enforces Ersilia's communication tone: direct, specific, no jargon</td></tr><tr><td>stylia-plotting</td><td>Creates publication-ready Python figures using Ersilia's <code>stylia</code> matplotlib wrapper. It enforces the correct API usage( <code>stylia.create_figure()</code>, <code>stylia.save_figure()</code>, <code>stylia.label())</code> and selects colors, sizes, and layouts following stylia's conventions. </td></tr><tr><td>test-skill</td><td>A minimal skill that verifies your local setup is working. It confirms the symlink is in place and reports success. Run this after installation to make sure everything is connected</td></tr></tbody></table>

## Contributing

To add a new skill:

1. Create a branch from `main`.
2. Add a new directory under `skills/` named after your skill (use lowercase and hyphens).
3. Write a `SKILL.md` file. The frontmatter must include at minimum `name` and `description`. Add `argument-hint` if the skill takes arguments, and `allowed-tools` to restrict which tools Claude can use.

{% hint style="info" %}
To make the process easier, use Anthropic's built-in `skill-creator` skill. Once you have Claude Code set up, run `/skill-creator` in your session and Claude will guide you through designing and writing a new skill from scratch, helping you structure the workflow, write the frontmatter, and identify what reference documents to include. It is the recommended starting point for anyone contributing their first skill to this repository.
{% endhint %}

1. Add a `references/` folder with any supporting documents Claude should read — guidelines, examples, templates, troubleshooting notes.
2. Open a pull request. The post-merge hook will ensure collaborators get the skill automatically after they pull
