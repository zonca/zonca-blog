---
layout: post
title: Deploy JupyterHub on Jetstream 2 on top of Kubernetes
categories: [kubernetes, jupyterhub, jetstream2]
slug: jetstream2-jupyterhub
---

This tutorial is a followup to: [Deploy Kubernetes on Jetstream 2 with Kubespray 2.18.0]({{< relref "2022-03-30-jetstream2_kubernetes_kubespray.md" >}}), so I'll assume Kubernetes is already deployed with a default storageclass.

## Clone the configuration files repository

    git clone https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream

This is the main repository which contains configuration files for all the tutorials I write,
I usually always work with this folder as the root folder.

## Install Jupyterhub

Inside the repository root, first run

```
bash create_secrets.sh
```

to create the secret strings needed by JupyterHub then edit its output
`secrets.yaml` to make sure it is consistent, edit the `hosts` lines if needed. For example, at the moment Jetstream 2 doesn't yet provide DNS entries like Jetstream 1 did (`js-XXX-YYY.jetstream-cloud.org`, they are working on it, I'll update the tutorial when it comes available), so you will need to create a A record that points to the IP of the master node. Most universities offer this service.

    bash configure_helm_jupyterhub.sh
    kubectl create namespace jhub

The newest Kubespray version doesn't install the CSI driver on the master node, so we cannot run the Hub pod on the master node, I have therefore removed the `nodeSelector` and tolerances I had on the configuration for Jetstream 1.

In any case, the Kubernetes ingress automatically handles network routing.

Finally run `helm` to install JupyterHub:

    bash install_jhub.sh

This is installing `zero-to-jupyterhub` `1.2.0`, you can check [on the zero-to-jupyterhub release page](https://github.com/jupyterhub/zero-to-jupyterhub-k8s/releases) if a newer version is available, generally transitioning to new releases is painless, they document any breaking changes very well.

Check pods running with:

    kubectl get pods -n jhub

Once the `proxy` is running, even if `hub` is still in preparation, you can check
in browser, you should get "Service Unavailable" which is a good sign that
the proxy is working.

You can finally connect with your browser to the domain you have configured and
check if the Hub is working fine, after that, the pods running using:

    kubectl get pods -n jhub

shoud be:

```
NAME                              READY   STATUS    RESTARTS   AGE
continuous-image-puller-xlkf6     1/1     Running   0          18m
hub-554bf64f9b-kc2h9              1/1     Running   0          2m26s
jupyter-zonca                     1/1     Running   0          12s
proxy-567d5d9f8d-jr4k9            1/1     Running   0          18m
user-scheduler-79c85f98dd-jpl9l   1/1     Running   0          18m
user-scheduler-79c85f98dd-sg78t   1/1     Running   0          18m
```

## Customize JupyterHub

After JupyterHub is deployed and integrated with Cinder for persistent volumes,
for any other customizations, first authentication, you are in good hands as the
[Zero-to-Jupyterhub documentation](https://zero-to-jupyterhub.readthedocs.io/en/stable/extending-jupyterhub.html) is great.

## Setup HTTPS with letsencrypt

Kubespray has the option of deploying also `cert-manager`, but I had trouble deploying an issuer,
it was easier to just deploy it afterwards following [my previous tutorial](https://zonca.dev/2020/03/setup-https-kubernetes-letsencrypt.html), recently updated.

## Feedback

Feedback on this is very welcome, please open an issue on the [Github repository](https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream).
