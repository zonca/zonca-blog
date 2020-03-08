---
layout: post
title: load arrays from a text file with numpy
date: 2010-01-05 16:32
categories: [python, numpy]
slug: load-arrays-from-text-file-with-numpy
---

<p>
 space separated text file with 5 arrays in columns:
 <br/>
 <br/>
 [sourcecode language="python"]
 <br/>
 ods,rings,gains,offsets,rparams = np.loadtxt(filename,unpack=True)
 <br/>
 [/sourcecode]
 <br/>
 <br/>
 quite impressive...
</p>
