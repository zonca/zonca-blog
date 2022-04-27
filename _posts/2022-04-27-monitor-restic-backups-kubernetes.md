---
layout: post
title: Monitor Restic backups on Kubernetes
categories: [jetstream, kubernetes]
---

For one of my production JupyterHub deployments on Kubernetes, I have setup an automated
system to perform nightly backup of the user data, see [the full tutorial on how to set it up](https://zonca.dev/2021/04/jetstream-backup-kubernetes-volumes-object-store.html).

The system writes the backups to Object store using [Restic](https://restic.net/).

In this tutorial I'll provide the configuration to have a CronJob on Kubernetes checking how old are the backups and be alerted if anything is not working using Healthchecks.io.

Healthchecks.io is a free service that gives you a URL endpoint you should regularly send a GET request to, for example from a bash script, if they don't receive a ping after a configurable amount of hours, they send an email.

## Setup the system

First register for a free account at <https://healthchecks.io>, configure an endpoint and get the related URL.

Checkout my usual `zonca/jupyterhub-deploy-kubernetes-jetstream` repository from Github, enter the `backup_volumes` folder.

Make sure you have the Restic password and the AWS-style credentials saved in text files in the same folder, make sure there is no newline at the end of the files (use `vim -b` with `:set noeol`, yes, this is a reminder for myself).

    bash create_aws_secret.sh

Then enter the `backup_volumes/monitor` subfolder:

* Edit `cronjob.yaml` and enter your Healthchecks.io URL
* Run the `backup_is_current.sh` script locally to make sure it works properly
* Run the `create_configmap_scripts.sh` bash script to load the previous script into Kubernetes as a configmap
* Modify the schedule in `cronjob.yaml` to "* * * * *", so every minute it has a chance to run correctly
* Finally run `kubectl apply -f cronjob.yaml` to create the CronJob
* Debug until it works!
* If you login to Healthchecks.io, you should see the GET requests coming in every minute.
* Relax, you'll be notified if backups stop working for any reason.
