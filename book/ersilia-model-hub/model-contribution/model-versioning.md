# Model versioning

Every model in the Hub has a version, `vMAJOR.MINOR.PATCH`, following [semantic versioning](https://semver.org/). The first release is `v1.0.0`. Each release is a GitHub release of the model's repository, and its Docker image is tagged with the same version, so a version can be cited and reused.

Releases are automatic. Every pull request to a model's `main` branch carries exactly one version label, and after the merge Ersilia's workflows release the model from it. Nobody creates releases by hand.

## Version labels

| Label | Use it when | After the merge | From `v1.2.3` |
|---|---|---|---|
| `semver:major` | The outputs change: values, columns or rows | The model is tested, rebuilt and released as `vN+1.0.0` | `v2.0.0` |
| `semver:minor` | The code, dependencies or checkpoints change, and the outputs stay the same | The model is tested, rebuilt and released as `vN.M+1.0` | `v1.3.0` |
| `semver:patch` | Only the metadata changes | The model is released as `vN.M.P+1`. Its image is retagged, not rebuilt, and the new metadata reaches the Hub | `v1.2.4` |
| `no-release` | Nothing that reaches the model changes, such as the workflows or git settings | No tests, rebuild or release | no new version |

If several rows apply, use the highest. Choose the label by the effect of the change on people who use the model, not by which files changed. The SIF image is built only for major and minor releases, see [Apptainer and Singularity images](../developer-docs/packaging-and-distribution/apptainer-singularity-images.md).

A **new incorporation** always uses `semver:major`, because it adds output columns to an empty template. Its first release is still `v1.0.0`: a repository with no releases starts there, whatever the label. See [Open the pull request](model-incorporation-workflow.md) in the incorporation workflow.

Adding a label needs triage permission on the model's repository. If you're an external contributor, an Ersilia maintainer adds it for you.

A pull request merged without a version label is rebuilt but not released, unless it is the first release of a new model. A run started by hand never releases either. Ersilia maintainers check that the label is there before merging.

## Choose the label

### `semver:major`

Changes that can break existing uses of the model:

* The checkpoints change: retraining, fine-tuning or new weights. A conversion of the same weights to another file format is also MAJOR, unless you compared the outputs and they match (see MINOR).
* The output columns change: a column is added, removed, renamed or reordered, or its type changes.
* The model returns a different kind of result for the same input. For example, inputs that used to return a value now return an empty row.
* The outputs for inputs that already returned a value change beyond numerical precision, whatever the cause: a dependency update, a preprocessing change, or a fix that brings the outputs back in line with the original authors' code.
* For generative models, a change to how molecules are produced: a different sampling algorithm, model or selection criteria.

### `semver:minor`

Changes to the code, dependencies or Docker image that keep the same output columns and the same outputs:

* Dependency updates, and the rebuilds they need, where the outputs match the previous release.
* Serving and performance changes, such as loading the model once in `load_tool()` and reusing it between batches, faster code, memory that no longer grows with the input, smaller images, or an added GPU option. GPU outputs may differ slightly from CPU outputs; report the difference.
* Bug fixes that make the model work as documented without changing outputs that users already have. For example, inputs that used to fail now return a result. If the fix also changes values for inputs that already worked, it's MAJOR.
* A conversion of the same weights to another file format, such as ONNX, TorchScript, npz or JSON, when the outputs match.
* Moving checkpoints into `model/checkpoints/`, because the paths in the code change.
* For generative models, changes that make the output match its description without changing the method: removing invalid SMILES, duplicates or copies of the input, or ordering each row by similarity to the input. This is MINOR only if the output columns stay the same and every input that returned molecules before still returns at least one. Removed molecules leave empty cells at the end of the row.

### `semver:patch`

No change to the code, dependencies, checkpoints or outputs:

* Metadata and README updates, including corrections to the license, task or output fields.
* Changes to the descriptions or direction in `run_columns.csv`, if the column names, order and types stay the same.

### `no-release`

Nothing that reaches the model's image or outputs changes: only the workflows in `.github/`, `.gitignore`, `.gitattributes`, `CLAUDE.md` or `AGENTS.md`.

Moving checkpoints between Git and eosvc is not `no-release`, even when the files are identical: the check sees changed files in `model/checkpoints/` and asks for at least `semver:minor`. Check that every file's sha256 is unchanged, and say so in the pull request.

## Numerical precision

Two versions match when, on the same molecules, every output value differs by at most 1e-4. For a MINOR or MAJOR change, run the same molecules through the previous release and the new version, and report the largest difference in the pull request. For generative models, compare summary statistics for the same inputs instead: the share of valid, unique and novel molecules, and the distribution of similarity to the input.

The release notes link to the pull request, so this is where people will look for what changed.

## The automatic check

The **Check version label** check runs on every pull request. It works out the minimum label from the files that changed:

| Change | Minimum |
|---|---|
| Only `.github/`, `.gitignore`, `.gitattributes`, `CLAUDE.md` or `AGENTS.md` | `no-release` |
| `metadata.yml` (or `metadata.json`), `README.md`, `LICENSE` or `access.json` | `semver:patch` |
| `run_columns.csv`, descriptions or direction only | `semver:patch` |
| `run_columns.csv`, column names, order or types | `semver:major` |
| `run_output.csv`, values beyond 1e-4, or columns or rows | `semver:major` |
| `run_input.csv` | `semver:minor` (the outputs can't be compared, so check by hand) |
| Anything else: code, dependencies or checkpoints | `semver:minor`, and the model is run on the example input |

When the model is run, its outputs are compared with the previous `run_output.csv`. Values that differ by more than 1e-4 raise the minimum to MAJOR. The model isn't run for Online models, or when `run_input.csv` also changed. For models with `Output Consistency: Variable`, only the columns and the number of rows are compared, not the values.

The check fails if the label is missing, if there's more than one, if it's lower than the minimum, or if the model can't be run on the example input. A higher label is allowed, and sometimes needed: the check only sees the three example molecules, so new checkpoints that happen to give the same outputs on them still need `semver:major`.
