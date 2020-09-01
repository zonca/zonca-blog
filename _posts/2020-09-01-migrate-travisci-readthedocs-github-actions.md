---
layout: post
title: Migrate from Travis-CI and Readthedocs to Github actions
categories: [python, docs, github]
---

For a number of years I have been concerned about the duplication of work having to maintain
continuous integration environments both on Travis-CI to run the unit tests and on Readthedocs
to build and host the documentation.

The solution I am proposing is to use Github actions to replace both systems. I chose Github actions
instead of Travis-CI because they are easier to configure, have better log-viewing interface and
surprisingly enough they are better integrated with Github!

The biggest hurdle was to find a way to reproduce the capability of `readthedocs` to host multiple
versions of the documentation together, fortunately the [`sphinx-multiversion`](https://holzhaus.github.io/sphinx-multiversion/master/index.html) gives a very similar functionality.

Next I'll be sharing an example configuration for `Github` actions that integrates:

* Installs requirements both via the package manager and `pip`
* Build and run unit tests for Python 3.6 and 3.7
* Just with Python 3.6, installs `sphinx-multiversion`, builds the docs for all tagged releases, pushes that to the documentation repository

<script src="https://gist.github.com/zonca/78125d40bb6ce70510f0dd4226d947ac.js"></script>

## Requirements

* Create a `docs/requirements.txt` file which includes `sphinx-multiversion`, I directly use `https://github.com/Holzhaus/sphinx-multiversion/archive/master.zip`
* Follow the `sphinx-multiversion` documentation to configure `docs/conf.py`, for example my configuration is:
        ```
# Sphinx multiversion configuration
extensions += ["sphinx_multiversion"]

templates_path = [
            "_templates",                                                                                                                      ]
                                                                                                                                   #html_sidebars = {'**':  [
#            "versioning.html",
#            ]}

# Whitelist pattern for tags (set to None to ignore all tags)                                                                      smv_tag_whitelist = r'^.*
# Whitelist pattern for branches (set to None to ignore all branches)                                                              smv_branch_whitelist = r'^master$'

# Whitelist pattern for remotes (set to None to use local branches only)                                                           smv_remote_whitelist = None

# Pattern for released versions                                                                                                    smv_released_pattern = r'^.*tags.*$'

# Format for versioned output directories inside the build directory                                                               smv_outputdir_format = '{ref.name}'
# Determines whether remote or local git branches/tags are preferred if their output dirs conflict                                 smv_prefer_remote_refs = False
        ```
* If necessary, customize the templates, for example I added `localtoc.html` and `page.html` to mine, see [this gist](https://gist.github.com/ad17f00a91355eaedd221abb34d75c11)
* Decide where you want to host the docs, I created a dedicated repository (same name of the python package replacing `_` by `-`), created a Github deploy key on it, and added it as `DEPLOY_KEY` within the secrets of the Github repository so that it is available in the Github action (`GITHUB_TOKEN` instead is automatically created).
