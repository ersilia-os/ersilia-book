# Precalculation store

[Isaura](https://github.com/ersilia-os/isaura) is Ersilia's precalculation store. It keeps model predictions permanently, so you, or anyone you share them with, can get results for molecules a model has already seen without running the model again. This makes a big difference where computing power is limited, and Ersilia offers its public precalculations for free.

Unlike [results caching](../running-models/results-caching.md), which keeps predictions on one machine for a few days, Isaura is a long-term store that can be shared through Ersilia's cloud.

## How it's organized

Isaura stores predictions in S3-compatible object storage ([MinIO](https://min.io/)), both on your machine and in Ersilia's cloud. Predictions are organized by model and version, inside buckets:

* **`isaura-public` and `isaura-private`** are the canonical buckets, for predictions anyone can read and for restricted ones.
* **Projects** are other buckets you create, used as staging areas before predictions are moved into a canonical bucket.

## Set up isaura

isaura needs Python 3.10 or later, Git, and Docker.

{% stepper %}
{% step %}
#### Install

```bash
git clone https://github.com/ersilia-os/isaura.git
cd isaura
pip install -e .
```

The first time you run any `isaura` command, it creates a configuration file at `~/.isaura/.env` with default settings.
{% endstep %}

{% step %}
#### Start the local store

With Docker running:

```bash
isaura engine --start
```

This starts MinIO on your machine and creates the `isaura-public` and `isaura-private` buckets. To browse them, open the MinIO console at `http://localhost:9001`. Use `isaura engine` to check its status and `isaura engine --stop` to stop it.
{% endstep %}

{% step %}
#### Connect to Ersilia's cloud (optional)

If you have access to Ersilia's cloud store, add your credentials:

```bash
isaura configure --remote
```

isaura asks for the cloud endpoint and your access keys, and saves them in `~/.isaura/.env`.

{% hint style="danger" %}
Keep your keys private. Don't put them in scripts or files that you commit to a repository.
{% endhint %}
{% endstep %}

{% step %}
#### Check the connection

```bash
isaura configure --test-credentials
```

isaura tests each store you've configured and shows whether it's connected.
{% endstep %}
{% endstepper %}

## Use Isaura with Ersilia

When isaura is installed in the same environment as Ersilia, `ersilia serve` can read from and write to your local store:

```bash
ersilia serve eos3b5e --read_store
ersilia serve eos3b5e --write_store --access public
```

* **`--read_store`** looks up each input in your local `isaura-public` bucket, then in `isaura-private`, and only runs the model for inputs that aren't stored. It uses the latest version of the model's predictions in the store.
* **`--write_store`** saves new predictions to the store. `--access` sets whether they're stored as `public` or `private`.

Ersilia reads only from your local store. To use predictions from Ersilia's cloud, pull them first, as described in [Share predictions through the cloud](precalculation-store.md#share-predictions-through-the-cloud).

## Store and read predictions

Write predictions, from an Ersilia output file, to a project:

```bash
isaura create -pn myproject --access public
isaura write -i eos3b5e_output.csv -m eos3b5e -v v1 -pn myproject
```

The file must have an `input` or `smiles` column. If its name contains a different model identifier, isaura stops before writing anything, to prevent storing predictions under the wrong model.

Read the stored predictions for a list of inputs:

```bash
isaura read -i inputs.csv -m eos3b5e -pn myproject -o outputs.csv
```

Without `-v`, isaura uses the latest version stored. Without `-o`, it shows the results instead of saving them.

To see which of your inputs are already stored, and what's in a project:

```bash
isaura inspect -m eos3b5e -pn isaura-public -i inputs.csv -o available.csv
isaura catalog -pn myproject
```

## Share predictions through the cloud

To publish predictions, write them to a project, move them into the canonical bucket for their access level, and push them:

```bash
isaura write -i eos3b5e_output.csv -m eos3b5e -v v1 -pn myproject
isaura persist -m eos3b5e -v v1 -pn myproject
isaura push -m eos3b5e -v v1 -pn isaura-public
```

To download predictions from the cloud into your local store, for the inputs in a file:

```bash
isaura pull -i inputs.csv -m eos3b5e -pn isaura-public
```

Pushing and pulling need cloud credentials, set with `isaura configure --remote`.

{% hint style="warning" %}
`isaura remove` and `isaura destroy` delete stored predictions. `destroy` can delete a whole project, or one model version inside it. Both ask for confirmation unless you add `--yes`.
{% endhint %}

## Learn more

The isaura repository has the full [CLI and Python API reference](https://github.com/ersilia-os/isaura/blob/main/docs/API_AND_CLI_USAGE.md), an explanation of [how it works](https://github.com/ersilia-os/isaura/blob/main/docs/HOW_IT_WORKS.md), all [configuration variables](https://github.com/ersilia-os/isaura/blob/main/docs/CONFIGURATION.md), and a [troubleshooting guide](https://github.com/ersilia-os/isaura/blob/main/docs/TROUBLESHOOTING.md).
