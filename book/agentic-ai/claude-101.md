---
description: Set up instructions for Claude usage with Ersilia's organisation
---

# Claude 101

Claude is an AI assistant made by Anthropic. There are several ways to interact with it, and at Ersilia we use three of them depending on the task. The web chat at [claude.ai](https://claude.ai) is the simplest entry point and works well for quick questions, drafting, and research. Claude Cowork is a desktop application aimed at file and operations work, where Claude can read and edit files on your computer, connect to apps like Airtable or Slack, and follow custom workflows. Claude Code is a command-line tool that gives Claude direct access to your terminal and your codebase, and is what we use for science and platform engineering.

This page covers what each one is and how we use them.

## Chat

The simplest way to interact with Claude is through the chat interface at [claude.ai](https://claude.ai). It works like any chat application: you type a message and Claude replies as a chatbot. You can upload files (PDFs, images, spreadsheets, code snippets, screenshots) for Claude to read and discuss, and you can have long, multi-turn conversations to refine an idea or work through a problem. Each conversation has its own memory but does not touch your computer or any of our systems.

At Ersilia we use Claude Chat as the default tool for general-purpose tasks that do not require Claude to act on our files or accounts: drafting and editing text, summarising papers, brainstorming, translating, sanity-checking an analysis, or exploring an idea before committing to it. It is the right tool when you want a thinking partner and a fast answer, not an agent that runs in the background.

Aquí va una versión mucho más visual, con iconos y comparación rápida tipo "ficha":

#### When to use it

Use **Claude Chat** when you want a fast answer or a thinking partner and Claude does not need to touch any of our systems. Examples: rewriting a paragraph, summarising a paper you paste in, brainstorming a workshop agenda, asking a quick scientific or technical question, translating a piece of text.

#### How to install it

To use it, sign in at [claude.ai](https://claude.ai) with your Google account.

## Claude Cowork

Claude Cowork is a desktop application that turns Claude into an agent that works alongside you on your computer. Unlike the chat interface, Cowork has access to a workspace folder on your machine, a sandboxed Linux shell, and a set of connectors to external apps (Airtable, Slack, Google Drive, GitHub, Notion, and many others). Within those boundaries Claude can read and write files, run code, query connected services, and produce real deliverables — Word documents, spreadsheets, slide decks, PDFs — that land directly in a folder you choose.

Cowork also supports **plugins** and **skills**: reusable workflow definitions that encode "how we do things" so you do not need to re-explain context every time. A skill is invoked from the chat and tells Claude exactly which steps to take, which tools to use, and what to produce. Ersilia maintains its own plugin with skills tailored to our operations work.

At Ersilia we use Cowork as the main interface for non-developer work that lives outside the codebase. The current set of Ersilia skills covers monitoring our funder pipeline, classifying and registering new organisations in Airtable and running the weekly grants review. Each skill is connected to the right Airtable bases and follows our internal taxonomy, so a short request like "weekly funder update" produces a full, consistent output, using the correspondent skills.

#### How to install it

To get started, download the Claude desktop app from [anthropic.com](https://www.anthropic.com), sign in with your Google account, and enable Cowork mode from the app menu. Then install the Ersilia plugin to load our skills and connectors automatically.

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

### What to use

Aquí va una versión mucho más visual, con iconos y comparación rápida tipo "ficha":

***

|                                                |   💬 **Chat**  |                         🖥️ **Cowork**                         |            ⌨️ **Code**           |
| ---------------------------------------------- | :------------: | :------------------------------------------------------------: | :------------------------------: |
| **Where**                                      |    claude.ai   |                           Desktop app                          |         Terminal / VSCode        |
| **Touches your files**                         |        ❌       |                                ✅                               |                 ✅                |
| **Connects to apps** (Airtable, Slack, Drive…) |        ❌       |                                ✅                               |                 ✅                |
| **Runs code**                                  |        ❌       |                           ✅ (sandbox)                          |           ✅ (your mach           |
|                                                |        —       | 📊 Funders · 💸 Grants · 📰 Newsletter · 🔗 LinkedIn · 🏢 Orgs | 🧬 Hub · 📦 Models · 📈 `stylia` |
| **Output**                                     |  💭 An answer  |                     📄 A file / app update                     |            🔧 A commit           |
| **Best for**                                   | Quick thinking |                         Operations work                        |         Engineering work         |
| **Who uses it most**                           |    Everyone    |                     Comms, fundraising, ops                    |       Scientists, engineers      |

***

#### Rule of thumb

> 💭 **Idea or sentence?** → Chat 📄 **File or app update?** → Cowork 🔧 **Code commit?** → Code

***

Si quieres llevarlo aún más lejos, puedo prepararte una imagen SVG con tres tarjetas (Chat / Cowork / Code) para incrustar en GitBook como diagrama. ¿La hago?
