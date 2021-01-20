---
layout: post
title: Deploy Kubernetes on Jetstream with Kubespray
categories: [kubernetes, kubespray, jetstream]
slug: kubernetes-jetstream-kubespray
---

**This tutorial is obsolete, check the [updated version of the tutorial](https://zonca.dev/2021/01/kubernetes-jetstream-kubespray.html)**

This is an update to previous tutorials, focused on deploying Kubernetes 1.17.6 (released in May 2020, based on 1.17.0 released in December 2019).

We will use Kubespray 2.13.1, which first runs `terraform` to create the Openstack resources,
then `ansible` to configure the servers to run all the Kubernetes services.

## Create Jetstream Virtual machines with Terraform

Terraform allows to execute recipes that describe a set of OpenStack resources and their relationship. In the context of this tutorial, we do not need to learn much about Terraform, we will configure and execute the recipe provided by `kubespray`.

### Requirements

On a Ubuntu 18.04 install `python3-openstackclient` with APT.
Any other platform works as well, also install `terraform` by copying the correct binary to `/usr/local/bin/`, see <https://www.terraform.io/intro/getting-started/install.html>.
Install the newest 0.12.x release, I tested with `0.12.26`.

### Request API access

In order to make sure your XSEDE account can access the Jetstream API, you need to contact the Helpdesk, see the [instructions on the Jetstream Wiki](https://iujetstream.atlassian.net/wiki/spaces/JWT/pages/39682057/Using+the+Jetstream+API). You will also receive your **TACC** password, which could be different than your XSEDE one (username is generally the same).

Login to the TACC Horizon panel at <https://tacc.jetstream-cloud.org/dashboard>, this is basically the low level web interface to OpenStack, a lot more complex and powerful than Atmosphere available at <https://use.jetstream-cloud.org/application>. Use `tacc` as domain, your TACC username (generally the same as your XSEDE username) and your TACC password.

First choose the right project you would like to charge to in the top dropdown menu (see the XSEDE website if you don't recognize the grant code).

Click on Compute / API Access and download the OpenRC V3 authentication file to your machine. Source it typing:

    source XX-XXXXXXXX-openrc.sh

it should ask for your TACC password. This configures all the environment variables needed by the `openstack` command line tool to interface with the Openstack API.

Test with:

    openstack flavor list

This should return the list of available "sizes" of the Virtual Machines.

### Clone kubespray

I needed to make a few modifications to `kubespray` to adapt it to Jetstream:

    git clone https://github.com/zonca/jetstream_kubespray
    git checkout -b branch_v2.13.1 origin/branch_v2.13.1

See an [overview of my changes compared to the standard `kubespray` release 2.13.1](https://github.com/zonca/jetstream_kubespray/pull/13).

### Run Terraform

Inside `jetstream_kubespray`, copy from my template:

    export CLUSTER=$USER
    cp -LRp inventory/zonca inventory/$CLUSTER
    cd inventory/$CLUSTER

Open and modify `cluster.tfvars`, choose your image and number of nodes.
Make sure to change the network name to something unique, like the expanded form of `$CLUSTER_network`.

You can find suitable images (they need to be JS-API-Featured, you cannot use the same instances used in Atmosphere):

    openstack image list | grep "JS-API"

The default is `JS-API-Featured-Ubuntu18-Latest`.

I already preconfigured the network UUID both for IU and TACC, but you can crosscheck
looking for the `public` network in:

    openstack network list

Initialize Terraform:

    bash terraform_init.sh

Create the resources:

    bash terraform_apply.sh

The last output log of Terraform should contain the IP of the master node `k8s_master_fips`, wait for it to boot then
SSH in with:

    ssh ubuntu@$IP

or `centos@$IP` for CentOS images.

Inspect with Openstack the resources created:

    openstack server list
    openstack network list

You can cleanup the virtual machines and all other Openstack resources (all data is lost) with `bash terraform_destroy.sh`.

## Install Kubernetes with `kubespray`

Change folder back to the root of the `jetstream_kubespray` repository,

First make sure you have a recent version of `ansible` installed, you also need additional modules,
so first run:

    pip install -r requirements.txt

It is useful to create a `virtualenv` and install packages inside that.
This will also install `ansible`, it is important to install `ansible` with `pip` so that the path to access the modules is correct. So remove any pre-installed `ansible`.

Or install `ansible` with `conda`, the minimum required version of `ansible` is 2.9.


Then following the [`kubespray` documentation](https://github.com/kubernetes-incubator/kubespray/blob/master/contrib/terraform/openstack/README.md#ansible), we setup `ssh-agent` so that `ansible` can SSH from the machine with public IP to the others:

    eval $(ssh-agent -s)
    ssh-add ~/.ssh/id_rsa

Test the connection through ansible:

    ansible -i inventory/$CLUSTER/hosts -m ping all

If a server is not answering to ping, first try to reboot it:

    openstack server reboot $CLUSTER-k8s-node-nf-1

Or delete it and run `terraform_apply.sh` to create it again.

check `inventory/$CLUSTER/group_vars/all.yml`, in particular `bootstrap_os`, I setup `ubuntu`, change it to `centos` if you used the Centos 7 base image.

Finally run the full playbook, it is going to take a good 10 minutes:

    bash k8s_install.sh

If the playbook fails with "cannot lock the administrative directory", it is due to the fact that the Virtual Machine is automatically updating so it has locked the APT directory. Just wait a minute and launch it again. It is always safe to run `ansible` multiple times.

If the playbook gives any error, try to retry the above command, sometimes there are temporary failed tasks, Ansible is designed to be executed multiple times with consistent results.

You should have now a Kubernetes cluster running, test it:

```
$ ssh ubuntu@$IP
$ sudo su
$ kubectl get pods --all-namespaces
NAMESPACE       NAME                                                   READY     STATUS    RESTARTS   AGE
cert-manager    cert-manager-597bc67bf9-cf7xv                 1/1     Running   0          2m9s
ingress-nginx   ingress-nginx-controller-tfqbn                1/1     Running   0          2m31s
kube-system     coredns-76798d84dd-hhjs9                      1/1     Running   0          98s
kube-system     coredns-76798d84dd-l6fh2                      1/1     Running   0          83s
kube-system     dns-autoscaler-85f898cd5c-kld9w               1/1     Running   0          90s
kube-system     kube-apiserver-dummy2-k8s-master-1            1/1     Running   1          5m11s
kube-system     kube-controller-manager-dummy2-k8s-master-1   1/1     Running   0          5m11s
kube-system     kube-flannel-tzj4h                            1/1     Running   0          3m13s
kube-system     kube-flannel-xqq4j                            1/1     Running   0          3m13s
kube-system     kube-proxy-tfpzd                              1/1     Running   0          3m39s
kube-system     kube-proxy-z2djx                              1/1     Running   0          3m39s
kube-system     kube-scheduler-dummy2-k8s-master-1            1/1     Running   1          5m11s
kube-system     kubernetes-dashboard-77475cf576-ht5th         1/1     Running   0          82s
kube-system     kubernetes-metrics-scraper-747b4fd5cd-sdxsc   1/1     Running   0          82s
kube-system     nginx-proxy-dummy2-k8s-node-nf-1              1/1     Running   0          3m21s
kube-system     nodelocaldns-sg8x4                            1/1     Running   0          86s
kube-system     nodelocaldns-vljgx                            1/1     Running   0          86s
```

Compare that you have all those services running also in your cluster.
We have also configured NGINX to proxy any service that we will later deploy on Kubernetes,
test it with:

```
$ wget localhost
--2018-09-24 03:01:14--  http://localhost/
Resolving localhost (localhost)... 127.0.0.1
Connecting to localhost (localhost)|127.0.0.1|:80... connected.
HTTP request sent, awaiting response... 404 Not Found
2018-09-24 03:01:14 ERROR 404: Not Found.
```

Error 404 is a good sign, the service is up and serving requests, currently there is nothing to deliver.
Finally test that the routing through the Jetstream instance is working correctly by opening your browser
and test that if you access `js-XX-XXX.jetstream-cloud.org` you also get a `default backend - 404` message.
If any of the tests hangs or cannot connect, there is probably a networking issue.

## (Optional) Setup kubectl locally

We also set `kubectl_localhost: true` and `kubeconfig_localhost: true`.
so that `kubectl` is installed on your local machine

Or install it yourself, I tested with 1.18.3.

it also copies `admin.conf` to:

    inventory/$CLUSTER/artifacts

I have a script to copy that to `.config/kube` and to replace the IP with localhost,
because we cannot replace it with the public floating ip because the certificate is not valid for that.

    bash k8s_configure_kubectl_locally.sh

Then make a SSH tunnel (lasts 3 hours):

    bash k8s_tunnel.sh

## (Optional) Setup helm locally

Install helm 3 from [the release page on Github](https://github.com/helm/helm/releases)

I tested with `v3.2.4`.

## Install Jupyterhub

It is preferable to run the Hub and the Proxy on the master node, just in case we
want to downsize the cluster to only one node to save resources.

We need to remove the taint from the master node because currently the JupyterHub recipe doesn't
allow to add tolerations to the master node.

    kubectl edit node $CLUSTER-k8s-master-1

Remove the 3 lines with the taint:

```
taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
```

Now checkout the JupyterHub configuration files repository on the local machine (if you have setup kubectl and helm locally,
otherwise on the master node).

    git clone https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream

Inside that, first run

```
bash create_secrets.sh
```

to create the secret strings needed by JupyterHub then edit its output
`secrets.yaml` to make sure it is consistent, edit the `hosts` lines if needed. For example, supply the Jetstream DNS name of the master node `js-XXX-YYY.jetstream-cloud.org` (XXX and YYY are the last 2 groups of the floating IP of the instance AAA.BBB.XXX.YYY).

    bash configure_helm_jupyterhub.sh
    kubectl create namespace jhub
    bash install_jhub.sh

Check some preliminary pods running with:

    kubectl get pods -n jhub

Once the `proxy` is running, even if `hub` is still in preparation, you can check
in browser, you should get "Service Unavailable" which is a good sign that
the proxy is working.

## Customize JupyterHub

After JupyterHub is deployed and integrated with Cinder for persistent volumes,
for any other customizations, first authentication, you are in good hands as the
[Zero-to-Jupyterhub documentation](https://zero-to-jupyterhub.readthedocs.io/en/stable/extending-jupyterhub.html) is great.

## Setup HTTPS with letsencrypt

Kubespray has the option of deploying also `cert-manager`, but I had trouble deploying an issuer,
it was easier to just deploy it afterwards following [my previous tutorial](https://zonca.dev/2020/03/setup-https-kubernetes-letsencrypt.html)

## Feedback

Feedback on this is very welcome, please open an issue on the [Github repository](https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream) or email me at `zonca` on the domain of the San Diego Supercomputer Center (sdsc.edu).
