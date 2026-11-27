# Model template

Every model in the Ersilia Model Hub lives in its own GitHub repository, created from the [Ersilia model template](https://github.com/ersilia-os/eos-template). This page describes what each part of that repository does. Use it as a reference whether you incorporate a model manually or with the skills: either way, the result must follow these rules.

## Repository structure

```
eosXXXX/
├── metadata.yml          # all information about the model
├── install.yml           # dependencies
├── README.md             # generated automatically, don't edit
├── LICENSE
├── access.json           # only if folders are stored with eosvc
└── model/
    ├── checkpoints/      # model weights and other model data
    └── framework/
        ├── run.sh        # entry point Ersilia calls
        ├── code/
        │   └── main.py
        ├── columns/
        │   └── run_columns.csv
        ├── examples/
        │   ├── run_input.csv
        │   └── run_output.csv
        └── fit/          # training code and data, if any
```

## How a model repository is created

When an Ersilia maintainer approves a model request, the model's repository is automatically created from the template. Fork it, make your changes in your fork, and open a pull request when the model is ready.

{% hint style="warning" %}
After your pull request is merged, delete your fork, so outdated copies of the model don't stay around.
{% endhint %}

## The `eos` identifier

Each model has an Ersilia Open Source (EOS) identifier, which is also the name of its repository: `https://github.com/ersilia-os/[EOS_IDENTIFIER]`.

The identifier matches the regular expression `eos[1-9][a-z0-9]{3}`: the prefix `eos`, one digit from 1 to 9 (0 is reserved for test models), and three lowercase letters or digits. Identifiers are assigned automatically when the repository is created. Don't change them.

## The `metadata.yml` file

`metadata.yml` holds all the information about the model, and it's the only place to edit it. When a pull request is merged, the Ersilia catalog, the Airtable backend, and the repository's `README.md` are updated from this file.

The file is filled in over time: some fields come from the model request, some are completed by the contributor, and some are added by Ersilia's workflows. Fields with predefined values are checked by automated tests, so spelling and capitalization must match exactly. The accepted values for each field are listed in the [metadata files](https://github.com/ersilia-os/ersilia/tree/master/ersilia/hub/content/metadata) of the Ersilia repository.

{% hint style="info" %}
If the accepted values don't cover your model, for example a disease that isn't listed as a tag, open a pull request to add it to the [metadata files](https://github.com/ersilia-os/ersilia/tree/master/ersilia/hub/content/metadata). Ersilia maintainers review these additions.
{% endhint %}

### Metadata at model request

These fields are filled in from the model request issue.

<table data-search="false"><thead><tr><th>Field</th><th>Description</th></tr></thead><tbody><tr><td>Identifier</td><td>The <code>eos</code> identifier. Filled in automatically. Don't edit.</td></tr><tr><td>Slug</td><td>A human-readable alternative to the identifier, one to four lowercase words joined by hyphens, up to 60 characters.</td></tr><tr><td>Title</td><td>A self-descriptive title, 10 to 300 characters.</td></tr><tr><td>Description</td><td>The model type, its results, and the training data, in 200 to 600 characters.</td></tr><tr><td>Tag</td><td>One to five labels that help people find the model, such as <code>Malaria</code> or <code>P.falciparum</code>. Use only the values in <a href="https://github.com/ersilia-os/ersilia/blob/master/ersilia/hub/content/metadata/tag.txt">tag.txt</a>.</td></tr><tr><td>Publication</td><td>A link to the original publication. For a peer-reviewed paper it must be a DOI link (<code>https://doi.org/...</code>), which the model test checks. Preprints should use their DOI too; check whether the preprint has since been published and, if so, cite the published version. If the model isn't published in a journal or as a preprint, contact Ersilia maintainers.</td></tr><tr><td>Source Code</td><td>A link to the original code. For models trained by Ersilia without other source code, link the ML package used for training.</td></tr><tr><td>License</td><td>The license of the original code, as an SPDX identifier, or <code>None</code> if the code has no license. Respect the original license's requirements, for example by keeping its license file with the code.</td></tr></tbody></table>

### Metadata at model contribution time

Complete these fields before you open your pull request.

<table data-search="false"><thead><tr><th>Field</th><th>Description</th></tr></thead><tbody><tr><td>Status</td><td><code>In Progress</code> during incorporation. It changes to <code>Ready</code> automatically. Don't edit.</td></tr><tr><td>Deployment</td><td>Where the model can run: <code>Local</code>, <code>Online</code>, or both.</td></tr><tr><td>Source</td><td><code>Local</code> if the model runs on Ersilia's infrastructure or the user's machine, or <code>Online</code> if it sends inputs to a server outside Ersilia. Ersilia can't guarantee the privacy of inputs sent to <code>Online</code> models.</td></tr><tr><td>Source Type</td><td><code>External</code> for third-party models, <code>Internal</code> for models developed by Ersilia, or <code>Replicated</code> for models retrained following the original authors' instructions.</td></tr><tr><td>Task</td><td><code>Annotation</code>, <code>Representation</code>, or <code>Sampling</code>.</td></tr><tr><td>Subtask</td><td>For Annotation: <code>Property calculation or prediction</code> or <code>Activity prediction</code>. For Representation: <code>Featurization</code> or <code>Projection</code>. For Sampling: <code>Similarity search</code> or <code>Generation</code>.</td></tr><tr><td>Input</td><td>The input type. Currently always <code>Compound</code>.</td></tr><tr><td>Input Dimension</td><td>The number of inputs per prediction. Currently always <code>1</code>. You can still pass a list of molecules: each prediction uses one.</td></tr><tr><td>Output</td><td><code>Score</code> (for example, a probability), <code>Value</code> (a measured or calculated property), <code>Compound</code> (a new molecule), or <code>Text</code>.</td></tr><tr><td>Output Dimension</td><td>The number of output values per input.</td></tr><tr><td>Output Consistency</td><td><code>Fixed</code> if the model always gives the same prediction for the same molecule, or <code>Variable</code> if not, as with most generative models.</td></tr><tr><td>Interpretation</td><td>How to read the results. For a classifier, include the experimental settings and the cutoff used, for example the IC50 threshold and the parasite strain.</td></tr><tr><td>Biomedical Area</td><td>One or more research areas, such as <code>ADMET</code> and <code>Malaria</code>. Use <code>Any</code> for models that apply to all areas, such as featurizers.</td></tr><tr><td>Target Organism</td><td>The pathogen or organism the model relates to, such as <code>Plasmodium falciparum</code> or <code>Homo sapiens</code>, or <code>Any</code>.</td></tr><tr><td>Publication Type</td><td><code>Peer reviewed</code>, <code>Preprint</code>, or, only in exceptional cases, <code>Other</code>.</td></tr><tr><td>Publication Year</td><td>The year the original model was published.</td></tr></tbody></table>

### Metadata updated by Ersilia

Ersilia's workflows add these fields after the pull request passes its tests. Don't edit or remove them. To learn how they're filled in, see [CI/CD workflows and testing](../developer-docs/testing-and-quality/ci-cd-workflows.md).

<table data-search="false"><thead><tr><th>Field</th><th>Description</th></tr></thead><tbody><tr><td>Contributor</td><td>The contributor's GitHub username.</td></tr><tr><td>Incorporation Date</td><td>When the model was first added to the Hub, as <code>YYYY-MM-DD</code>.</td></tr><tr><td>S3</td><td>The URL of the zipped model in Ersilia's AWS storage.</td></tr><tr><td>DockerHub</td><td>The URL of the model's Docker image.</td></tr><tr><td>Docker Architecture</td><td>The architectures the image is available for, usually <code>AMD64</code> and <code>ARM64</code>.</td></tr><tr><td>Apptainer</td><td>The URL of the model's SIF image</td></tr><tr><td>Model Size, Environment Size, Image Size</td><td>The size of the repository, the conda environment, and the Docker image, in MB.</td></tr><tr><td>Computational Performance 1 to 5</td><td>The time to run 1, 10, 100, 1,000, and 10,000 inputs, in seconds.</td></tr><tr><td>Last Packaging Date</td><td>When the model was last packaged and uploaded to Docker Hub, as <code>YYYY-MM-DD</code>.</td></tr><tr><td>Release</td><td>The model's version, following semantic versioning. The first release is <code>v1.0.0</code>.</td></tr></tbody></table>

## The `README.md` file

The `README.md` file is generated from `metadata.yml` when a pull request is merged. Don't edit it by hand.

## The `LICENSE` file

Code written for Ersilia is licensed under [GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html), so the repository's main `LICENSE` file is GPLv3. Code from third parties keeps its own license notice, in the folder where that code lives.

## The `install.yml` file

`install.yml` lists everything the model needs to run. Ersilia uses it to build the model's conda environment or Docker image. It has two required keys:

* `python`: the Python version, as a string. Python 3.8 or later.
* `commands`: a list of installation commands.

{% code title="install.yml" %}
```yaml
python: "3.10"
commands:
  - ["pip", "rdkit-pypi", "2022.3.1b1"]
  - ["conda", "pandas", "1.3.5", "default"]
```
{% endcode %}

Each entry in `commands` takes one of these forms:

* **A shell command, as a string**, passed as is: `"apt-get update"`.
* **A pip package**, as `["pip", "package", "version"]`, optionally followed by pip flags: `["pip", "mypkg", "0.1.2", "--no-deps"]`. For packages installed from Git, use a two-item list: `["pip", "git+https://github.com/user/repo.git"]`.
* **A conda package**, either in full form, `["conda", "install", "-y", "-c", "channel", "package=version"]` (recommended), or in short form, `["conda", "package", "version", "channel"]`. Ersilia adds `-y` if it's missing, and uses the default channel if none is given.

When the model runs from source, Ersilia always creates a conda environment for it. When the model is packaged in Docker, conda is used only if `install.yml` includes conda packages.

Keep the dependencies to the minimum the model needs to run, and always pin their versions, so the model keeps working over time.

## The `model` folder

The `model` folder contains exactly two subfolders:

* `framework`: the code that runs the model, with the example and columns files.
* `checkpoints`: the model's weights and other model data.

{% hint style="danger" %}
Don't put anything else directly in `model`. When Ersilia fetches a model, it keeps only these two subfolders and ignores everything else.
{% endhint %}

### The `run.sh` file

`run.sh`, in `framework`, is the entry point Ersilia calls to run the model. It takes three arguments, and only three unless strictly necessary:

```bash
bash run.sh [FRAMEWORK_DIR] [DATA_FILE] [OUTPUT_FILE]
```

In the template, it runs `main.py`:

{% code title="run.sh" %}
```bash
python $1/code/main.py $2 $3
```
{% endcode %}

### The `main.py` file

`main.py` is where most of the incorporation work happens. It's organized in two functions:

* `load_tool()` does the slow, one-time setup, such as imports, reading the checkpoints, and building the model, and returns the model.
* `run_tool(model, input_file, output_file)` runs the model on one batch: it reads the input file, makes the predictions, and writes the output file.

When a model is served, Ersilia calls `load_tool()` once and keeps the model in memory between batches, which is often several times faster than loading it for every batch. See [Warm serving](../developer-docs/running-models/warm-serving.md). Don't keep anything between batches in `run_tool()`: each batch must give the same result, whatever ran before it.

In the template, the model is a placeholder that calculates molecular weight:

{% code title="code/main.py" %}
```python
import os
import sys
import numpy as np
from rdkit import Chem
from rdkit.Chem.Descriptors import MolWt
from ersilia_pack_utils.core import read_smiles, write_out

root = os.path.dirname(os.path.abspath(__file__))
checkpoints = os.path.join(root, "..", "..", "checkpoints")


# Load the model once. When served, Ersilia keeps the result in memory between batches.
def load_tool():
    return MolWt


# Run the model on one batch.
def run_tool(model, input_file, output_file):
    _, smiles_list = read_smiles(input_file)
    outputs = np.array([[model(Chem.MolFromSmiles(smi))] for smi in smiles_list])
    assert len(smiles_list) == len(outputs)
    header = [f"feat_{str(i).zfill(3)}" for i in range(outputs.shape[1])]
    write_out(outputs, header, output_file, np.float32)


if __name__ == "__main__":
    run_tool(load_tool(), sys.argv[1], sys.argv[2])
```
{% endcode %}

`read_smiles` and `write_out`, from `ersilia_pack_utils`, read a CSV file of SMILES with a header and write the output with one column per value. In a real model, `load_tool()` usually loads the model from a third-party library or from other scripts in `code`. Many third-party models already have a command-line interface. In that case, you can write scripts that adapt the input and output, and run the model as is.

### Example files

The `framework/examples` folder holds an example input and its expected output, which Ersilia's tests use to check the model:

* `run_input.csv`: a single `smiles` column with three molecules.
* `run_output.csv`: the model's output for those three molecules, with only the output columns, not `key` or `input`.

Ersilia routinely checks the model's output against `run_output.csv`, except for models with `Variable` output consistency, so make sure it's correct.

### Columns file

The `framework/columns/run_columns.csv` file describes every output column. It has four columns:

* `name`: the column name, exactly as the model outputs it. Use lowercase letters, digits, and underscores only. For models that generate molecules, use `smi_00`, `smi_01`, and so on. For featurizers without specific names, use `feat_00`, `feat_01`, and so on, padded with zeros to the same length.
* `type`: `string`, `float`, or `integer`. A model can mix `float` and `integer` columns, but not text and numbers.
* `direction`: `high` or `low`, depending on which values mean more of what the column measures. For example, a probability of activity has direction `high`. Leave it empty if direction doesn't apply, such as for generated molecules.
* `description`: a one-line description of the column, without commas.

## Checkpoints and storage

These rules apply to every model repository:

1. Weights that the model loads at inference go in `model/checkpoints/`, not in `model/framework/code/`. Update the paths in the code to match.
2. If every file in `model/checkpoints/` is smaller than 100 MiB (104,857,600 bytes, GitHub's file size limit), commit the folder to Git. If any file is 100 MiB or larger, store the whole folder in S3 with [eosvc](../developer-docs/data-storage/ersilia-version-control.md) instead.
3. Store `model/framework/fit/` with eosvc as soon as it holds training code or data, whatever its size.
4. Don't use Git LFS, and don't add a `.gitattributes` file with LFS rules.
5. The model must not download weights at run time, for example from Hugging Face, Google Drive, or Zenodo. Download them once, put them in `model/checkpoints/`, and load them from there.

{% hint style="info" %}
Uploading to eosvc requires AWS credentials. If you're an external contributor, an Ersilia maintainer will help you with this step.
{% endhint %}

To store a folder with eosvc, list it in an `access.json` file at the root of the repository, add it to `.gitignore`, and upload it. This example is for a model with both folders on eosvc. List only the folders you store with eosvc:

{% code title="access.json" %}
```json
{
    "checkpoints": "public",
    "fit": "public"
}
```
{% endcode %}

{% code title=".gitignore" %}
```
#eosvc managed
model/checkpoints/*
model/framework/fit/*
```
{% endcode %}

```bash
eosvc upload --path checkpoints
eosvc upload --path fit
```

{% hint style="danger" %}
Upload only the folders that go to eosvc, one at a time. Never run `eosvc upload --path .`. Leaving a folder out of `access.json` doesn't stop eosvc: it treats the folder as public, and the CI's `eosvc download --path .` replaces the Git copy with any copy in S3. Never keep the same files in both Git and S3. If you committed a folder before moving it to eosvc, untrack it with `git rm -r --cached model/checkpoints` (or `model/framework/fit`).
{% endhint %}

If neither folder goes to eosvc, the repository doesn't need an `access.json` file. A model without a training step keeps the template's `model/framework/fit/` skeleton (a `README.md` file and empty `data`, `results`, and `src` folders) in Git, and doesn't list `fit` in `access.json` or `.gitignore`.
