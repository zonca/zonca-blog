---
layout: post
title: execute bash script remotely with ssh
date: 2010-01-07 14:37
categories: [linux, bash]
slug: execute-bash-script-remotely-with-ssh
---

<p>
 a bash script launched remotely via ssh does not load the environment, if this is an issue it is necessary to specify --login when calling bash:
 <br/>
 <br/>
 <code>
  ssh user@remoteserver.com 'bash --login life_om/cronodproc' | mail your@email.com -s cronodproc
 </code>
</p>
