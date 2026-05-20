---
description: Set up instructions for Claude usage with Ersilia's organisation
---

# Claude 101

Claude is an AI assistant made by Anthropic.&#x20;





## Chat



## Claude Cowork



## Claude Code

You can use it through a chat interface at claude.ai, but for technical work the most powerful way to interact with it is through **Claude Code:** a command-line tool that gives Claude direct access to your terminal, your files, and your codebase. Instead of copy-pasting code back and forth in a chat window, Claude Code lets Claude read, write, and run things in your actual working environment.

At Ersilia we use Claude Code as our primary AI interface for science, platform, and communications work.

### Installing Claude Code

### In the terminal

Open a terminal and run:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Once installed, start a session from any directory by running:

```bash
claude
```

The first time you run it, Claude Code will prompt you to log in with your Anthropic account and authorize the CLI

### In VSCode

Search for **Claude Code** in the VSCode Extensions marketplace and install it. Once installed, it appears as a panel in your sidebar. It runs the same Claude Code session you would get in the terminal, but embedded directly in your editor, but with the added benefit that Claude can see which file you have open and any code you have selected

{% hint style="info" %}
Both the terminal and the VSCode extension share the same installation and configuration, so you can use whichever feels more natural for the task at hand.
{% endhint %}

## What is a Claude code skill

A skill is a reusable workflow definition for Claude. It lives in a `SKILL.md` file and becomes a slash command you can invoke in any Claude Code session. When you run `/skill-name`, Claude loads the workflow and follows it knowing what inputs to expect, what steps to take, what tools to use, and what output to produce.

Without skills, you would need to re-explain context and process to Claude every time you start a task. Skills remove that friction by encoding the "how we do things" layer directly into the tool. &#x20;

Skills can include supporting reference documents (guidelines, templates, real examples) that Claude reads automatically as part of the workflow, so the relevant context is always loaded without you having to provide it manually.

Anthropic maintains a registry of general-purpose skills for common tasks. Ersilia maintains its own library of skills tailored to our specific work that is what the `ersilia-skills` repository is (see [ersilia skills](ersilia-skills.md))

## Installing Anthropic skills

Anthropic ships a set of built-in skills you can install directly from their registry. These are general-purpose skills for common tasks like creating new skills, building documents, designing interfaces, and more.

To install them, run the following inside a Claude Code session:

```
/find-skills
```

Claude will show you what is available and guide you through installing the ones you want. The most important one to install first is **skill-creator**, which you will use to build new Ersilia skills:

```
/skill-creator
```

This skill walks you through designing a new skill from scratch, structuring the workflow, writing the frontmatter, and deciding what reference documents to include.

## Skills: system-wide vs project-wide installation

When you install a skill, it can live in two places:

* **System-wide** skills are installed in `~/.claude/skills/`. They are available in every Claude Code session you open, regardless of which directory or repository you are working in. It is the right place for tools you use across many projects.
* **Project-wide** skills are defined inside a specific repository, typically referenced in a `.claude/settings.json` file at the root of the project. They are only active when Claude Code is running inside that project. This is useful for skills that are tightly coupled to a specific codebase or that you do not want bleeding into unrelated work.\
  <br>
