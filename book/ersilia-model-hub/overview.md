# Overview

The Ersilia Model Hub is our main platform for serving AI/ML models related to infectious and neglected disease drug discovery. The Hub contains both models built by Ersilia as well as models collected from the literature, all packaged in a standard way for easy access. All our models (code and checkpoints) are open source, and training data is also provided  when available.

The Ersilia Model Hub is built upon an interlinked ecosystem of tools described below. In this section of the Ersilia Book you will find how these tools work together to keep the Hub running and accessible as well as the necessary documentation to start working with them.

<figure><picture><source srcset="../.gitbook/assets/ersilia-ecosystem-dark.svg" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/ersilia-ecosystem-light (1).svg" alt=""></picture><figcaption><p>Ersilia's ecosystem</p></figcaption></figure>

## TL;DR

Models enter the ecosystem in two ways. We either train our own models from public and private data, or we incorporate them directly from the scientific literature. Models are built with automated AI/ML workflows which are described in detail in the [Chemistry Tools](https://app.gitbook.com/s/-Mj44wxA7bU1hQH19m8I/chemistry-tools) section. Each trained model is built and packaged in a standard format and published to GitHub, Docker Hub, and as an SIF image, so that it runs across different platforms, including HPC clusters. In addition, we use standardised inputs to generate precalculated results for all the models in Ersilia, and make those available to the community as well. A series of workflows enable semi-automated maintenance of the Hub and ensure all models remain usable. Finally, two web apps make the models available to first-time users: the catalog to browse our offerings and the online interface to try out models without writing code.

Every Ersilia repository is open source and available in Ersilia's [GitHub organization](https://github.com/ersilia-os).

## Getting started

Use the following sections to run, contribute, or develop models in the Hub:

* [Online inference](/broken/pages/wlr6FbCr1pfsiXl1EZr2): select the model of your choice and run predictions without any need for coding skills (for end users without computational background).
* [Local inference](/broken/pages/gGZC225cfiDyKWK3K3j0): install Ersilia locally, download the models and run predictions in your system (for end users with basic knowledge of bash and python).
* [Model contribution](model-contribution/): incorporate models from the scientific literature in the Hub (for computational biologists and research software engineers wishing to leverage Ersilia as a dissemination platform for their models).
* [Developers](developer-docs/): support the development of the Hub (for software engineers and DevOps engineers).
* [Model Hub Maintenance](model-hub-maintenance.md): overview of the infrastructure and processes used to maintain the Ersilia Model Hub. Includes automated workflows, public reports, and maintenance guidelines for monitoring model health and resolving issues (this repository is aimed at developers).

