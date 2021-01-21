---
layout: post
title: Autoscaling script for JupyterHub on top of Kubespray
categories: [kubernetes, kubespray, jetstream]
---

I think the most reliable way of deploying Kubernetes on Openstack is Kubespray,
see [my latest tutorial about it focused on Jetstream](https://zonca.dev/2021/01/kubernetes-jetstream-kubespray.html).

The main issue with this strategy, compared to using Magnum, is that it is not supported by the Cluster Autoscaler, so we don't have an automatic way of scaling up and down the deployment based on load.

For testing purposes I developed a small script that implements this, just to set the expectations,
I called it `hacktoscaler`.

It is a bash script that runs on a server (ideally will deploy inside Kubernetes but for now is external),
every minute:

* it checks if there are JupyterHub single user server pending for more than 3 minutes
* if there are and we are below the preconfigured max number of nodes, we launch first terraform and then ansible to add another node, it is a bit slow, it takes about 15 minutes.
* if instead there are empty nodes (over the preconfigured min number of nodes), it waits half-an-hour and if a node is still empty, it calls Ansible to remove it from Kubernetes and then deletes the instance

## Requirements

* All that is needed to run Kubespray, see my latest tutorial linked above
* Symlink the `kubespray` folder inside the `hacktoscaler` folder

## How to run

You should have already checked out my [`jupyterhub-deploy-kubernetes-jetstream`](https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream) repository, cd into the `hacktoscaler` folder.

Check its configuration by editing `hacktoscaler_daemon.sh` and then launch it:

    bash hacktoscaler_daemon.sh

Then you can simulate some users to trigger the scaling (this requires `hubtraf`):

    bash 0_simulate_users.sh

After this you should be able to see the logs of what the `hacktoscaler` is doing.

If you weren't already warned by the name of this script, **USE AT YOUR OWN RISK**, and good luck.
