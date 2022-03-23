---
layout: post
title: Workflow for Jupyter Notebooks under version control
categories: [jupyter-notebook,git]
---

I'll present here my strategy for keeping Jupyter Notebooks under version control.

TLDR:

* Use `nbstripout` to only commit input cells
* Snapshot executed Notebooks to a Gist with `gh gist create`
* Save link to Gist in commit message
* see below for script to automate

## Requirements

* Github handles ipynb pretty well, so I prefer not to use [`jupytext`](https://github.com/mwouts/jupytext)
* I want small repositories, so only commit the input cells
* I want to save the executed notebooks, not in the repo, but where I can easily reference them if needed

## Workflow

### Notebooks and git

I work on Notebooks as I work with normal Python files, so I always run `black` on them to fix formatting, use `git add -p` to add snippet by snippet. I don't mind the little extra escaping the Notebook introduces.

I have the `nbstripout` filters activated (`nbstripout --install`) so that even if the notebook is partially executed, when I run `git add -p` I only get the patches on the input cells.

Moreover, I configure `nbstripout` [to remove metadata](https://github.com/kynan/nbstripout#stripping-metadata) like the kernelspec or the Python version, which doesn't do by default

### Snapshot executed Notebooks

The Jupyter Notebook inside the repository has only the inputs, but I would like to save executed Notebooks for tracking purposes without increasing the size of the repository.

First I do a clean execution of the Notebook (with Restart & Run All), then I save the changes to the input cells to the repository. I don't need to clear the outputs from the Notebook, `nbstripout` does it on th e fly before submitting changes to git.

Then I post the executed Notebook with all outputs to a Gist from the command line with the Github CLI:

    gh gist create my_notebook.ipynb

Optionally with `--public` to make it show on my Gist profile.

The `gh` tool returns the link to the Gist, that I can add to the commit message or post in a Pull Request or an Issue.

### Add executed Notebooks to the documentation

Once I have the final version of a Notebook, I often use `nbsphinx` to add it to the documentation.

So I disable `nbstripout` with:

    nbstripout --uninstall

Then I add just the last executed state of the Notebook to the repository, so that Sphinx/Readthedocs can compile it into the documentation, including all plots.

## Automation script

I have created a bash script that automates the process:

* call with `snapshot_nb your_notebook.ipynb`
* creates a Gist with the Notebook
* amends the last commit to add a link to it
* it actually works with any text file, even multiple files, so it could be used for log outputs for example

See <https://gist.github.com/2f2eba4f0288ca4079f7f83efa6b9048>
