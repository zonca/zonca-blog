---
layout: post
title: Science Gateway with Dask and Zarr
categories: [kubernetes, openstack, jetstream2, jupyterhub, dask]
---

This material was presented on April 2022 at the [MiniGateways 2022 conference](https://sciencegateways.org/minigateways2022) organized by the wonderful Science Gateways Community Institute (SGCI).

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/6S1_T3se828" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

See the slides and the abstract on [Figshare](https://doi.org/10.6084/m9.figshare.19674516.v1), please also cite the Figshare record if you are referencing this material.

In this tutorial, we will glue several intersting technologies together to show a toy Science Gateway deployment which runs inside Kubernetes, uses Dask to scale up distributed computations across multiple workers and writes output data to Object store using the Zarr format.

## Requirements

* [Kubernetes deployment on Jetstream 2](https://zonca.dev/2022/03/kubernetes-jetstream2-kubespray.html)
* Optionally have [JupyterHub installed as well to interact with the gateway locally](https://zonca.dev/2022/03/jetstream2-jupyterhub.html)
* [Dask Gateway to handle Dask clusters](https://zonca.dev/2022/04/dask-gateway-jupyterhub.html)
* AWS style credentials for writing to object store, see [the tutorial about Zarr on Jetstream 2](https://zonca.dev/2022/04/zarr-jetstream2.html)

## Architecture

## Authentication

As usual first checkout the `zonca/jupyterhub-deploy-kubernetes-jetstream` repository.

Dask gateway has been deployed with JupyterHub based autentication, therefore we need to create a Token from the JupyterHub control panel at <https://jupyterhub-address.edu/hub/token>

Request a new token with no expiration named for example `gatewaydaskzarr`, then save it into the `my_aws_config` file, the same used for the AWS credentials:

    jhub_token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

We need to provide AWS credentials both to the gateway application and to the dask workers, so we store them in a Kubernetes Secret:

    cd gateway-dask-zarr
    bash create_secret.sh

We then configure Dask Gateway so it mounts this secret into the workers, add:

    c.KubeClusterConfig.worker_extra_container_config = {
                "envFrom": [
                            {"secretRef": {"name": "awsconfig"}}
                                ]
                            }

to `dask_gateway/config_dask-gateway.yaml` and redeploy Dask Gateway with Helm.

## Deploy the gateway

The gateway itself is a toy Flask app, see the `gateway.py` file in the `gateway-dask-zarr` folder.

We can create a Kubernetes Deployment with:

    kubectl create -f deploy_gateway.yaml

* It pulls the `zonca/gateway-dask-zarr:latest` which has been build with the [Dockerfile in the gateway-dask-zarr folder](https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream/blob/master/gateway-dask-zarr/Dockerfile)
* It mounts the secret with AWS and JupyterHub credentials
* In the initialization phase it creates a Dask cluster with 3 workers that will be used for all the jobs

We can test that the deployment has been successful by logging in to the JupyterHub deployment on Jetstream and running the first cells of [this notebook on Gist](https://gist.github.com/9b65ecde689c30f765688c4bbbf93a62).

## Run jobs

Running the following cells in the test notebook above we can launch a job, every time we send a get request to the URL:

    http://gateway-svc/submit_job/<job_id>

the gateway:

 * Gets a "Client" instance connected to the Dask cluster
 * Prepares a 1000x1000 Zarr array with 100 chunks of size 100x100
 * Instructs the 3 Dask workers to create a random array distributely
 * Instructs the 3 Dask workers to write that array concurrently to Object Store as a Zarr file

Therefore no computation is executed in the Flask App, it is offloaded to the Dask workers. Data transfer as well directly flows from the workers to Object Store.

## Inspect the results

Finally, in the same notebook, we can access Object Store directly, load and plot the data.
