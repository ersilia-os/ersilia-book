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

At Ersilia, we have divided the skills we are developing and using into several categories, according to what we use them for.&#x20;

They are still being developed and tested by the Ersilia team, and once they are ready they will be listed and available on [GitHub](https://github.com/?utm_source=chatgpt.com).

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
