---
layout: post
title: using gnu find
date: 2006-10-03 14:00
categories: [linux, bash]
slug: using-gnu-find
---

<p>
 list all the directories excluding ".":
 <br/>
</p>
<blockquote>
 find . -maxdepth 1 -type d -not -name ".*"
</blockquote>
<br/>
find some string in all files matching a pattern in the subfolders (with grep -r you cannot specify the type of file)
<br/>
<blockquote>
 find . -name '*.py' -exec grep -i pdb '{}' \;
</blockquote>
