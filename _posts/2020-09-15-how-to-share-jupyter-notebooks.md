---
layout: post
title: How to share Jupyter Notebooks
categories: [jupyter, notebook, github]
---

I tend to preserve and post online all the work I do for later reference and sharing.
Jupyter Notebooks are a bit more complex than standard source code because they
are natively in a format (JSON) that needs to be rendered to be readable.

Most of the times you want to share a static copy of your "executed" notebook which contains
all the generated plots and outputs.

## Share on gist through the browser

The easiest way to share a Jupyter Notebook is to use a `gist`, which is
quick sharing service provided by Github based on copy-paste.

Navigate to <https://gist.github.com/>, it shows a text-area for copy-pasting
content, you can drag your `ipynb` file from your File Manager into the text-area
and it will paste its content into the area, then you can click on
"Create a public gist" button and it will be uploaded and rendered automatically.
You can also upload multiple notebooks to the same `gist`.

The good thing is that behind this interface there is an actual `git` repository,
so you can later clone and update it using `git`. Or you can even
use the web interface to copy-paste a newer version of your Notebook.

## Share on gist from the command line

If you are working on a remote machine (e.g. HPC), it is way better to setup the [`gh`
client for Github](https://cli.github.com/), then you can directly create a gist from the command line:

    gh gist create --public -d "My description" *.ipynb

and it will print to stdout the gist URL.

## Render larger Notebooks via nbviewer

The Github rendering engine is not the most reliable: it fails for larger notebooks,
it doesn't work on mobile and it doesn't support some JavaScript based libraries, like `altair`.

Then after having uploaded your Notebook to `gist`, you can go to <https://nbviewer.jupyter.org/>,
paste the full link to your `gist` and share the `nbviewer` rendering instead.

## Save clean Notebooks in Github repositories

I also recommend to save a copy of notebooks without any outputs (either doing "Clear all outputs" from
the interface or using `nbstripout`) into a related Github repository, this makes it easier to reference
it later on. However don't save executed notebooks inside Github repositories, they are too large
and cause ugly `diff` between versions (also use `nbdime` to help manage notebooks inside repos).

## Share related group of notebooks

For teaching it is often useful to share a number of related notebooks, in this case you can
create a repository full of notebooks and then point `nbviewer` to the address of that repository,
it will automatically create an index where people can navigate through notebooks.
See for example <https://nbviewer.jupyter.org/github/zonca/mapsims_tutorials/tree/master/>.

For teaching it is often useful to have the main repository which contains clean versions of the
notebooks and then have a Github fork with a version of the notebooks all executed. This is useful
for later reference and for people that have trouble executing the notebooks. Better use a fork
because if we used a `branch`, we would make the main repository un-necessarily heavier to download.

## Make notebooks executable in the browser

Once you have notebooks in a repository, you can also plug them into <https://mybinder.org/> to allow
people to spin a Virtual Machine on Google Cloud automatically and for free and execute them
in their browser. You can also provide a `environment.yml` file to setup the requirements.

See for example [the `healpy` tutorial gist](https://gist.github.com/zonca/9c114608e0903a3b8ea0bfe41c96f255),
which can be executed on Binder clicking on the following Icon: [![Binder logo](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gist/zonca/9c114608e0903a3b8ea0bfe41c96f255/master)
