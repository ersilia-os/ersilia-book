---
description: Documentation to run Ersilia models online
---

# Online inference old

To ensure users from all backgrounds are able to benefit from our tools, we provide a ready-to-use, no-code solution to obtain predictions for your molecules. Before continuing, be aware of the limitations of the online inference:

* The maximum list of molecules to predict is 1,000
* Not all models are available in the platform, at your request we can include some if needed (hello@ersilia.io).
* Your molecules will be stored in Ersilia's AWS backend. We will not publish or use them but take this into consideration if you are concerned about privacy.

{% hint style="info" %}
If your user needs are not aligned with these points, please look at running models [locally](local-inference-old.md), especially for large inference exercises. We can provide limited support for the setup if needed.
{% endhint %}

## 1. Select your model of interest

We offer a broad range of models, from bioactivity prediction against several pathogens (malaria, tuberculosis, schistosomiasis, ESKAPE pathogens...) to ADME endpoints and toxicity predictions. Use our [catalog](https://catalog.ersilia.io/) to browse available models.

{% hint style="success" %}
Ensure you understand the output of each model before using it. _For example, a classifier will output the probability of being active in a particular assay, for example, probability that a molecule kills the malaria parasite in an in vitro assay, and a regressor might output the predicted IC50 value at which the molecule inhibits the growth of the parasite._
{% endhint %}

## 2. Prepare your input data

The molecules must be displayed in SMILES notation. You can use [PubChem](https://pubchem.ncbi.nlm.nih.gov/) to find the SMILES notation of a given compound:  enter the compound name in the search bar (for example, aspirin), select the best result and scroll to the SMILES section within "Name and Identifiers" (in this case; `CC(=O)OC1=CC=CC=C1C(=O)O`). If your starting input data is an `.sdf` file, use your preferred visualiser, like ChemDraw, to open the molecule and obtain its SMILES representation. To deal with multiple molecular file formats, including SDF, you can use [OpenBabel](https://openbabel.org/) to convert them into SMILES notation. Alternatively, you can also use free software like [Marvin.js](https://marvinjs-demo.chemaxon.com/latest/demo.html) to draw a molecule and then simply click on save 💾 it as a SMILES.

Collect your list of SMILES in a simple `.csv`  file **containing a single column with a header**, for example "smiles" or "input".

## 3. Run predictions and download results

Go to our [online inference app](https://hub.ersilia.io/), register as a user if it is your first time and select your model of choice from the drop down list.

Add the .csv you have generated with "Attach file" and start the inference. Models may take from a few seconds to up to 30 minutes for the slowest ones. Once the run is complete, download your results.

## 4. Check your predictions

By default, Ersilia will provide a downloadable `.csv` file summarizing the results, containing the following columns:

* key: 32-character unique identifier of the molecule created by Ersilia
* input: the input SMILES (please note that these might have been standardized if they were not provided in the standard format).
* Model output: one or several columns containing the predictions of the selected model. Explanation about each column's meaning can be found on the model's GitHub repository under `model/framework/columns`. For example, for model `eos3b5e` you can go to [the model repository](https://github.com/ersilia-os/eos3b5e) and navigate to the [run\_columns.csv](https://github.com/ersilia-os/eos3b5e/blob/main/model/framework/columns/run_columns.csv) file.
