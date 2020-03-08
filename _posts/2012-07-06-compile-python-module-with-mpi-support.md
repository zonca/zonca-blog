---
layout: post
title: compile python module with mpi support
date: 2012-07-06 16:08
slug: compile-python-module-with-mpi-support
---

<p>
 CC=mpicc LDSHARED="mpicc -shared" python setup.py build_ext -i
</p>
