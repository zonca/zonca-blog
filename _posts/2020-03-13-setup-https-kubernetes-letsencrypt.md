---
layout: post
title: Setup HTTPS on Kubernetes with Letsencrypt
categories: [kubernetes, openstack, jetstream, jupyterhub]
---

This is a follow-up to the Magnum-based deployment running on Jetstream,
see [my recent tutorial about that](https://zonca.github.io/2019/06/kubernetes-jupyterhub-jetstream-magnum.html), however it is not specific to that deployment strategy.

First make sure your payload, for example JupyterHub, is working without HTTPS, so that you check that the ports are open, Ingress is working, and JupyterHub itself can accept connections.

Let's follow the [`cert-manager` documentation](https://cert-manager.io/docs/installation/kubernetes/), for convenience I pasted the commands below:

    kubectl create namespace cert-manager
    kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.14.0/cert-manager-legacy.yaml

Once we have `cert-manager` setup we can create a Issuer in the `jhub` workspace,
(first edit the `yml` and add your email address):

    kubectl create -f setup_https/https_issuer.yml

Then we modify the JupyterHub ingress configuration to use this Issuer,
modify `secrets.yaml` to:

```
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: "nginx"    
    cert-manager.io/issuer: "letsencrypt"
  hosts:
      - js-XXX-YYY.jetstream-cloud.org
  tls:
      - hosts:
         - js-XXX-YYY.jetstream-cloud.org
        secretName: certmanager-tls-jupyterhub
```

Finally update the JupyterHub deployment rerunning the deployment script (no need to delete it):

    bash install_jhub.sh
