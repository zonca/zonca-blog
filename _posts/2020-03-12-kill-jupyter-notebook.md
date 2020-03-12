---
layout: post
title: Kill Jupyter Notebook servers
categories: [python, jupyter]
slug: kill-jupyter-notebook
---

Just today I learned how to properly stop previously running Jupyter Notebook
servers, here for future reference:

    jupyter notebook stop

This is going to print all the ports of the currently running servers.
Choose which ones to stop then:

    jupyter notebook stop PORTNUMBER
