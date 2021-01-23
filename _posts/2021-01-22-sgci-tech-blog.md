---
layout: post
title: Science Gateways Tech blog - Kubernetes and JupyterHub on Jetstream
categories: [kubernetes, kubespray, jetstream]
hide: true
search_exclude: true
slug: sgci-tech-blog
---

This is a guest post by [Andrea Zonca](https://zonca.dev/about/), computational scientist at the [San Diego Supercomputer Center](https://www.sdsc.edu/) and consultant for XSEDE through the [Extended Collaborative Support Services](https://www.xsede.org/for-users/ecss) for Science Gateways.

JupyterHub handles authentication and routing for any number of users to each have access to an interactive computational environment via their browser.

The most impactful application of JupyterHub in the context of Science Gateways is to deploy it as a companion application for the Science Gateway to allow users to access a pre-configured computing environment to allow them to use the Gateway programmatically and apply custom processing of the gateway input or outputs.
For example, the Gateway users could use their login credentials and have access to a Jupyter Notebook environment with tutorial notebooks that show example analysis pipelines that show how they can:

* upload raw data
* use packages installed in the Jupyter Notebook environment to pre-process them to create data in the format suitable for the Science Gateway
* submit the data for processing using remote execution API
* check the job processing status
* retrieve the results
* load the output data from disk
* post-process and plot the results

Let's process now to show an overview on how to deploy first Kubernetes and then JupyterHub on Jetstream.

## Jetstream

Jetstream (and the upcoming [Jetstream 2](https://news.iu.edu/stories/2020/06/iub/releases/01-jetstream-cloud-computing-awarded-nsf-grant.html) is a cloud deployment part of XSEDE, the sciency equivalent of Amazon Elastic Compute Cloud (EC2) or Google Cloud Platform. Many Science Gateways already run on Jetstream.

Jetstream allows each user to programmatically launch Virtual Machines with the desired amount of CPU/RAM resources, a pre-configured OS (Ubuntu, CentOS...), connect them together on a internal network and expose them to the Internet with public IPs. Users have then full administrative access to the machines to install any software package.

## Single server deployment

In case we only need a test deployment with a small amount of users each with very limited computational resources, we can avoid the complications of a distributed deployment.

In this case we can just spawn a single large-memory Virtual Machine on Jetstream and then follow the ["Littlest JupyterHub" documentation to install JupyterHub](https://tljh.jupyter.org/en/latest/install/jetstream.html).

## Kubernetes

However, if the sum of the memory needed by all the users is higher than 120GB, we need to deploy JupyterHub across multiple Jetstream Virtual Machines.
The recommended way of deploying a distributed instance of JupyterHub is on top of Kubernetes, which is able to provide networking/logging/reliability/scalability.

I have configured and adapted the Kubespray tool to deploy Kubernetes to Jetstream and written a [step-by-step tutorial about it](https://zonca.dev/2021/01/kubernetes-jetstream-kubespray.html), which includes all the necessary configuration files.
A Jetstream user can launch this script to programmatically launch any number of Jetstream instances and deploy Kubernetes across them.
Consider also that Kubernetes would also be a good platform to deploy a Science Gateway itself, especially if it has already a distributed architecture (for example a main website and separate workers).

After Kubernetes is deployed, we have now a programmatic interface where we can deploy applications packaged as Docker containers, monitor their execution, log their warnings or errors, and scale them based on load.

## JupyterHub

Finally, we can leverage the [Zero to JupyterHub project](https://zero-to-jupyterhub.readthedocs.io/) and customize their recipe to automatically deploy all the components needed for a distributed JupyterHub deployment on Kubernetes.

We can configure authentication with Github/Google/XSEDE/CILogon, choose a Docker container with all the packages needed by the users, configure a public URL and setup HTTPS.

Once the deployment is complete, the users can point their browser to the master node of the deployment, authenticate and have a Jupyter Notebook be spawned for them across the cluster of Jetstream instances with a dedicated amount of computing resources.

We can also pre-populate the computational environment with tutorial notebooks that display example data analysis pipelines that jointly leverage JupyterHub and the Science Gateway.
After a few hours that a user disconnects, their container is decommissioned to free-up resources for other users, but their data is saved as a permanent disk on Jetstream and is mounted again the next time they reconnect.

## Conclusion

I showed an overview of the different components involved in deploying a distributed instance of JupyterHub on Jetstream, to know more, checkout the list of references below.
If you have any feedback or would like to collaborate, please [open an issue in my `jupyterhub-deploy-kubernetes-jetstream` repository](https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream/issues/new).

## References

* ["Gateways 2020" paper and video recording](https://zonca.dev/2020/09/gateways-2020-paper.html)
* [Video tutorial on Openstack and Kubernetes for the ECSS Symposium](https://www.youtube.com/watch?v=jiYw4g4RX-w)
* [Tutorial on how to deploy Kubernetes and JupyterHub on Jetstream via Kubespray](https://zonca.dev/2021/01/kubernetes-jetstream-kubespray.html)
* [Tutorial on how to setup SSL for HTTPS connection to JupyterHub](https://zonca.dev/2020/03/setup-https-kubernetes-letsencrypt.html)
* [Tutorial on how to deploy Dask for distributed computing](https://zonca.dev/2020/08/dask-gateway-jupyterhub.html)
* [Prototype autoscaler for JupyterHub on Jetstream](https://zonca.dev/2021/01/autoscaling_script_kubespray_jupyterhub.html)
