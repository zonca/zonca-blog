---
layout: post
title: Deploy Dask Gateway with JupyterHub on Kubernetes
categories: [kubernetes, openstack, jetstream, jupyterhub, dask]
---

This tutorial follows the work by the Pangeo collaboration,
the main difference is that I prefer to keep JupyterHub and the Dask infrastructure
in 2 separate `Helm` recipes.

I assume to start from a Kubernetes cluster already running and
JupyterHub deployed on top of it via Helm. And SSL encryption also activated (it isn't probably necessary, but I haven't tested that).
I tested on Jetstream, but this is agnostic of that.

## Preparation

Clone on the machine you use to run `helm` and `kubectl` the repository
with the configuration files and scripts:

	git clone https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream/

Then you need to setup one API token, create it with:

	openssl rand -hex 32

Then paste it both in `dask_gateway/config_jupyterhub.yaml` and `dask_gateway/config_dask-gateway.yaml`,
look for the string `TOKEN` and replace it.

## Launch dask gateway

See the [dask gateway documentation](https://gateway.dask.org/install-kube.html) for reference:

	$ helm repo add daskgateway https://dask.org/dask-gateway-helm-repo/
	$ helm repo update

enter the `dask_gateway` folder and run:

	$ bash install_dask-gateway.sh

You might want to check `config_dask-gateway.yaml` for extra configuration options, but for initial setup and testing it shouldn't be necessary.

After this you should see the 3 dask gateway pods running, e.g.:

	$ kubectl -n jhub get pods
	NAME                                       READY   STATUS    RESTARTS   AGE
	api-dask-gateway-64bf5db96c-4xfd6          1/1     Running   2          23m
	controller-dask-gateway-7674bd545d-cwfnx   1/1     Running   0          23m
	traefik-dask-gateway-5bbd68c5fd-5drm8      1/1     Running   0          23m

## Modify the JupyterHub configuration

Only 2 options need to be changed in JupyterHub:

* We need to run a image which has the same version of `dask-gateway` we installed on Kubernetes (currently `0.8.0`)
* We need to proxy `dask-gateway` through JupyterHub so the users can access the Dask dashboard

If you are using my `install_jhub.sh` script to deploy JupyterHub,
you can modify it and add another `values` option at the end, `--values dask_gateway/config_jupyterhub.yaml`.

You can modify the image you are using for Jupyterhub in `dask_gateway/config_jupyterhub.yaml`.

To assure that there are not compatibility issues, the "Client" (JupyterHub session), the dask gateway server, the scheduler and the workers should all have the same version of Python and the same version of `dask`, `distributed` and `dask_gateway`. If this is not possible, you can test different combinations and they might work. The Pangeo notebook image I am using has a `dask` version too new compared to Dask Gateway 0.9.0, so I downgrade it directly in the example Notebook.

Then redeploy JupyterHub:

	bash install_jhub.sh

Check that the service is working correctly,
if open a browser tab and access <https://js-XXX-YYY.jetstream-cloud.org/services/dask-gateway/api/health>, you should see:

	{"status": "pass"}

If this is not working, you can open login to JupyterHub, get a terminal and first check if the service is working:

    >  curl http://traefik-dask-gateway/services/dask-gateway/api/health

Should give:

    {"status": "pass"}


## Create a dask cluster

You can now login to JupyterHub and check you can connect properly to `dask-gateway`:

```python
from dask_gateway import Gateway
gateway = Gateway(
    address="http://traefik-dask-gateway/services/dask-gateway/",
    public_address="https://js-XXX-YYY.jetstream-cloud.org/services/dask-gateway/",
    auth="jupyterhub")
gateway.list_clusters()
```

Then create a cluster and use it:

```python
cluster = gateway.new_cluster()
cluster.scale(2)
client = cluster.get_client()
```

Client is a standard `distributed` client and all subsequent calls to dask will go
through the cluster.

Printing the `cluster` object gives the link to the Dask dashboard.

For a full example and screenshots of the widgets and of the dashboard see:

<https://gist.github.com/zonca/355a7ec6b5bd3f84b1413a8c29fbc877>

(Click on the `Raw` button to download notebook and upload it to your session).
