# Maintenance skills

The Ersilia team uses [skills for Claude Code](https://github.com/ersilia-os/ersilia-skills) for much of its maintenance work. Each skill describes a task step by step, so Claude can carry it out and check the result. This page summarizes what each one does. Each skill's `SKILL.md` file in the repository describes it in full, and can also be followed by hand.

To set up the skills, see [Claude at Ersilia](../../../agentic-ai/ersilia-skills.md).

## Monitor the Hub

<table><thead><tr><th width="231.3828125">Skill</th><th>What it does</th></tr></thead><tbody><tr><td><code>model-monitoring</code></td><td>Produces an HTML report on the state of the Hub: which models have their predictions stored in the <a href="../data-storage/precalculation-store.md">precalculation store</a>, which have an <a href="../packaging-and-distribution/apptainer-singularity-images.md">Apptainer image</a>, and the week's activity in the maintenance repository.</td></tr><tr><td><code>github-digest</code></td><td>Produces a report on issue and pull request activity across Ersilia's repositories, highlights what needs attention, such as stale pull requests and long-open issues, and checks that the repository registry in Airtable is up to date.</td></tr></tbody></table>

## Test and fix models

These two skills work together:

<table><thead><tr><th width="208.9765625">Skill</th><th>What it does</th></tr></thead><tbody><tr><td><code>ersilia-model-test</code></td><td>Runs <code>ersilia test --shallow</code> on a local copy of a model, reads the results, identifies the failing checks, and proposes how to fix them. It doesn't change anything.</td></tr><tr><td><code>model-fixing</code></td><td>Takes the failures found by <code>ersilia-model-test</code>, agrees a plan with you, and applies the fixes to the model's files (<code>main.py</code>, <code>metadata.yml</code>, <code>install.yml</code>, and <code>run_columns.csv</code>). It then tests the model again and checks that its example output hasn't changed.</td></tr></tbody></table>

## Improve models

These skills change a model without changing its outputs. Each one takes the model's identifier and the path to a local copy:

<table><thead><tr><th width="231.2890625">Skill</th><th>What it does</th></tr></thead><tbody><tr><td><code>model-speedup</code></td><td>Makes a model faster, and stops its memory use from growing with the number of inputs. It measures the model, finds where the time goes, applies known fixes, and checks the outputs stay the same.</td></tr><tr><td><code>model-optimization</code></td><td>Makes a model smaller, by finding and removing the packages it doesn't need at run time, such as GPU builds or unused dependencies. It checks the outputs after every change.</td></tr><tr><td><code>model-gpu</code></td><td>Adapts a model so it can also run on a GPU, on a separate <code>gpu</code> branch, without changing how it runs on CPU.</td></tr></tbody></table>

For example:

```
/model-speedup eos3b5e ~/models/eos3b5e
```

{% hint style="info" %}
Skills prepare changes, but a person reviews them. Check the results before you open a pull request, as you would with any other change.
{% endhint %}
