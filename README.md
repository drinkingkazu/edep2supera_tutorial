# edep2supera Tutorials

This is a collection of Jupyter notebooks and tutorials to get you up to speed on [`edep2supera`](https://github.com/DeepLearnPhysics/edep2supera). You can view the tutorial page [here](https://deeplearnphysics.org/edep2supera_tutorial/).

Below is an instruction for how to contribute in this documentation

## Contributing

Install `jupyter-book` and `ghp-import` to contribute to the repository (see below how to contribute). 

```bash
$ pip install -U jupyter-book ghp-import
```

The above command installs at your local area (i.e. `$HOME/.local`). You might have to explicitly add an executable path:
```
export PATH=$HOME/.local/bin:$PATH
```

### Writing content
Create your Markdown files / Jupyter notebooks. Reference them in the TOC (`_toc.yml`). 
See https://jupyterbook.org/intro.html for more guidance on how to write your pages.

For better version control, it is preferred that you write your Jupyter notebook using 
Markdown. A Jupyter notebook written entirely in Markdown needs a YAML frontmatter and looks like this:

````md
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
execution:
  timeout: 240
---

# Your title

```{code-cell}
code that will be executed like a Jupyter notebook cell
```

````

See https://jupyterbook.org/file-types/myst-notebooks.html for more information.

### Building
Every time you want to build:

```bash
$ jupyter-book build book
```

### Previewing Changes

You can now open the file `_build/html/index.html` to preview your changes.

## Updating Github Pages

If you have the right access permissions, you can easily update the Github Pages after building:

```bash
$ ghp-import -n -p -f book/_build/html
```

## Built with
Using the awesome [Jupyter Book](https://jupyterbook.org/) and [Binder](https://mybinder.readthedocs.io/). Hosted on Github Pages.
