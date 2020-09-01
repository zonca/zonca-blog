---
layout: post
title: Redirect readthedocs documentation to another website
categories: [python, docs]
---

This is useful if you switch to hosting your own documentation, for example using [Sphinx Multiversion](https://pypi.org/project/sphinx-multiversion/) on Github pages, tutorial coming soon.

We want to be able to redirect from `readthedocs` keeping the relative url.

First we can setup user-defined redirects from the admin page on `readthedocs`,
see the [full documentation](https://docs.readthedocs.io/en/stable/user-defined-redirects.html#exact-redirects),
you can choose "Exact redirect", I only care about redirecting the `latest` version, so:

```
/en/latest/$rest -> https://myorganization.github.io/myrepo/master/
```

`$rest` is a special variable which redirects also all the other pages correctly.

The only issue now is that this redirect only works when the documentation is not found,
therefore I made a temporary commit to `master` which deletes all of the Sphinx pages
of the documentation and replaces `index.rst` with:

```
.. raw:: html

    <script type="text/javascript">
        window.location.replace('https://myorganization.github.io/myrepo/master/');
    </script>
```

After `readthedocs` builds this version, go to <https://github.com/myorganization/myrepo/settings/hooks> and disable
the `readthedocs` web hook.

Finally restore the documentation on your master branch and push.
