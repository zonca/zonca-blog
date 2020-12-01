---
layout: post
title: CFITSIO file writing with checkpointing
categories: [astropy]
---

I wrote a prototype implementation of writing FITS files in C
checkpointing using the FITS flush function so that if the code
segfaults, the file is not corrupted, see:

* <https://github.com/zonca/cfitsio_checkpoint_write>
