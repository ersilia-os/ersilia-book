---
description: Documentation to run models on-premises
---

# Local inference

Run Ersilia models on your own computer or cluster for large scale screenings, privacy control and accessing all the models available in the Hub. You only need to install Ersilia once and then run models through the command line, as a python package, via AI agents or as SIF images on a cluster with Apptainer.

## Quick start

If you already have conda and Docker running, these commands install Ersilia and run a model on your molecules:

```bash
conda create -n ersilia python=3.12 -y
conda activate ersilia
pip install ersilia
ersilia fetch eos3b5e
ersilia serve eos3b5e
ersilia run -i input.csv -o output.csv
ersilia close
```

For a step-by-step setup, keep reading.

## Before you begin

* **Operating system:** Ersilia runs on Linux and macOS. On Windows, use the [Windows Subsystem for Linux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/install).
* **Python:** Ersilia supports Python 3.10 and above. The commands on this page use Python 3.12.
* **Conda:** Install [Miniconda](https://www.anaconda.com/docs/getting-started/miniconda/install) or [Anaconda](https://www.anaconda.com/docs/getting-started/anaconda/install), following the instructions for your operating system.
* **Docker:** [Install Docker](https://docs.docker.com/get-docker/) and make sure it's running before you use Ersilia. Models are distributed as Docker images on Docker Hub.

## Install Ersilia

Create a separate conda environment for Ersilia, then install it from PyPI or conda-forge.

{% tabs %}
{% tab title="PyPI" %}
```bash
conda create -n ersilia python=3.12
conda activate ersilia
pip install ersilia
```
{% endtab %}

{% tab title="conda-forge" %}
```bash
conda create -n ersilia python=3.12
conda activate ersilia
conda install -c conda-forge ersilia
```
{% endtab %}
{% endtabs %}

To check that the installation worked, run:

```bash
ersilia --help
```

## Prepare your input

Ersilia models take chemical structures as input, written as SMILES strings. To find the SMILES string for a compound, search for it on [PubChem](https://pubchem.ncbi.nlm.nih.gov/).

Save your molecules in a CSV file with a single column and a header:

{% code title="input.csv" %}
```
smiles
C1=C(SC(=N1)SC2=NN=C(S2)N)[N+](=O)[O-]
CC(C)CC1=CC=C(C=C1)C(C)C(=O)O
```
{% endcode %}

## Run models

Each model has an identifier, such as `eos3b5e`. Find the model you need and its identifier in the [Ersilia catalog](https://catalog.ersilia.io/).

{% tabs %}
{% tab title="Command line" %}
{% stepper %}
{% step %}
## Fetch

Download the model and its dependencies to your machine.

```bash
ersilia fetch eos3b5e
```

{% hint style="warning" %}
Ersilia downloads models from Docker Hub, so make sure Docker is running before you fetch a model.
{% endhint %}
{% endstep %}

{% step %}
## Serve

Start the model so it's ready to accept inputs.

```bash
ersilia serve eos3b5e
```

{% hint style="info" %}
If you serve a model you haven’t fetched yet, Ersilia asks whether to fetch it first.
{% endhint %}
{% endstep %}

{% step %}
## Run

Pass your input file through the model and save the predictions.

```bash
ersilia run -i input.csv -o output.csv
```
{% endstep %}

{% step %}
## Close

Stop the model to free up resources on your machine.

```bash
ersilia close
```
{% endstep %}
{% endstepper %}
{% endtab %}

{% tab title="Python" %}
The Python API mirrors the command line steps:

```python
from ersilia.api import Model

input_list = [
    "C1=C(SC(=N1)SC2=NN=C(S2)N)[N+](=O)[O-]",
    "CC(C)CC1=CC=C(C=C1)C(C)C(=O)O"
]

mdl = Model(model="eos3b5e")
mdl.fetch()
mdl.serve()
df = mdl.run(input_list)
mdl.close()
```

`run` returns the predictions as a pandas DataFrame.
{% endtab %}

{% tab title="AI agent" %}
The [Ersilia MCP server](https://github.com/ersilia-os/ersilia-mcp) lets an AI agent, such as Claude, search the catalog and fetch, serve, and run models for you. You describe what you need in plain language, and the agent does the rest on your machine.

For example, you could ask:

> Find a model that predicts antimalarial activity and run it on the molecules in input.csv.

The agent searches the catalog, picks a model, downloads and serves it, runs your molecules through it, saves the predictions as a CSV file, and closes the model when it's done.

#### Set up the MCP server

The Ersilia MCP server needs conda, Docker, and git.

```bash
git clone git@github.com:ersilia-os/ersilia-mcp.git
cd ersilia-mcp
conda create -n ersilia-mcp python=3.12
conda activate ersilia-mcp
pip install poetry
poetry config virtualenvs.create false --local
poetry env use $(which python)
poetry install
```

If you use Claude Code, the server registers automatically when you start Claude Code inside the `ersilia-mcp` folder. For other clients, such as Gemini, follow the [setup guides in the repository](https://github.com/ersilia-os/ersilia-mcp/tree/main/docs/mcp_setup).
{% endtab %}

{% tab title="HPC (Apptainer)" %}
Every model is also published as a SIF image, which you can run with [Apptainer](https://apptainer.org/) on HPC clusters where Docker isn't available. We provide SIF images for the latest version of the model only. Use the link in the model metadata to obtain the image directly from S3.

{% hint style="info" %}
Check with your cluster administrators that Apptainer is installed, and run models through your cluster's job scheduler, such as Slurm, rather than on a login node.
{% endhint %}
{% endtab %}
{% endtabs %}

## Understand model outputs

Ersilia hosts many kinds of models, so the output depends on the model you run. Every output file contains at least two columns:

* `key`: a 32-character unique identifier that Ersilia creates for each molecule.
* `input`: the input SMILES string. Ersilia might standardize SMILES strings that weren't in standard format.

The remaining columns contain the model's predictions. To learn what each column means, what type of data it holds, and how to interpret its values, check the model's documentation in either of these places:

* The README file in the model's GitHub repository, for example [ersilia-os/eos3b5e](https://github.com/ersilia-os/eos3b5e).
* The `run_columns.csv` file in the model's repository, at `model/framework/columns/run_columns.csv`, which defines every output column.

To make sense of the results of several models at once, you can use the [molecule-auditing](https://github.com/ersilia-os/ersilia-skills/tree/main/skills/molecule-auditing) skill for Claude Code. Given a table of molecules with model predictions, it helps you interpret the columns, filter and prioritise the molecules, and produces an interactive report with their 2D structures.

## Get help

If something doesn't work, [open an issue on GitHub](https://github.com/ersilia-os/ersilia/issues) and include the command you ran and the error message.
