# ZairaChem

[ZairaChem](https://github.com/ersilia-os/zaira-chem) is [Ersilia](https://ersilia.io)'s **AutoML** tool for supervised learning of small molecule activity and property data. In the field of computer-aided drug discovery, this task is known as Quantitative Structure Activity/Property Relationship Modeling (**QSAR/QSPR**).

ZairaChem offers a relatively complex **ensemble modeling** pipeline, showing robust performance over a wide set of tasks. If, instead, you want to build quick baseline models, we recommend to check [LazyQSAR](https://github.com/ersilia-os/lazy-qsar), the light-weight modeling tool of Ersilia.

Currently, ZairaChem is focused on **binary classification** tasks. We presented ZairaChem in a joint publication with the [H3D Centre](http://www.h3d.uct.ac.za/) (South Africa). Please cite: [Turon, Hlozek et al, Nature Communications, 2023](https://www.nature.com/articles/s41467-023-41512-2).

In brief, in ZairaChem molecules are represented numerically using a combination of distinct **descriptors**, including physicochemical parameters ([Mordred](https://jcheminf.biomedcentral.com/articles/10.1186/s13321-018-0258-y)), 2D structural fingerprints ([ECFP](https://www.rdkit.org/docs/GettingStartedInPython.html)), inferred bioactivity profiles ([Chemical Checker](https://www.nature.com/articles/s41587-020-0502-7)), graph-based embeddings ([GROVER](https://arxiv.org/pdf/2007.02835.pdf)), and chemical language models ([ChemGPT](https://huggingface.co/ncfrey/ChemGPT-4.7M)). Any other descriptor from the [Ersilia Model Hub](https://ersilia.io/model-hub) can be selected. The rationale is that combining multiple descriptors will enhance applicability over a broad range of tasks, ranging from aqueous solubility predictions to phenotypic outcomes. Subsequently, an array of AI/ML algorithms is applied using modern AutoML techniques aimed at yielding accurate models without the need for human intervention (i.e. algorithm choice, hyperparameter tuning, etc.). The **AutoML frameworks** [FLAML](https://microsoft.github.io/FLAML/), [AutoGluon](https://auto.gluon.ai/stable/index.html), [Keras Tuner](https://keras.io/keras_tuner/), [TabPFN](https://github.com/automl/TabPFN) and [MolMapNet](https://github.com/shenwanxiang/bidd-molmap) are incorporated, covering mostly tree-based methods (Random Forest, XGBoost, etc.) and neural network architectures.

## Installation

ZairaChem can be installed as follows:

```bash
git clone https://github.com/ersilia-os/zairachem-docker.git
cd zairachem-docker
conda create -n zairachem python=3.12
conda activate zairachem
pip install -e .
```

```bash
zairachem --help
```

## Input and Output

To run ZairaChem, a file with two columns is required:

* SMILES: a list of smiles for model training
* Label: a binary label (1 or 0) or a continuous experimental value. If the latter, the binarization cut-off and direction must also be specified by the user (see below)

The output of ZairaChem is a model folder that contains all the necessary files to run the model as well as automated reports on performance. Please note that ZairaChem will not automatically do train/test splits and those need to be prepared by the user and run in subsequent trials.

## Quick start

To get started, let's use a classification task from [Therapeutic Data Commons](https://tdcommons.ai/).

```
zairachem example --classification --file_name input.csv
```

This file can be split into train and test sets.

```
zairachem split -i input.csv
```

The command above will generate two files in the current folder, named `train.csv` and `test.csv`. By default, the train:test ratio is 80:20.

## Fit

You can train a model as follows:

```
zairachem fit -i train.csv -m model
```

This command will run the full ZairaChem pipeline and produce a `model` folder with processed data, model checkpoints, and reports.

### Predict

You can then run predictions on the test set:

```
zairachem predict -i test.csv -m model -o test
```

ZairaChem will run predictions using the checkpoints stored in `model` and store results in the `test` directory. Several performance plots will be generated alongside prediction outputs.

## Pipeline steps

Internally, the ZairaChem pipeline consists of the following steps:

1. `session`: a session is initialized pointing to the necessary system paths.
2. `setup`: data is processed and stored in a cleaned form.
3. `describe`: molecular descriptors are calculated.
4. `estimate`: models are trained or predictions are done on trained models.
5. `pool`: results from multiple models from the ensemble are aggregated.
6. `report`: output data is assembled in a spreadsheet, and plots are created for easy inspection of results.
7. `distill` (mainly based on the [Olinda](https://github.com/ersilia-os/olinda) package; integration in progress :construction\_worker:): lightweight versions of the models are created for quick prediction.
8. `finish`: the session is closed and residual files are deleted.

### Session

You can start a ZairaChem training session as follows:

```
zairachem session --fit -i train.csv -m model
```

Likewise, you can start a prediction session:

```
zairachem session --predict -i test.csv -m model -o test
```

The `session` command will simply create the necessary folders and a session log.

### Setup

In this step, data preparation is done, including:

* Identification of relevant columns (compound identifier, SMILES, and value) in the input file.
* Chemical structure standardization.
* Deduplication.
* Data balancing and augmentation using a reference set of molecules (e.g. ChEMBL).
* Binarization when a cutoff is specified.
* Transformation (Gaussianization) of continuous data.
* Folds and clusters assignments.

Give an initialized session (fit or predict), data preparation will be done accordingly. To perform this step, simply run:

```
zairachem setup
```

Most data generated in the `setup` step will be stored in `model/data` (fit) or `test/data` (predict). The most important file in this folder is `data.csv`, containg the result of the data preparation step. Other files are generated, like `mapping.csv`, which match `data.csv` to the row indices of the input file.

### Describe

In the `describe` step, small molecule descriptors are calculated. ZairaChem provides a set of default descriptors, including the [Chemical Checker signaturizer](https://bioactivitysignatures.org), Grover embeddings and Morgan fingerprints and Mordred descriptors.

Other descriptors can be easily incorporated thanks to the [Ersilia Model Hub](https://ersilia.io/model-hub). They can be specified in a `parameters.json` file.

Several operations are performed for each of the descriptors, including:

* Calculation of descriptors for each molecule using the Ersilia Model Hub.
* Removal of constant-value columns and columns with a high degree of missing values.
* Imputation of the rest missing values.
* Robust scaling of continuous descriptors.

In addition, a reference descriptor is calculated (Grover). To this reference descriptors, the following dimensionality reduction techniques are applied:

* UMAP
* PCA

Optionally, supervised versions of these algorithms are applied:

* Supervised UMAP
* LolP

All of the above can be performed by running the following command:

```
zairachem describe
```

Please note that calculating some descriptors (for example, GROVER) may be a slow procedure. However, the Ersilia backend is linked to an in-house caching library called [Isaura](https://github.com/ersilia-os/isaura) that is able to access pre-calculated data. At the moment, Isaura works on local caching. However, we are currently setting up a cloud-based database in order to facilite access to pre-calculations stored online.

### Estimate

This step is aimed at training AutoML models based on the descriptors calculated above.

The following supervised models are applied:

* Baseline LazyQSAR models (based on Morgan fingerprints and classic descriptors).
* FLAML models on each of the pre-calculated descriptors.
* AutoGluon model based on the manifolds of the reference embedding.
* Keras Tuner fully-connected network based on the reference embedding.
* MolMap convolutional neural network.

All of these steps can be performed with the following command.

```
zairachem estimate
```

### Pool

In the pooling step, results from the estimators above are aggregated. A weighted average is applied, based on the expected performance of each of the individual estimators.

Pooling can be performed with the following command:

```
zairachem pool
```

### Report

ZairaChem provides automated performance reports as well as a output table.

* Output table
* Performance table
* Plots

```
zairachem report
```

### Distill

ZairaChem models are computationally demanding. At the end of the procedure, our goal is to provide a distilled model. This distilled model is stored in an interoperable format (ONNX) and can be deployed as an AWS lambda. The Ersilia package for creating distilled models is called [Olinda](https://github.com/ersilia-os/olinda).

### Finish

The finish command simply offers options for cleaning

```
ersilia finish
```

## How to run a step of interest

It is possible to run a specific step from a previous session. In this case, simply initialize the session pointing to the relevant folders:

```
zairachem session --path model
```

ZairaChem will automatically identify the session as training (fit) task or as a prediction task.

Once the session has been set, you can run the command of choice. For example:

```
zairachem describe
```

### The session file

In the session file, multiple steps are specified. Each step in ZairaChem has an associated name. You can restart the pipeline at any given step.
