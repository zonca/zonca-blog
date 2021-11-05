---
layout: post
title: Kubernetes certifications CKA and CKAD
categories: [kubernetes]
slug: cka-ckad-kubernetes-certification
---

I recently pursued 2 Kubernetes certifications by Linux Foundation:

* Certified Kubernetes Administrator (CKA)
* Certified Kubernetes Application Developer (CKAD)

I have been deploying, managing and using Kubernetes on Jetstream for more than 4 years
([my first tutorial back in 2017](https://zonca.dev/2017/12/scalable-jupyterhub-kubernetes-jetstream.html)).

However I never did any formal training so my knowledge was sparse.
I was extremely useful to follow the 2 classes by Linux Foundation related to the certifications, they gave
me a more systematic view of all parts of Kubernetes.

I decided to follow both, but there is a lot of overlap, so better choose one of the 2,
if you are more interested in using Kubernetes to deploy applications, do only CKAD and the related class,
if you need to administer Kubernetes deployments, take only CKA.

The most important part of the training is the "test session" on Killer.sh, this is a simulation of the real
certification exam and gives you a lot of experience in being fast in navigating the docs and using `kubectl`.
The exam itself also teaches a lot, you are logging into Kubernetes clusters, solving issues and performing real-world tasks.

The certification exam is done via proctoring, but the process is quite painless.
For the exam you really need to be fast and know how to create resources with `kubectl create` instead of writing
YAML every time, go for YAML just for the more complicated resources. I got to the end of both exams with 15 minutes to spare on the total of 2 hours, that I used to debug the questions I couldn't do in the first pass.

## Suggestions for the tests

Have bookmarks ready, I found a set on the web and added a few of my own, see <https://gist.github.com/zonca/b1f7ee0f884cae8011e86a41e6c525d5>, you can also copy the links to the YAML files and do `wget` from the terminal.

### Bash

You need to memorize these variables and aliases to type into `.bashrc`:

```
export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"
alias  kr="kubectl --replace $now -f"
export VISUAL=vim
export EDITOR=vim
```

The variables about `vim` are needed to have `vim` keybindings in TMUX.
`kr` is very useful because you can use it instead of `k apply -f` and you have it ready if you need to modify the YAML and replace the resource.

`$do` is necessary to create the YAML for a resource with `k create` and then go from there.
`$now` just to quickly delete resources.

### Vim

Minimal `.vimrc` for editing YAML files:

    set expandtab
    set shiftwidth=2
    set tabstop=2

### TMux

You already use TMux, so it is useful to have 2 windows, the first one for `kubectl` and the second for `vim`, it is important not to confuse them, you only go configure the environment in the first window, so you can run `kubectl` only there.

In case you use the `screen` mode with `CTRL-A` as I do, no need to memorize the spell, just open `man tmux`, search for `default prefix` and copy paste into `.tmux.conf`:

    set-option -g prefix C-a
    unbind-key C-b
    bind-key C-a send-prefix
