---
layout: post
title: Setup HTTPS on Kubernetes with Letsencrypt
categories: [kubernetes, openstack, jetstream, jupyterhub]
---

**Updated in May 2020**: changes for Kubernetes 1.15

This is a follow-up to the Magnum-based deployment running on Jetstream,
see [my recent tutorial about that](https://zonca.github.io/2019/06/kubernetes-jupyterhub-jetstream-magnum.html), however it is not specific to that deployment strategy.

First make sure your payload, for example JupyterHub, is working without HTTPS, so that you check that the ports are open, Ingress is working, and JupyterHub itself can accept connections.

Let's follow the [`cert-manager` documentation](https://cert-manager.io/docs/installation/kubernetes/), for convenience I pasted the commands below:

    kubectl create namespace cert-manager
    # Kubernetes 1.15+
    $ kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v0.15.0/cert-manager.yaml

    # Kubernetes <1.15
    $ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.0/cert-manager-legacy.yaml


Once we have `cert-manager` setup we can create a Issuer in the `jhub` workspace,
(first edit the `yml` and add your email address):

    kubectl create -f setup_https/https_issuer.yml

After this, we can display all the resources in the `cert-manager` namespace to
check that the services and pods are running:

    kubectl get all --namespace=cert-manager

The result should be something like:

```
NAME                                           READY   STATUS    RESTARTS   AGE
pod/cert-manager-77f4c9d4b-4228j               1/1     Running   0          55s
pod/cert-manager-cainjector-7cd4857fc7-shlpj   1/1     Running   0          56s
pod/cert-manager-webhook-586c9597db-t6fqv      1/1     Running   0          54s

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/cert-manager           ClusterIP   10.254.78.6     <none>        9402/TCP   56s
service/cert-manager-webhook   ClusterIP   10.254.237.64   <none>        443/TCP    56s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cert-manager              1/1     1            1           55s
deployment.apps/cert-manager-cainjector   1/1     1            1           56s
deployment.apps/cert-manager-webhook      1/1     1            1           54s
                                                                                                                                     NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/cert-manager-77f4c9d4b               1         1         1       55s
replicaset.apps/cert-manager-cainjector-7cd4857fc7   1         1         1       56s                                                 replicaset.apps/cert-manager-webhook-586c9597db      1         1         1       54s
```

## Setup JupyterHub

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

After a few minutes we should have a certificate resource available:

```
> kubectl get certificate --all-namespaces

NAMESPACE   NAME                         READY     SECRET                       AGE
jhub        certmanager-tls-jupyterhub   True      certmanager-tls-jupyterhub   11m
```

for newer versions, check the `certificaterequest` resource instead:

```
kubectl get certificaterequest --all-namespaces
NAMESPACE   NAME                                   READY   AGE
jhub        certmanager-tls-jupyterhub-781206586   True    9m5s
```
