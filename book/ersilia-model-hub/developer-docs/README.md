---
description: >-
  In-depth documentation of the Ersilia Model Hub to help developers contribute
  to our open source platform.
---

# For developers

This chapter is mainly intended to developers who want to contribute to Ersilia's infrastructure. If you intend to contribute AI/ML models, please refer to the [Model contribution](../model-contribution/) chapter.

## The Ersilia CLI

The main codebase of Ersilia is the [Ersilia CLI](https://github.com/ersilia-os/ersilia). For a full reference of all commands available, please visit [this page](command-line-interface.md). A detailed reference of the Ersilia API can be accessed [here](https://ersilia-os.github.io/ersilia).&#x20;

To start contributing to Ersilia, please [fork our master branch](https://github.com/ersilia-os/ersilia/fork) and work on it locally. We recommend installing Ersilia in editable mode inside a Conda environment:

```bash
conda create -n ersilia python=3.12
conda activate ersilia
git clone https://github.com/your_username/ersilia
cd ersilia
pip install -e . 
```

### Working with models as Docker containers

All models incorporated in Ersilia are [dockerized](https://hub.docker.com/u/ersiliaos) for easy deployment. While the dockerization step happens as part of our [CI/CD workflows](ci-cd-workflows.md), it is recommended to install Docker for model testing purposes.

{% hint style="success" %}
Running models as Docker container is the recommended way since it maximizes interoperability across systems and persistency.
{% endhint %}

### Working with models from source

Ersilia models can also be packaged from source. The source code and parameters for a given model are available in its corresponding GitHub repository. Checkpoints and training data are stored in the model repository, except a folder that holds a file larger than 100 MB, which is stored in S3 with [eosvc](https://github.com/ersilia-os/eosvc) instead. Some older models keep large checkpoints in [git-lfs](https://git-lfs.com/), so make sure it is installed and active in your system if you work with models from source.

{% hint style="info" %}
Working with models from source is often recommendable to quickly explore potential issues with the model. Typically, a dedicated Conda environment will be created by Ersilia automatically at fetch time.
{% endhint %}

## CI/CD workflows and testing

Ersilia relies heavily on GitHub Actions workflows for automation and testing. Visit the corresponding sections to learn more about:

* [Basic concepts of CI/CD at Ersilia](./#ci-cd-workflows-and-testing)
* [The model testing command](model-testing.md)
* [A fully-featured testing playground for the Ersilia CLI](cli-testing.md)
* [Styling and guidelines to ensure code quality in an automater manner](developer-guide-for-codebase-quality-and-consistency.md)

## Model packaging

An important part of the Ersilia infrastructure is the packaging of models with [Ersilia Pack](https://github.com/ersilia-os/ersilia-pack). Please see the [Ersilia Pack documentation ](./#model-packaging)for more information.

{% hint style="warning" %}
The legacy method for packaging models was strongly based on [BentoML](https://github.com/bentoml/). This is progressively being deprecated in favor our Ersilia Pack, which is built on top of [FastAPI](https://fastapi.tiangolo.com/).
{% endhint %}

