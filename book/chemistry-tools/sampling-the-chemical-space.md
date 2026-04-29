---
description: >-
  We present ChemSampler, a simple tool to sample molecules in a given region of
  the chemical space
---

# Sampling the chemical space

{% hint style="danger" %}
This page is :construction\_worker: **work in progress** :construction\_worker:!
{% endhint %}

ChemSampler acts as a framework that fetches generative AI models from the Ersilia Model Hub and runs iterative rounds of generation to provide the user with a pool of new or sampled existing molecules for downstream filtering using bioactivity predictors and/or docking approaches.

## Basic structure

ChemSampler accepts one single molecule (<mark style="color:green;">seed\_smiles</mark>) as input. The seed smiles is passed to each one of the Ersilia Model Hub Generative models (<mark style="color:orange;">samplers</mark>) specified in the `params.json` file (see below). All results of this first round are collated in a single output .csv file, and a random sample of 100 molecules from the generated list is provided to the user for ease of evaluation.&#x20;

ChemSampler runs as many rounds as specified in the `params.json` file or until the maximum number of desired molecules has been achieved. In each round, ChemSampler will evaulate whether the <mark style="color:green;">seed\_smiles</mark> is still providing enough good candidates (new unique molecules) or whether a new <mark style="color:purple;">input\_smiles</mark> needs to be selected from the pool of generated molecules. The ranking of newly generated molecules is done by similarity search using <mark style="color:yellow;">molecular descriptors</mark> specified by the user. If the user is interested in small modifications of the seed\_smiles, 2D descriptors should be prioritized. On the contrary for scaffold hopping exercises, 3D and pharmacophore descriptors should be prioritized. The molecule most similar to the <mark style="color:green;">seed\_smiles</mark> according to the selected descriptors will be used as <mark style="color:purple;">input\_smiles</mark> for the next round. A schema is shown below:

<figure><img src="../.gitbook/assets/Screenshot from 2023-12-14 14-37-53.png" alt=""><figcaption><p>General schema of a round of ChemSampler generation</p></figcaption></figure>

### The parameters file

The parameters file is a `.json` file that allows the user to specify:

```json
{
    "seed_smiles": <smiles of origin used for the generation of new candidates>,
    "keep_smiles": <smiles of a substructure to maintain>,
    "avoid_smiles": <smiles of a substructure to discard in the generative process>,
    "samplers": [ <list of generative models (ersilia identifier code)> ],
    "descriptors": [ <list of molecular descriptors (ersilia identifier code)> ],
    "num_samples": <maximum number of candidates to generate>,
    "max_rounds": <maximum number of rounds>,
    "time_budget_sec": <maximum time spent per round>,
    "saturation_number": <minimum number of new candidates obtained from the input smiles to continue using the same input in the next round>,
    "output_folder": <folder where results will be stored>
}
```

_\* If keep\_smiles or avoid\_smiles are specified, all molecules that do not fulfill either criteria will be removed, which potentially could lead to low number of generated candidates_

#### Input

A seed\_smiles, a list of samplers and a list of descriptors. All of those must be specified in the parameters file.

#### Output

* A list of generated/sampled molecules and its similarity to the seed\_smiles. Similarity is calculated using the molecular descriptors and either euclidean distances or tanimoto similarity.
* A info.json file where information about each round (how many molecules were generated, how many of those were new, which was the input smiles...) is stored
* A sample of 100 molecules randomly selected to be drawn for the user to evaluate the performance of each round

## Available Samplers

<table><thead><tr><th width="123">EOSID</th><th width="179">Slug</th><th>Description</th></tr></thead><tbody><tr><td>eos1d7r</td><td>small-world-zinc</td><td>Small World is an index of chemical space containing more than 230B molecular substructures. Here we use the Small World API to post a query to the SmallWorld server. We sample 100 molecules within a distance of 10 specifically for the Wuxi map, not the entire SmallWorld domain. Please check other small-world models available in our hub.</td></tr><tr><td>eos3kcw</td><td>small-world-wuxi</td><td>Small World is an index of chemical space containing more than 230B molecular substructures. Here we use the Small World API to post a query to the SmallWorld server. We sample 100 molecules within a distance of 10 specifically for the ZINC map, not the entire SmallWorld domain. Please check other small-world models available in our hub.</td></tr><tr><td>eos9ueu</td><td>small-world-enamie-real</td><td>Small World is an index of chemical space containing more than 230B molecular substructures. Here we use the Small World API to post a query to the SmallWorld server. We sample 100 molecules within a distance of 10 specifically for the ZINC map, not the entire SmallWorld domain. Please check other small-world models available in our hub.</td></tr><tr><td>eos1noy</td><td>chembl-sampler</td><td>A simple sampler of the ChEMBL database using their API. It looks for similar molecules to the input molecule and returns a list of 100 molecules by default. This model has been developed by Ersilia. It posts queries to an online server.</td></tr><tr><td>eos2hzy</td><td>pubchem-sampler</td><td>A simple sampler of the PubChem database using their API. It looks for similar molecules to the input molecule and returns a list of 100 molecules by default. This model has been developed by Ersilia and posts queries to an online server.</td></tr><tr><td>eos8fma</td><td>stoned-sampler</td><td>The STONED sampler uses small modifications to molecules represented as SELFIES to perform a search of the chemical space and generate new molecules. The use of string modifications in the SELFIES molecular representation bypasses the need for large amounts of data while maintaining a performance comparable to deep generative models.</td></tr><tr><td>eos4qda</td><td>fasmifra</td><td>FasmiFra is a molecular generator based on (deep)SMILES fragments. The authors use Deep SMILES to ensure the generated molecules are syntactically valid, and by working on string operations they are able to obtain high performance (>340,000 molecule/s). Here, we use 100k compounds from ChEMBL to sample fragments. Only assembled molecules containing one of the fragments of the input molecule are retained.</td></tr><tr><td>eos9taz</td><td>moler-enamine-fragmens</td><td>MoLeR is a graph-based generative model that combines fragment-based and atom-by-atom generation of new molecules with scaffold-constrained optimization. It does not depend on generation history and therefore MoLeR is able to complete arbitrary scaffolds. The model has been trained on the GuacaMol dataset. Here we sample a fragment library from Enamine.</td></tr><tr><td>eos633t</td><td>moler-enamine-blocks</td><td>MoLeR is a graph-based generative model that combines fragment-based and atom-by-atom generation of new molecules with scaffold-constrained optimization. It does not depend on generation history and therefore MoLeR is able to complete arbitrary scaffolds. The model has been trained on the GuacaMol dataset. Here we sample the 300k building blocks library from Enamine.</td></tr></tbody></table>

## Available molecular descriptors



<table><thead><tr><th width="130">EOSID</th><th width="182">Slug</th><th>Description</th></tr></thead><tbody><tr><td>eos4wt0</td><td>morgan-fps</td><td>The Morgan Fingerprints are one of the most widely used molecular representations. They are circular representations (from an atom,search the atoms around with a radius n) and can have thousands of features. This implementation uses the RDKit package and is done with radius 3 and 2048 dimensions,providing a binary vector as output. For Morgan counts, see eos5axz.</td></tr><tr><td>eos5axz</td><td>morgan-counts</td><td>The Morgan Fingerprints, or extended connectivity fingerprints (ECFP4) are one of the most widely used molecular representations. They are circular representations (from an atom, search the atoms around with a radius n) and can have thousands of features. This implementation uses the RDKit package and is done with radius 3 and 2048 dimensions.</td></tr><tr><td>eos8a4x</td><td>rdkit-descriptors</td><td>A set of 200 physicochemical descriptors available from the RDKIT, including molecular weight, solubility and druggability parameters. We have used the DescriptaStorus selection of RDKit descriptors for simplicity.</td></tr><tr><td>eos7jio</td><td>rdkit-fingerprint</td><td>Path-based fingerprints calculated with the RDKit package Chem.RDKFingerprint. It is inspired in the Daylight fingerprint. As explained in the RDKit Book, the fingerprinting algorithm identifies all subgraphs in the molecule within a particular range of sizes, hashes each subgraph to generate a raw bit ID, mods that raw bit ID to fit in the assigned fingerprint size, and then sets the corresponding bit.</td></tr><tr><td>eos78ao</td><td>mordred</td><td>A set of ca 1,800 chemical descriptors, including both RDKit and original modules. It is comparable to the well known PaDEL-Descriptors (see eos7asg), but has shorter calculation times and can process larger molecules.</td></tr><tr><td>eos4u6p</td><td>cc-signaturizer</td><td>A set of 25 Chemical Checker bioactivity signatures (including 2D &#x26; 3D fingerprints, scaffold, binding, crystals, side effects, cell bioassays, etc) to capture properties of compounds beyond their structures. Each signature has a length of 128 dimensions. In total, there are 3200 dimensions. The signaturizer is periodically updated. We use the 2020-02 version of the signaturizer.</td></tr><tr><td>eos7w6n</td><td>grover-embedding</td><td>GROVER is a self-supervised Graph Neural Network for molecular representation pretrained with 10 million unlabelled molecules from ChEMBL and ZINC15. The model provided has been pre-trained on 10 million molecules (GROVERlarge). GROVER has then been fine-tuned to predict several activities from the MoleculeNet benchmark, consistently outperforming other state-of-the-art methods for serveral benchmark datasets.</td></tr><tr><td>eos3ae6</td><td>whales-descriptor</td><td>Weighted Holistic Atom Localization and Entity Shape (WHALES) is a descriptors based on 3D structure to facilitate natural product featurization. It is aimed at scaffold hopping exercises from natural products to synthetic compounds</td></tr><tr><td>eos4x30</td><td>pmapper-3d</td><td>The pharmacophore mapper (pmapper) identifies common 3D pharmacophores of active compounds against a specific target and uniquely encodes them with hashes suitable for fast identification of identical pharmacophores. The obtained signatures are amenable for downstream ML tasks.</td></tr><tr><td>eos2gw4</td><td>eosce</td><td>Bioactivity-aware chemical embeddings for small molecules. Using transfer learning, we have created a fast network that produces embeddings of 1024 features condensing physicochemical as well as bioactivity information The training of the network has been done using the FS-Mol and ChEMBL datasets, and Grover, Mordred and ECFP descriptors</td></tr><tr><td>eos7asg</td><td>padel</td><td>PaDEL is a commonly used molecular descriptor. It calculates 1875 molecular descriptors (1444 1D and 2D descriptors, 431 3D descriptors) and 12 types of fingerprints for small molecule representation. </td></tr></tbody></table>
