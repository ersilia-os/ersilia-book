---
description: Documentation for those who want to contribute new models to the Hub
---

# Model contribution

Anyone can contribute a model to the Ersilia Model Hub. If you're a researcher who has published a model, or you've found one in the literature that would help infectious and neglected disease research, you can incorporate it so that others can run it with a single command.

At Ersilia we use the [literature-review](https://github.com/ersilia-os/ersilia-skills/tree/main/skills/literature-review) skill for Claude Code to help us find models worth incorporating. Given a disease, target or method, it reviews recent literature and highlights models and datasets not yet in the Hub, helping us decide which ones to incorporate next.

Every model goes through the same steps: request it, fill in its metadata, add its code to the model template, test it, reproduce the original results, and open a pull request. You can do them by hand, or with Ersilia's skills for Claude Code, which automate most of the work. Either way, the decisions are yours: whether a model is worth incorporating, whether its results are reproduced faithfully, and whether the final pull request is right.

### In this section

* [Model template](model-template.md): what each file and folder in a model repository is for, and the rules it must follow.
* [Model incorporation workflow](model-incorporation-workflow.md): the steps to incorporate a model, with a worked example. Start here.
* [Incorporate a model manually](https://ersilia.gitbook.io/ersilia-book/ersilia-book/ersilia-book-archive/ersilia-model-hub/model-incorporation-workflow), in the Ersilia Book Archive: every step done by hand, for contributors who don't use the skills.
* [Troubleshooting models](troubleshooting-models.md): what to do when a model fails to fetch, run, or pass its tests.
* [Model versioning](model-versioning.md): how models are versioned and released, and which version label a pull request needs.

### Get help

If you get stuck, comment on your model request issue so an Ersilia maintainer can help.
