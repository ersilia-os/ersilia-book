# Python API

High level overview of the Python API for the Ersilia Model Hub. Detailed information can be found in the [API Reference](https://ersilia-os.github.io/ersilia/source/autodoc/ersilia.api.html#submodules).

## Fetch

```python
from ersilia.api import Model

mdl = Model(model_id="eos3b5e", verbose=True)
mdl.fetch()
```

By default, models are fetched from DockerHub. Make sure to have Docker installed and active on your system. To specify other fetching modes, follow the CLI structure passing them as arguments (for example: `mdl.fetch(from_github=True)`)

To know if a model is already fetched, you can use the following function:

```python
mdl.is_fetched()
```

## Serve

To serve the model, simply use:

```python
mdl.serve()
```

The same flags specified in the CLI can be passed as arguments to the serve command to specify the caching functionalities and others. For more detailed information, see our [API Reference](https://ersilia-os.github.io/ersilia/source/autodoc/ersilia.api.html#ersilia.api.create_api.ErsiliaAPIModel.serve).

## Run

Unlike the CLI, the Python package only allows as input lists of strings. The output will automatically be converted to a pandas dataframe.

```python
input_list = [
    "C1=C(SC(=N1)SC2=NN=C(S2)N)[N+](=O)[O-]",
    "CC(C)CC1=CC=C(C=C1)C(C)C(=O)O"
]

df = mdl.run(input_list)
```

## Info

To get the metadata of a model that is already served, use the info command, which will be saved as a dictionary.

```python
info = mdl.info()
```

## Example

The example command will generate a list of examples randomly by default unless another mode (`deterministic`, `predefined`) is specified.

```python
example_list = mdl.example()
```

## Close

Close the model after using it:

```python
mdl.close()
```

## Delete

Eliminate the model from your system, including docker images:

```python
mdl.delete()
```

### Using the `with` statement

A more concise way to run prediction would be to use the `with` clause, if the model is already fetched. The with clause will automatically serve and close the model.

```python
from ersilia.api import Model

input_list = [
    "C1=C(SC(=N1)SC2=NN=C(S2)N)[N+](=O)[O-]",
    "CC(C)CC1=CC=C(C=C1)C(C)C(=O)O"
]

with Model("eos3b5e") as mdl:
    df = mdl.run(input_list)
```

## Catalog

This command allows users to access a catalog of models available either locally or in the Ersilia Model Hub. To access more information, set the argument `more=True`.

```python
from ersilia.api import Catalog

cat = Catalog()
df = cat.catalog(hub=False) # Local catalog
df = cat.catalog(hub=True) # Entire hub catalog
```
