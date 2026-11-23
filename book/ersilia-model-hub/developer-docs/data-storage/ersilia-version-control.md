---
description: Ersilia's version control system for large files, or eosvc.
---

# Ersilia version control

[eosvc](https://github.com/ersilia-os/eosvc) keeps large files, such as model checkpoints and training data, in Amazon S3, while the code stays in Git. It uploads and downloads folders, shows the differences between your local copy and S3, and deletes remote files with a confirmation step. It doesn't handle Git itself: use `git` as usual for the code.

## Where files are stored

eosvc works with two kinds of repositories, and tells them apart from the `access.json` file at the repository's root:

| Repository          | Folders managed                                 | Buckets                                       |
| ------------------- | ----------------------------------------------- | --------------------------------------------- |
| Model repository    | `model/checkpoints/` and `model/framework/fit/` | `eosvc-models-public`, `eosvc-models-private` |
| Analysis repository | `data/` and `output/`                           | `eosvc-public`, `eosvc-private`               |

Files are stored under a prefix named after the repository, such as `s3://eosvc-models-public/eos3b5e/model/checkpoints/`. By default, eosvc uses the name of your local folder. If it's different from the repository's name, set the right one:

```bash
export EVC_REPO_NAME="eos3b5e"
```

{% hint style="warning" %}
eosvc does not recognise any folder outside the defined structure for analysis and model repositories.
{% endhint %}

## Install

```bash
git clone https://github.com/ersilia-os/eosvc.git
cd eosvc
pip install -e .
eosvc --help
```

## Credentials

Anyone can download from the public buckets without credentials. Downloading from the private buckets, and any upload or deletion, needs AWS credentials with access to Ersilia's buckets.

{% hint style="info" %}
External contributors don't have AWS credentials. If your model needs eosvc, an Ersilia maintainer will help you upload the files.
{% endhint %}

To save your credentials, run:

```bash
eosvc config \
  --access-key-id "<ACCESS_KEY_ID>" \
  --secret-access-key "<SECRET_ACCESS_KEY>" \
  --region "<REGION>"
```

This saves them in `~/.eosvc/.config`. Add `--session-token` if you use temporary credentials. eosvc also reads the standard AWS environment variables and `~/.aws` files.

## Set up a model repository

List the folders that eosvc manages in `access.json`, with their access level, and keep them out of Git with `.gitignore`. This example stores both folders with eosvc:

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

List only the folders you store with eosvc. For the rules on which folders belong in S3, see [Checkpoints and storage](../../model-contribution/model-template.md#checkpoints-and-storage).

{% hint style="warning" %}
* A folder missing from `access.json` is treated as public.
* An `access.json` file can't mix model folders (`checkpoints`, `fit`) with standard ones (`data`, `output`).
* Access levels are locked after the first upload: eosvc records them in `.eosvc/access.lock.json` and refuses to move files between public and private.
{% endhint %}

## Commands

Run every command from the root of the repository. In model repositories, you can write `checkpoints` and `fit` instead of their full paths.

| Command                             | What it does                                                                                                                         |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `eosvc upload --path checkpoints`   | Uploads a folder or file to S3.                                                                                                      |
| `eosvc download --path checkpoints` | Downloads a folder or file from S3. Use `--path .` to download every managed folder.                                                 |
| `eosvc view`                        | Compares your local files with S3, showing what's in sync, modified, only local, or only remote. Add `--path` to look at one folder. |
| `eosvc delete --path checkpoints`   | Deletes a folder or file from S3, after confirmation. Your local files aren't touched.                                               |

{% hint style="danger" %}
Upload only the folders that go to S3, one at a time, and never run `eosvc upload --path .`. Never keep the same files in both Git and S3: if a folder was committed before you moved it to eosvc, untrack it with `git rm -r --cached model/checkpoints`.
{% endhint %}

## Analysis repositories

Outside model repositories, eosvc manages `data/` and `output/` in the same way, for example in analysis projects:

{% code title="access.json" %}
```json
{
  "data": "public",
  "output": "private"
}
```
{% endcode %}
