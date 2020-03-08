---
layout: post
title: tar quickref
date: 2006-09-25 13:19
categories: [linux, bash]
slug: tar-quickref
---

<p>
 compress: tar cvzf foo.tgz *.cc *.h
 <br/>
 check inside: tar tzf foo.tgz | grep file.txt
 <br/>
 extract: tar xvzf foo.tgz
 <br/>
 extract 1 file only: tar xvzf foo.tgz path/to/file.txt
</p>
