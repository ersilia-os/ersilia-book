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

At Ersilia, we have divided the skills we are developing and using into several categories, according to what we use them for.<br>

### Visibility

Related to social media, Ersilia’s public image, and networking.

| Name            | Skill                                                                                                                                         |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| event-discovery | Discover interesting events for Ersilia and write a summarised report. Classify between categories (local/global, science/philanthropy, etc.) |
| branding        | Convert documents, including slides, diagrams, posters, into Ersilia branded formats                                                          |

### Platform

Related to the Ersilia Model Hub: onboarding, fixing, and maintaining models.

| Name                | Skill                                                                 |
| ------------------- | --------------------------------------------------------------------- |
| model-incorporation | Incorporate model into the Ersilia Model Hub and test it              |
| model-discovery     | Review the literature in search for models and datasets of interest   |
| issue-tracking      | Track open issues, produce summaries for tech tracking meetings, etc. |
| model-monitoring    | Track pending models, stored data, etc                                |
| model-fixing        | When a model fails test, review where and fix it automatically        |

### Science

Skills that support Ersilia’s scientific work: literature suggestions, summaries, and advice on written articles.

| Name              | Skill                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| literature-review | Given a topic, offer a structured review from the literature                                            |
| paper-summary     | Summarize a given paper and put it in context of Ersilia’s interests                                    |
| peer-reviewing    | Emulate a peer review and suggest how to address changes                                                |
| literature-digest | Produce a weekly literature digest for Ersilia                                                          |
| molecule-auditing | Audit small molecules suggested in Ersilia screening and score them according to parameters of interest |

### Day-to-day

Skills that help streamline the everyday and recurring tasks of Ersilia members, such as preparing internal presentations and organizing calendars.

| Name                   | Skill                                                                                                         |
| ---------------------- | ------------------------------------------------------------------------------------------------------------- |
| slack-summaries        | Produce a summary of recent Slack activity and prioritise action items                                        |
| calendars              | Check for availability, suggest timeslots, book rooms in Norrsken, create meeting invites and Calendar events |
| time-tracking          | Check how you’ve been spending time (including meetings), alignment with objectives, etc.                     |
| meeting-minutes        | Take meeting minutes dynamically, aware of agendas, etc.                                                      |
| internal-presentations | <p>Create Google Slides from an Ersilia's template using the information we provide him<br></p>               |

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
