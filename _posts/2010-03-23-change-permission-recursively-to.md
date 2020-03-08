---
layout: post
title: change permission recursively to folders only
date: 2010-03-23 17:58
categories: [linux]
slug: change-permission-recursively-to
---

<code>
 find . -type d -exec chmod 777 {} \;
</code>
