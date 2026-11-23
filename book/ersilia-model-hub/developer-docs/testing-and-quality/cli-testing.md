# CLI testing

The testing playground checks that the Ersilia CLI works end to end: it runs commands such as `fetch`, `serve`, and `run` on real models, from different sources and in several Python versions, and checks what each command produces. It's meant for Ersilia developers changing the CLI, and covers more combinations than [`ersilia test`](model-testing.md), which tests one model.

{% hint style="warning" %}
The playground only works on Linux.
{% endhint %}

## Set up

Install Ersilia from a clone of its repository, with its test dependencies:

```bash
conda create -n ersilia python=3.12
conda activate ersilia
git clone https://github.com/ersilia-os/ersilia.git
cd ersilia
pip install -e ".[test]"
```

## Run the playground

The playground runs in [nox](https://nox.thea.codes/) sessions, from the `test/playground` folder:

```bash
cd test/playground
nox -s execute    # run the tests
nox -s clean      # delete the files and logs the playground created
```

Each session installs Ersilia from your clone in a separate environment, so your own environment isn't touched. The playground keeps its environments, test files, and logs in `~/eos/playground`.

{% hint style="info" %}
nox separates its own options from the playground's options with `--`:

`nox -s execute [nox options] -- [playground options]`
{% endhint %}

## Default behavior

Without options, `nox -s execute` uses the defaults in `test/playground/config.yml`: it runs all commands on `eos3b5e`, in Python 3.10, 3.11, and 3.12, using conda environments.

## nox options

| Option       | Description                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------------ |
| `-s execute` | The session to run. Required.                                                                    |
| `-p 3.12`    | Only run in these Python versions, for example `-p 3.11 3.12`.                                   |
| `-fb venv`   | The environment backend to use instead of conda, such as `venv`, `virtualenv`, `uv`, or `mamba`. |
| `-v`         | Show more output.                                                                                |

## Playground options

These go after `--`, and override the values in `config.yml`.

### What to run

| Option       | Default                   | Description                                                                                                            |
| ------------ | ------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `--cli`      | `all`                     | The commands to run. A single command also runs the ones it needs first: `--cli run` runs `fetch`, `serve`, and `run`. |
| `--runner`   | `single`                  | `single` to test one model, or `multiple` to test several.                                                             |
| `--single`   | `eos3b5e`                 | The model for `single` mode.                                                                                           |
| `--multiple` | `eos5axz eos4e40 eos2r5a` | The models for `multiple` mode.                                                                                        |

### Command options

Each of these passes options to one Ersilia command. For example, `--fetch from_dockerhub` runs `ersilia fetch --from_dockerhub`.

| Option      | Default                  | Example                         |
| ----------- | ------------------------ | ------------------------------- |
| `--fetch`   | Ersilia's default source | `--fetch from_s3`               |
| `--catalog` | `--more --local`         | `--catalog hub`                 |
| `--example` | `--n_samples 10`         | `--example n_samples 5`         |
| `--test`    | `--deep --from_github`   | `--test shallow from_dockerhub` |
| `--delete`  | Not set                  | `--delete all`                  |

### Inputs and outputs

| Option          | Default                  | Description                                                    |
| --------------- | ------------------------ | -------------------------------------------------------------- |
| `--outputs`     | `result.csv` `result.h5` | The output files for `run`, saved in `~/eos/playground/files`. |
| `--input_types` | `csv`                    | The input formats to test with `run`.                          |
| `--num_samples` | `10`                     | The number of input molecules for `run`.                       |

### Settings

| Option                  | Default | Description                                                                                            |
| ----------------------- | ------- | ------------------------------------------------------------------------------------------------------ |
| `--activate_docker`     | `true`  | Start Docker if it isn't running. Set to `false` to test how Ersilia behaves without Docker.           |
| `--max_runtime_minutes` | `10`    | The maximum time `run` can take before the test fails. Use it to catch models that have become slower. |
| `--log_error`           | `true`  | Save the errors of each failing command to a file in `~/eos/playground/logs`.                          |
| `--silent`              | `true`  | Hide Ersilia's own output.                                                                             |
| `--show_remark`         | `false` | Add a remarks column to the summary table.                                                             |

## Examples

Test fetching a model from Docker Hub, in Python 3.12 only:

```bash
nox -s execute -p 3.12 -- --cli fetch --fetch from_dockerhub
```

Run the full cycle on several models:

```bash
nox -s execute -- --runner multiple --cli run
```
