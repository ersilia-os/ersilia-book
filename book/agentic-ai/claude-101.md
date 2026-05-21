---
description: Set up instructions for Claude usage with Ersilia's organisation
---

# Claude 101

Claude is an AI assistant made by Anthropic. It can read, write, reason, run code, and work with the apps and files you use every day. At Ersilia we rely on Claude across science, engineering, communications, and operations.

There are three ways to interact with Claude, and at Ersilia we use all three depending on the task:

* **Claude Chat** at [claude.ai](https://claude.ai) is the simplest entry point and works well for quick questions, drafting, and research.
* **Claude Cowork** is a desktop application aimed at file and operations work, where Claude can read and edit files on your computer, connect to apps like Airtable or Slack, and follow custom workflows.
* **Claude Code** is a command-line tool that gives Claude direct access to your terminal and your codebase, and is what we use for science and platform engineering.

All three interfaces share a key concept that makes Claude actually useful for our day-to-day work: **skills**. Skills can be used in Chat as well, but because a skill usually encodes a recurring task, they are most often used in Cowork or Code, where Claude is acting on our files and systems. Before going into each interface, the next section explains what a skill is and why it matters.

## Claude Skills

A skill is a reusable workflow definition for Claude. It lives in a **SKILL.md** file that tells Claude what inputs to expect, what steps to take, what tools to use, and what output to produce for a given task. Skills can also include supporting reference documents — guidelines, templates, real examples — that Claude reads automatically as part of the workflow, so the relevant context is always loaded without you having to provide it manually.

Without skills, you would need to re-explain context and process to Claude every time you start a task. Skills remove that friction by encoding the "_how we do things_" layer directly into the tool.&#x20;

The same skill works in Chat, Cowork, and Code. A skill written once works across all three Claude interfaces — Chat, Cowork, and Code — because the `SKILL.md` format is identical everywhere; what changes is how the skill is distributed and how you invoke it (a slash command in Code, natural language in Cowork, an attached file in Chat).

Anthropic maintains a registry of general-purpose skills for common tasks. Ersilia maintains its own library of skills tailored to our specific work — that is what the ersilia-skills repository is (see [ersilia-skills](https://docs.google.com/document/d/1s9O-mjzwREH5DODOfzJqQJiqGXZyLXqLxLlbUcM22hg/edit)).

### Installing Skills in Claude Chat

To install a skill in **Claude Chat**, open [claude.ai](https://claude.ai/) and go to _Settings → Customize → Skills_. Click _Create skill_ (or _Upload skill_) and either upload the skill folder as a `.zip` — containing the `SKILL.md` file and any supporting reference documents — or paste the contents directly into the editor. Once saved, the skill is added to your account and stays available across all future conversations.

### Installing Skills in Claude Code

To install a skill in **Claude Cowork**, open the desktop app and go to _Customize → Explore plugins_. From this screen you can upload an individual skill (a folder containing the `SKILL.md` file and any supporting reference documents), create one from scratch, or install a skill published by Anthropic. Once added, the skill becomes available across all your Cowork sessions — you do not need to enable it per conversation. To use it, just describe the task in natural language and Claude will pick the right skill based on its description.

In practice, however, skills are rarely installed one by one in Cowork. Most of the time they come bundled inside a **plugin**: a single installable package that groups together a set of skills, the connectors they need (MCPs like Airtable, Slack or Drive), and any slash commands the team has defined. You can install plugins from your organization or Antropic from _Customize → Explore plugins_ and it gives you the whole setup in one step instead of uploading each skill and configuring each connector separately. You can also use the tool in _Customize → Create plugins to_ create plugins with Claude or uploaded them from your computer or the marketplace.

#### Installing Anthropic skills

Anthropic ships a set of built-in skills you can install directly from their registry. These are general-purpose skills for common tasks like creating new skills, building documents, designing interfaces, and more.

To install them, check the available skills in the Anthropic [repository](https://github.com/anthropics/claude-plugins-official) and  run the following inside a Claude Code session:

```
/plugin install skill-creator@claude-plugins-official
```

The skill-creator for example will guide you in creating a new skill from scratch, structuring the workflow, writing the front matter, and deciding what reference documents to include.

```
/skill-creator
```

#### Skills: system-wide vs project-wide installation

When you install a skill, it can live in two places:

* **System-wide** skills are installed in `~/.claude/skills/`. They are available in every Claude Code session you open, regardless of which directory or repository you are working in. It is the right place for tools you use across many projects.
* **Project-wide** skills are defined inside a specific repository, typically referenced in a `.claude/settings.json` file at the root of the project. They are only active when Claude Code is running inside that project. This is useful for skills that are tightly coupled to a specific codebase or that you do not want bleeding into unrelated work.

Once a skill has been created, it can be used across the different ways of interacting with Claude.&#x20;

## Claude Chat

The simplest way to interact with Claude is through the chat interface at [claude.ai](https://claude.ai). It works like any chatbot. You can upload files (PDFs, images, spreadsheets, code snippets, screenshots) for Claude to read and discuss, and you can have long, multi-turn conversations to refine an idea or work through a problem. Each conversation has its own memory but does not touch your computer or any of our systems.

At Ersilia we use Claude Chat as the default tool for general-purpose tasks that do not require Claude to act on our files or accounts, such as drafting and editing text.

#### When to use it

Use Claude Chat when you want a fast answer or a thinking partner and Claude does not need to touch any of our systems.&#x20;

#### How to install it

To use it, sign in at [claude.ai](https://claude.ai) with your Google account.

#### How to invoke a skill

In Claude Chat, you can attach a skill file or paste its contents to apply the workflow to that conversation.

## Claude Cowork

Claude Cowork is a desktop application that turns Claude into an agent that works alongside you on your computer. Cowork can — if you grant it permission — access a workspace folder on your machine, a sandboxed Linux shell, and a set of connectors to external apps (Airtable, Slack, Google Drive, GitHub, Notion, and many others). You connect external apps one by one under _Customize → Connectors_. You can revoke any of these at any time from the same screens. Within those boundaries Claude can read and write files, run code, query connected services, and produce real deliverables — Word documents, spreadsheets, slide decks, PDFs — that land directly in a folder you choose.

Cowork also supports reusable workflow definitions by invoking a skill so you do not need to re-explain context every time.

At Ersilia we use Cowork as the main interface for non-developer work that lives outside the codebase. The current set of Ersilia skills covers monitoring our funder pipeline, classifying and registering new organisations in Airtable and running the weekly grants review. Each skill is connected to the right Airtable bases and our Drive and follows our internal taxonomy, so a short request like "weekly funder update" produces a full, consistent output, using the correspondent skills.

#### When to use it

Use Claude Cowork when your task involves files on your computer or data in our connected apps (Airtable, Slack, Drive…) and the output is a document, spreadsheet, or an update written into one of our systems.

#### How to install it

To get started, download the Claude desktop app from [anthropic.com](https://www.anthropic.com), sign in with your Google account, and enable Cowork mode from the app menu. Then install the Ersilia plugin to load our skills and connectors automatically.

#### How to invoke a Skill

In Claude Cowork, you do not type the skill name. You describe the task in natural language ("draft the May newsletter", "weekly funder update") and Claude automatically picks the right skill based on its description.

## Claude Code

You can use it through a chat interface at claude.ai, but for technical work the most powerful way to interact with it is through **Claude Code:** a command-line tool that gives Claude direct access to your terminal, your files, and your codebase. Instead of copy-pasting code back and forth in a chat window, Claude Code lets Claude read, write, and run things in your actual working environment.

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
Both the terminal and the VSCode extension share the same installation and configuration, so you can use whichever feels more natural for the task at hand.<br>
{% endhint %}

#### When to use it

Use Claude Code when you are working inside a codebase — writing, reviewing, debugging, or running code — and you want Claude to act directly in your terminal or editor instead of through a chat window.

#### How to invoke a skill

In Claude Code, skills are invoked as slash commands: typing `/skill-name` loads the workflow.

### What to use in each case

|                        |    **Chat**    |        **Cowork**       |        **Code**       |
| ---------------------- | :------------: | :---------------------: | :-------------------: |
| **Where**              |    claude.ai   |       Desktop app       |   Terminal / VSCode   |
| **Touches your files** |        ❌       |            ✅            |           ✅           |
| **Connects to apps**   |        ❌       |            ✅            |           ✅           |
| **Runs code**          |        ❌       |       ✅ (sandbox)       |    ✅ (your machine)   |
| **Output**             |    An answer   |   A file or app update  |        A commit       |
| **Best for**           | Quick thinking |     Operations work     |    Engineering work   |
| **Who uses it most**   |    Everyone    | Comms, fundraising, ops | Scientists, engineers |

