# For developers

This section is for developers who want to understand or contribute to the infrastructure behind the Ersilia Model Hub: the Ersilia CLI and Python package, and the tools that package, run, store, test, and maintain models. If you want to contribute a model, see [Model contribution](../model-contribution/) instead.

### Set up a development environment

Fork the [Ersilia repository](https://github.com/ersilia-os/ersilia), then install your fork in editable mode, with its test dependencies:

bash

```bash
conda create -n ersilia python=3.12
conda activate ersilia
git clone https://github.com/<your_username>/ersilia.git
cd ersilia
pip install -e ".[test]"
pre-commit install
```

Install [Docker](https://docs.docker.com/get-docker/) too: Ersilia fetches models as Docker images by default, and needs Docker for results caching.

To check that everything works, run `ersilia --help`. Before you open a pull request, read Best code practices and Ersilia's [contribution guide](https://github.com/ersilia-os/ersilia/blob/master/CONTRIBUTING.md).

### What's in this section

The pages follow the parts of the Ersilia ecosystem:

* **Interfaces**: the command line interface and the Python API, in full.
* **Packaging and distribution**: how Ersilia Pack turns a model into a server, and how models are distributed as Docker and Apptainer images.
* **Running models**: how warm serving and results caching make runs faster.
* **Data storage**: the precalculation store, and how large files are stored with eosvc.
* **Testing and quality**: model and CLI testing, the CI/CD workflows, and code standards.
* **Maintenance**: how the Hub is monitored and how failing models are fixed.

For a reference of Ersilia's Python code, generated from its docstrings, see the [API documentation](https://ersilia-os.github.io/ersilia/).
