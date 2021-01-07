---
layout: post
title: Deploy Kubernetes and JupyterHub on Jetstream with Magnum
categories: [kubernetes, openstack, jetstream, jupyterhub]
slug: kubernetes-jupyterhub-jetstream-magnum
---

This tutorial deploys Kubernetes on Jetstream with Magnum and then
JupyterHub on top of that using [zero-to-jupyterhub](https://zero-to-jupyterhub.readthedocs.io/).

This is an updated version of the [Kubernetes on Jetstream with Magnum tutorial](https://zonca.dev/2019/06/kubernetes-jupyterhub-jetstream-magnum.html) based now on Kubernetes 1.15.7 instead of Kubernetes 1.11, the node images are based on Fedora Atomic 29 and the Jetstream Magnum deployment is now updated to the Openstack Train release.
If you need a newer version of Kubernetes, you can install Kubernetes with Kubespray instead, see [this tutorial](https://zonca.dev/2020/06/kubernetes-jetstream-kubespray.html).

## Setup access to the Jetstream API

First install the OpenStack client, please use these exact versions, also please run at Indiana, the TACC deployment has an older release of Openstack.

    pip install python-openstackclient==3.18 python-magnumclient==2.10

Load your API credentials from `openrc.sh`, check [documentation of the Jetstream wiki for details](https://iujetstream.atlassian.net/wiki/spaces/JWT/pages/39682064/Setting+up+openrc.sh).

You need to have a keypair uploaded to Openstack, this just needs to be done once per account. See [the Jetstream documentation](https://iujetstream.atlassian.net/wiki/spaces/JWT/pages/35913730/OpenStack+command+line) under the section "Upload SSH key - do this once".

## Create the cluster with Magnum

As usual, checkout the repository with all the configuration files on the machine you will use the Jetstream API from, typically your laptop.

    git clone https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream
    cd jupyterhub-deploy-kubernetes-jetstream
    cd kubernetes_magnum


Now we are ready to use Magnum to first create a cluster template and then the actual cluster, edit first `create_cluster.sh` and set the parameters of the cluster on the top. Also make sure to set the keypair name.
Create the network resources and the cluster template, these need to be created just
once, then they can be reused by clusters created later on:

    bash create_network.sh
    bash create_template.sh

Then a cluster can be created with:

    bash create_cluster.sh

The cluster consumes resources when active, it can be switched off with:

    bash delete_cluster.sh

Consider this is deleting all Jetstream virtual machines and data that could be stored in JupyterHub.

I have setup a test cluster with only 1 master node and 1 normal node but you can modify that later.

Check the status of your cluster, after about 10 minutes, it should be in state `CREATE_COMPLETE`:

    openstack coe cluster show k8s

### Configure kubectl locally

Install the `kubectl` client locally, first check the version of the master node:

    openstack server list # find the floating public IP of the master node (starts with 149_
    IP=149.xxx.xxx.xxx
    ssh fedora@$IP
    kubectl version

Now install the same version following the [Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

Now configure `kubectl` on your laptop to connect to the Kubernetes cluster created with Magnum:

    mkdir kubectl_secret
    cd kubectl_secret
    openstack coe cluster config k8s

This downloads a configuration file and the required certificates.

and returns  `export KUBECONFIG=/absolute/path/to/config`

See also the `update_kubectl_secret.sh` script to automate this step, but it requires to already have setup the environment variable.

execute that and then:

    kubectl get nodes

we can also verify the Kubernetes version available, it should now be `1.15.7`:

    kubectl version
    Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.7", GitCommit:"6c143d35bb11d74970e7bc0b6c45b6bfdffc0bd4", GitTreeState:"clean", BuildDate:"2019-12-11T12:34:17Z", GoVersion:"go1.12.12", Compiler:"gc", Platform:"linux/amd64"}

## Configure storage

Magnum configures a provider that knows how to create Kubernetes volumes using Openstack Cinder,
but does not configure a `storageclass`, we can do that with:

    kubectl create -f storageclass.yaml

We can test this by creating a Persistent Volume Claim:

    kubectl create -f persistent_volume_claim.yaml

    kubectl describe pv

    kubectl describe pvc

```
Name:            pvc-e8b93455-898b-11e9-a37c-fa163efb4609
Labels:          failure-domain.beta.kubernetes.io/zone=nova
Annotations:     kubernetes.io/createdby: cinder-dynamic-provisioner
                 pv.kubernetes.io/bound-by-controller: yes
                 pv.kubernetes.io/provisioned-by: kubernetes.io/cinder
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    standard
Status:          Bound
Claim:           default/pvc-test
Reclaim Policy:  Delete
Access Modes:    RWO
Capacity:        5Gi
Node Affinity:   <none>
Message:
Source:
    Type:       Cinder (a Persistent Disk resource in OpenStack)
    VolumeID:   2795724b-ef11-4053-9922-d854107c731f
    FSType:
    ReadOnly:   false
    SecretRef:  nil
Events:         <none>
```

We can also test creating an actual pod with a persistent volume and check
that the volume is successfully mounted and the pod started:

    kubectl create -f ../alpine-persistent-volume.yaml
    kubectl describe pod alpine

### Note about availability zones

By default Openstack servers and Openstack volumes are created in different availability zones. This created an issue with the default Magnum templates because we need to modify the Kubernetes scheduler policy to allow this. Kubespray does this by default, so I created a [fix to be applied to the Jetstream Magnum templates](https://github.com/zonca/magnum/pull/2), this needs to be re-applied after every Openstack upgrade. The Jetstream team has applied these fixes, they are linked here just for reference.

## Install Helm

The Kubernetes deployment from Magnum is not as complete as the one out of Kubespray, we need
to setup the NGINX ingress ourselves.

[Install the Helm client on your laptop](https://helm.sh/docs/using_helm/#installing-helm),
make sure you install Helm 3 or later.

Run `helm ls` and make sure it doesn't give any error message but just an empty result.


## Setup NGINX ingress

We need to have the NGINX web server to act as front-end to the services running inside the Kubernetes cluster.

### Open HTTP and HTTPS ports

First we need to open the HTTP and HTTPS ports on the master node, you can either connect to the Horizon interface,
create new rule named `http_https`, then add 2 rules, in the Rule drop down choose HTTP and HTTPS; or from the command line:

    openstack security group create http_https
    openstack security group rule create --ingress --protocol tcp --dst-port 80 http_https
    openstack security group rule create --ingress --protocol tcp --dst-port 443 http_https

Then you can find the name of the master node in `openstack server list` then add this security group to that instance:

    openstack server add security group  k8s-xxxxxxxxxxxx-master-0 http_https

### Install NGINX ingress with Helm

We prefer to run the NGINX ingress on the master node, in fact in the configuration in `nginx.yaml` specifies:

```
nodeSelector:
    node-role.kubernetes.io/master: ""
```

This is useful to reduce traffic across the cluster, install NGINX using Helm:

    bash install_nginx_ingress.sh

Note, the documentation says we should add this annotation to ingress with `kubectl edit ingress -n jhub`, but I found out it is not necessary:

    metadata:
      annotations:
        kubernetes.io/ingress.class: nginx

If this is correctly working, you should be able to run `curl localhost` from the master node and get a `Default backend: 404` message.

## Install JupyterHub

Finally, we can go back to the root of the repository and install JupyterHub, first create the secrets file:

    bash create_secrets.sh

Then edit `secrets.yaml` and modify the hostname under `hosts` to display the hostname of your master Jetstream instance, i.e. if your instance public floating IP is `aaa.bbb.xxx.yyy`, the hostname should be `js-xxx-yyy.jetstream-cloud.org` (without `http://`).

You should also check that connecting with your browser to `js-xxx-yyy.jetstream-cloud.org` shows `default backend - 404`, this means NGINX is also reachable from the internet, i.e. the web port is open on the master node.

Finally:

    bash configure_helm_jupyterhub.sh
    kubectl create namespace jhub
    bash install_jhub.sh

Connect with your browser to `js-xxx-yyy.jetstream-cloud.org` to check if it works.

We are installing the `zero-to-jupyterhub` helm recipe version `0.9.0` instead of `0.8.2`.

### Allow services on master

By default the new Kubernetes version has 1 taint on the master node:

```
  taints:
  - effect: NoSchedule
    key: dedicated
    value: master
```

The JupyterHub recipe does not allow to automatically set tolerations on the hub and the proxy
pods, therefore if we want to run them on master, the easiest way is to delete that taint
from the master node:

    kubectl edit node NODENAME

## Issues and feedback

Please [open an issue on the repository](https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream/) to report any issue or give feedback. Also you find out there there what I am working on next.

## Acknowledgments

Many thanks to Jeremy Fischer and Mike Lowe for upgrading the infrastructure to the new Magnum and Kubernetes versions and applying the necessary fixes.
