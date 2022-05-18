---
layout: post
title: Remove unique cell id from Jupyter Notebooks
categories: [git, notebook]
---

I know! Jupyter is littering your `git diff` with randomly generated cell ids and `nbstripout` doesn't remove them, (I'm sure they are useful for some reason).

Open the Notebook with your editor of choice, `vim`, then look for `minor` and set `nbformat_minor` to 4:

    "nbformat_minor": 4

Open and save again from Jupyter, cell ids should be gone for good!
