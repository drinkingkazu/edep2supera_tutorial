---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.10.3
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---


# Quick Start
With the right `edep-sim` file and the software environment, running `edep2supera` and generating a `larcv` file is simple. In this example we use a prepared example file which we download below.

```{code-cell}
! wget https://web.stanford.edu/~kterao/edep-sim-example.root
```

## Running `edep2supera`

`edep2supera` installation should come with `run_edep2supera.py` executable. If you installed locally by `pip install --user`, make sure your local installation is exposed to `$PATH` (i.e. `export PATH=$HOME/.local/bin:$PATH`)

```{code-cell}
! run_edep2supera.py
```

The first argument should be the output (`larcv`) filename to be created. The second argument should be the first `edep-sim` file to be converted. You can keep adding more arguments to add more `edep-sim` files.

```{code-cell}
! run_edep2supera.py larcv.root edep-sim-example.root
```

... and now you should have `larcv.root` :)
