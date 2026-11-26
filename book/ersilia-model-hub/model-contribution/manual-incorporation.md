# Incorporate a model manually

This page walks through every step of the [Model incorporation workflow](model-incorporation-workflow.md) by hand, using a small, well-known model as an example: the synthetic accessibility (SA) score by [Ertl and Schuffenhauer (2009)](https://jcheminf.biomedcentral.com/articles/10.1186/1758-2946-1-8). Each step produces the same result as the skill named on the workflow page, and each skill's `SKILL.md` file in [ersilia-skills](https://github.com/ersilia-os/ersilia-skills) is a useful checklist.

{% hint style="info" %}
The SA score is already in the Hub, so use this page as a guide for your own model: don't open a model request for it. The commands use `eosXXXX` for the identifier of your model.
{% endhint %}

Before you start, install what you need, as described in [Before you begin](model-incorporation-workflow.md#before-you-begin).

## Choose a model

### Read the publication

Read the original publication to understand the training data, the validation and the limitations of the model. The SA score combines two parts: fragment contributions, derived from about one million molecules in PubChem, and a penalty for molecular complexity, such as large rings, stereocenters and unusual structural features. It goes from 1, easy to synthesize, to 10, very difficult. The authors validated it against the estimates of medicinal chemists for 40 molecules.

### Find the code and parameters

The code isn't linked from the publication, but the authors contributed it to RDKit, in [`Contrib/SA_Score`](https://github.com/rdkit/rdkit/tree/Release_2023_03_1/Contrib/SA_Score). The folder has three files you need:

* `sascorer.py`, the **model code**;
* `fpscores.pkl.gz`, the precomputed fragment scores, which are the **model parameters**;
* `data/zim.100.txt`, 100 molecules with the scores the authors expect, which you'll use to check the reproduction.

### Run the code outside Ersilia

Before you incorporate a model, make sure you can run the authors' code yourself. Create a conda environment, and note the version of every package you install, because Ersilia needs them:

```bash
conda create -n sa-score python=3.10
conda activate sa-score
pip install rdkit==2023.3.1 numpy==1.26.4
```

Download the code and parameters from a fixed version of the repository, not from its default branch, which can change at any time:

```bash
mkdir SA_Score SA_Score/data
cd SA_Score
BASE=https://raw.githubusercontent.com/rdkit/rdkit/Release_2023_03_1/Contrib/SA_Score
wget $BASE/sascorer.py $BASE/fpscores.pkl.gz
wget -P data $BASE/data/zim.100.txt
```

`sascorer.py` reads a file of SMILES and identifiers, separated by a space, with a header:

{% code title="molecules.smi" %}
```
smiles identifier
CC(=O)NC1=CC=C(O)C=C1 mol1
CN(C)CCC1=CNC2=C1C=C(CS(=O)(=O)N1CCCC1)C=C2 mol2
CCN(CC)CC1=C(O)C=CC(NC2=C3C=CC(Cl)=CC3=NC=C2)=C1 mol3
```
{% endcode %}

```bash
python sascorer.py molecules.smi
```

```
smiles	Name	sa_score
CC(=O)Nc1ccc(O)cc1	mol1	1.407299
CN(C)CCc1c[nH]c2ccc(CS(=O)(=O)N3CCCC3)cc12	mol2	2.319770
CCN(CC)Cc1cc(Nc2ccnc3cc(Cl)ccc23)ccc1O	mol3	2.249844
```

Keep this output: you'll compare the incorporated model with it later.

## Request the model

Open a [model request](https://github.com/ersilia-os/ersilia/issues/new?template=model_request.yml) on `ersilia-os/ersilia`. The form asks for the model's name, a short description, a slug, tags, the publication, the source code and the license. Fill it in as accurately as you can: these fields go into the model's metadata.

An Ersilia maintainer reviews the request and approves it by commenting `/approve`. This creates the model's repository, `ersilia-os/eosXXXX`, from the [model template](model-template.md). Don't create it yourself. Then fork the repository to your GitHub account and clone your fork:

```bash
git clone https://github.com/<your-username>/eosXXXX.git
```

## Fill in the metadata

Complete `metadata.yml`. Some fields come from the model request; complete the others from the paper and the source code. Fields with predefined values must match the accepted values exactly; see [The `metadata.yml` file](model-template.md#the-metadata.yml-file). Leave `Status` as it is: it changes to `Ready` automatically.

{% code title="metadata.yml" %}
```yaml
Identifier: eosXXXX
Slug: sa-score
Status: In progress
Title: Synthetic accessibility score
Description: |
  Estimates how easy a molecule is to synthesize, from 1 (easy) to 10 (very
  difficult). The score combines fragment contributions, derived from about
  one million molecules in PubChem, with a penalty for molecular complexity,
  such as large rings, stereocenters and unusual structural features. The
  authors validated it against the estimates of medicinal chemists for 40
  molecules (r2 = 0.89).
Deployment:
  - Local
Source: Local
Source Type: External
Task: Annotation
Subtask: Property calculation or prediction
Input:
  - Compound
Input Dimension: 1
Output:
  - Score
Output Dimension: 1
Output Consistency: Fixed
Interpretation: |
  Synthetic accessibility from 1 (easy to synthesize) to 10 (very difficult
  to synthesize).
Tag:
  - Synthetic accessibility
  - Chemical synthesis
Biomedical Area:
  - Any
Target Organism:
  - Any
Publication Type: Peer reviewed
Publication Year: 2009
Publication: https://jcheminf.biomedcentral.com/articles/10.1186/1758-2946-1-8
Source Code: https://github.com/rdkit/rdkit/tree/master/Contrib/SA_Score
License: BSD-3-Clause
```
{% endcode %}

Check every field against the paper, especially the source code, the license and the publication year. Check `Output Dimension` and `Interpretation` again once the code is in place.

## Add the model code

### Copy the code and parameters

From the folder that holds both `SA_Score` and your clone, put the code in `model/framework/code/` and the parameters in `model/checkpoints/`:

```bash
cp SA_Score/sascorer.py eosXXXX/model/framework/code/
cp SA_Score/fpscores.pkl.gz eosXXXX/model/checkpoints/
```

Weights always go in `model/checkpoints/`. If the authors' code expects them somewhere else, pass the new path from `main.py` rather than editing the authors' code. `fpscores.pkl.gz` is 3.7 MB, so it's committed to Git with the rest of the code. A file of 100 MiB or more goes to S3 instead; see [Checkpoints and storage](model-template.md#checkpoints-and-storage). The model must not download anything at run time.

### Write `main.py`

Ersilia runs the model through `model/framework/run.sh`, which calls `main.py` with the input and output files. Don't edit `run.sh`. The `main.py` of the template has two functions; keep that structure:

* `load_tool()` does the slow, one-time setup, such as reading the checkpoints, and returns the model. When the model is served, Ersilia calls it once and keeps the model in memory between batches.
* `run_tool(model, input_file, output_file)` runs the model on one batch: it reads the input file, predicts and writes the output file.

Wrap the authors' code without changing what it computes. Here, `readFragmentScores` accepts the path to the fragment scores, and `calculateScore` scores one molecule, so `sascorer.py` stays exactly as the authors wrote it:

{% code title="model/framework/code/main.py" %}
```python
# imports
import os
import sys

import numpy as np
from rdkit import Chem
from ersilia_pack_utils.core import read_smiles, write_out

# current file directory, and the checkpoints folder
root = os.path.dirname(os.path.abspath(__file__))
checkpoints = os.path.join(root, "..", "..", "checkpoints")

# the authors' code, unchanged, next to this file
sys.path.insert(0, root)
import sascorer  # noqa: E402


# load the model once: read the fragment scores from the checkpoints folder.
# When served, Ersilia calls this once and keeps the result in memory between batches.
def load_tool():
    sascorer.readFragmentScores(os.path.join(checkpoints, "fpscores"))
    return sascorer.calculateScore


# run the model on one batch: read the input file, predict, write the output file.
def run_tool(model, input_file, output_file):
    # read SMILES from the input file
    _, smiles_list = read_smiles(input_file)

    # run the model on each molecule; a molecule that can't be read gets NaN
    outputs = []
    for smi in smiles_list:
        mol = Chem.MolFromSmiles(smi)
        if mol is None or mol.GetNumAtoms() == 0:
            outputs.append([np.nan])
        else:
            outputs.append([model(mol)])
    outputs = np.array(outputs, dtype=np.float32)

    # check input and output have the same length
    assert len(smiles_list) == len(outputs)

    # write the output file
    write_out(outputs, ["sa_score"], output_file, np.float32)


if __name__ == "__main__":
    # parse arguments
    input_file = sys.argv[1]
    output_file = sys.argv[2]
    run_tool(load_tool(), input_file, output_file)
```
{% endcode %}

Keep these rules in mind for your own model:

* Return exactly one row per input molecule, in the same order. A molecule that can't be read gets an empty result, here `NaN`; it must not stop the batch or be dropped.
* Read and write files with `read_smiles` and `write_out` from `ersilia_pack_utils`, which handle every format Ersilia may pass to the model.
* Put everything slow and one-time in `load_tool()`. `run_tool()` must not keep anything between batches, so each batch gives the same result whatever ran before it.
* Keep the `if __name__ == "__main__":` block, so that `run.sh` keeps working.

### Pin the dependencies

List every package the model needs in `install.yml`, with exact versions that are still available to install:

{% code title="install.yml" %}
```yaml
python: "3.10"

commands:
  - ["pip", "rdkit", "2023.3.1"]
  - ["pip", "numpy", "1.26.4"]
  - ["pip", "ersilia-pack-utils", "0.1.5"]
```
{% endcode %}

### Describe the output columns

`model/framework/columns/run_columns.csv` describes every output column; see [Columns file](model-template.md#columns-file). The direction says which values mean more of what the column measures, not which values are better. A higher SA score means a molecule is harder to make, so its direction is `high`:

{% code title="run_columns.csv" %}
```
name,type,direction,description
sa_score,float,high,Synthetic accessibility score from 1 (easy to make) to 10 (very difficult to make)
```
{% endcode %}

### Write the example files

Put three molecules in `model/framework/examples/run_input.csv`, with a `smiles` header:

{% code title="run_input.csv" %}
```
smiles
CC(=O)NC1=CC=C(O)C=C1
CN(C)CCC1=CNC2=C1C=C(CS(=O)(=O)N1CCCC1)C=C2
CCN(CC)CC1=C(O)C=CC(NC2=C3C=CC(Cl)=CC3=NC=C2)=C1
```
{% endcode %}

Then, in a fresh environment with the packages from `install.yml`, run the model through `run.sh` from outside the repository, as Ersilia will, to produce `run_output.csv`:

```bash
FRAMEWORK=eosXXXX/model/framework
bash $FRAMEWORK/run.sh $FRAMEWORK $FRAMEWORK/examples/run_input.csv $FRAMEWORK/examples/run_output.csv
```

{% code title="run_output.csv" %}
```
sa_score
1.4072988
2.3197696
2.249844
```
{% endcode %}

Also run it on a file with a SMILES that can't be read, such as `not_a_smiles`, and check that it gets its own empty row.

## Test the model

Test the model the way Ersilia will run it in the Hub:

```bash
conda activate ersilia
ersilia test eosXXXX --shallow --from_dir eosXXXX
```

The test installs the model from your clone, runs it with every input and output format, checks that the metadata and the output files agree, and checks that repeated runs give the same results. The same test runs on your pull request, so fix every failed check before you continue. See [Troubleshooting models](troubleshooting-models.md).

{% hint style="warning" %}
`ersilia test`, and fetching or deleting a model, removes every conda environment whose name starts with the model identifier, and runs `docker system prune -f`. Don't name your own environments after the model.
{% endhint %}

## Reproduce the original results

First, check that the incorporated model gives the same outputs as the authors' code on the same molecules. The three molecules above match the authors' script, 1.407299, 2.319770 and 2.249844, up to the precision of the output. If they don't match, the incorporation has a bug: fix it before you continue.

Then compare the model with the results the authors report, where the data allow. For the SA score, the authors ship reference scores for 100 molecules in `data/zim.100.txt`. Run the model on them and compare:

```python
import csv
import subprocess

rows = [line.rstrip("\n").split("\t") for line in open("SA_Score/data/zim.100.txt")][1:]
with open("zim_input.csv", "w") as f:
    f.write("smiles\n")
    f.writelines(r[0] + "\n" for r in rows)

framework = "eosXXXX/model/framework"
subprocess.run(["bash", f"{framework}/run.sh", framework, "zim_input.csv", "zim_output.csv"], check=True)

predicted = [float(r["sa_score"]) for r in csv.DictReader(open("zim_output.csv"))]
expected = [float(r[2]) for r in rows]
print(sum(round(p, 3) == e for p, e in zip(predicted, expected)), "of", len(rows), "match")
```

All 100 scores match the authors' values to three decimals. The paper's own validation, against the estimates of medicinal chemists, can only be recomputed with their data. If a result can't be reproduced, or differs from the paper, say so in the pull request and explain why.

## Open the pull request

Remove any file you don't need, such as test inputs and outputs. Then commit your work to your fork and push it:

```bash
cd eosXXXX
git add .
git commit -m "Incorporate the SA score model"
git push
```

Open a pull request from your fork to the `main` branch of `ersilia-os/eosXXXX`. In its description, say how you tested the model and what you reproduced, including any result you couldn't reproduce.

Two checks run on every pull request:

* **Test model on PR** runs the same test as in [Test the model](#test-the-model).
* **Check version label** checks that the pull request has a version label. An Ersilia maintainer adds it.

If a check fails, push the fix to the same pull request. An Ersilia maintainer reviews the pull request and merges it once every check passes.

## Release the model

After the merge, Ersilia's workflows test the model again, build its Docker images and SIF image, update the metadata and the catalog, and release it as `v1.0.0`. You don't need to do anything, but keep an eye on your model request issue in case users report problems with the model.

Once the model is released, you can delete your fork.
