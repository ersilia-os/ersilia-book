# Online Inference

The [Ersilia Hub](https://hub.ersilia.io/) lets you run models in your browser, with no installation or coding. Upload your molecules, choose a model, and download the predictions.

## Before you begin

* You can submit up to 1,000 molecules per run.
* Not all models are available in the Hub. To request a model, write to [hello@ersilia.io](mailto:hello@ersilia.io).
* You need to register as a user the first time you use the Hub.

{% hint style="warning" %}
Your molecules are stored on Ersilia's servers on AWS. We don't publish or use them, but if you're working with confidential or IP-protected molecules, run models on your own machine with [local inference](local-inference.md) instead.
{% endhint %}

{% hint style="info" %}
For large jobs, or models that aren't in the Hub, use [local inference](local-inference.md). We can offer limited support with the setup if you need it.
{% endhint %}

## Run a model

{% stepper %}
{% step %}
#### Find a model

Ersilia offers a broad range of models, from bioactivity against pathogens such as malaria, tuberculosis, schistosomiasis, and the ESKAPE pathogens, to ADME and toxicity predictions. Browse them in the [Ersilia catalog](https://catalog.ersilia.io/).

{% hint style="success" %}
Make sure you understand what a model predicts before you use it. For example, a classifier outputs the probability of being active in a particular assay, such as the probability that a molecule kills the malaria parasite in vitro. A regressor might output a predicted IC50, the concentration at which the molecule inhibits the parasite's growth.
{% endhint %}
{% endstep %}

{% step %}
#### Prepare your molecules

Models take chemical structures as input, written as SMILES strings. To get them:

* **From a compound name:** search for the compound on [PubChem](https://pubchem.ncbi.nlm.nih.gov/), for example aspirin, open the best result, and find the SMILES string under **Names and Identifiers**. For aspirin, it's `CC(=O)OC1=CC=CC=C1C(=O)O`.
* **From an SDF or other file format:** open the file in a molecule visualizer such as ChemDraw, or convert it with [OpenBabel](https://openbabel.org/).
* **From a drawing:** draw the molecule in [Marvin JS](https://marvinjs-demo.chemaxon.com/latest/demo.html) and save it as SMILES.

Save your molecules in a CSV file with a single column and a header, such as `smiles` or `input`:

{% code title="input.csv" %}
```
smiles
C1=C(SC(=N1)SC2=NN=C(S2)N)[N+](=O)[O-]
CC(C)CC1=CC=C(C=C1)C(C)C(=O)O
```
{% endcode %}
{% endstep %}

{% step %}
#### Run predictions

Go to [hub.ersilia.io](https://hub.ersilia.io/) and sign in, then choose your model from the dropdown list.

Click **Attach file**, select your CSV file, and start the run.

Most models finish in a few seconds. The slowest can take up to 30 minutes.
{% endstep %}

{% step %}
#### Download your results

When the run is complete, download your predictions as a CSV file.
{% endstep %}
{% endstepper %}

## Understand model outputs

Ersilia hosts many kinds of models, so the output depends on the model you run. Every output file contains at least two columns:

* `key`: a 32-character unique identifier that Ersilia creates for each molecule.
* `input`: the input SMILES string. Ersilia might standardize SMILES strings that weren't in standard format.

The remaining columns contain the model's predictions. To learn what each column means, what type of data it holds, and how to interpret its values, check the model's documentation in either of these places:

* The README file in the model's GitHub repository, for example [ersilia-os/eos3b5e](https://github.com/ersilia-os/eos3b5e).
* The `run_columns.csv` file in the model's repository, at `model/framework/columns/run_columns.csv`, which defines every output column.

## Get help

If something doesn't work, write to [hello@ersilia.io](mailto:hello@ersilia.io) with the model you used and a description of the problem.
