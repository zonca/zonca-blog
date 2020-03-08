---
layout: post
title: force local install of python module
date: 2010-12-03 22:18
categories: [python]
slug: force-local-install-of-python-module
---

<code>
 python setup.py install --prefix FOLDER
 <br/>
</code>
<br/>
<br/>
creates lib/python2.6/site-packages, to force a local install you should use:
<br/>
<br/>
<code>
 python setup.py install --install-lib FOLDER
</code>
