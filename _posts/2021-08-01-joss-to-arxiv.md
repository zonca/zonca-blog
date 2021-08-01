---
layout: post
title: Upload a JOSS paper to Arxiv
categories: [writing]
---

If you are in Astrophysics, you probably want to have your [JOSS](https://joss.theoj.org/) paper published to the [Arxiv](https://arxiv.org/).

Unfortunately some authors don't get it accepted due to the paper being too short for the Arxiv standard.

Anyway, here I am explaining how to make the upload using Github Actions and Overleaf, so we do not even need a machine with Latex.
I started from the suggestions discussed [in this issue](https://github.com/openjournals/joss/issues/132), where you also find other methods.

## Instructions

* modified Github Action and saved all artifacts, see the [diff of my modification](https://github.com/galsci/pysm/commit/9c91011133329877df685e0f293b2f856a74eee8)
* downloaded JOSS logo from <https://github.com/openjournals/whedon/blob/master/resources/joss/logo.png>
* edited the `tex` file to point to local `logo.png`
* uploaded `paper.tex` `paper.bib` `logo.png` to Overleaf
* downloaded `output.bbl` from Overleaf, renamed to `paper.bbl`
* uploaded `paper.tex`, `logo.png` and `paper.bbl` to arxiv
