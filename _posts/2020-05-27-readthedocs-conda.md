---
layout: post
title: Minimal example of readthedocs configuration for conda
categories: [python, conda]
---

Prompted by the [announced better support of conda in readthedocs](https://blog.readthedocs.com/better-conda-support/),
more memory! I setup a Python package with `conda` to automatically build the documentation on
[readthedocs](https://readthedocs.org).

I had some trouble because I couldn't find a minimal example that gives all the necessary configuration options,
for example if `python / install` is not provided, the project is not even built.

See [these files on Gist](https://gist.github.com/zonca/c6f29060a2e77cffcfb155dd4ef1a558)

<script src="https://gist.github.com/zonca/c6f29060a2e77cffcfb155dd4ef1a558.js"></script>
