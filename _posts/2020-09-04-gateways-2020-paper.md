---
layout: post
title: Gateways 2020 paper about Kubernetes and JupyterHub on Jetstream
categories: [jupyterhub, jetstream, gateways, kubernetes]
---

My paper in collaboration with Richard Signell (USGS), Julien Chastang (UCAR), John Michael Lowe (IU/Jetstream), Jeremy Fischer (IU/Jetstream) and Robert Sinkovits (SDSC) was accepted at [Gateways 2020](https://sciencegateways.org/web/gateways2020) (October 12–23, 2020).

It gives an overview of the architecture of the deployments of the container orchestration engine Kubernetes
on the NSF-funded Jetstream Openstack cloud deployment at Indiana University.
It refers to my previously published tutorials for step-by-step instructions and configuration files,
the 2 most important tutorials explain the 2 strategies for deploying Kubernetes on Jetstream:

* [Using Magnum, the Openstack functionality to provide a ready-made Kubernetes cluster](https://zonca.dev/2020/05/kubernetes-jupyterhub-jetstream-magnum.html)
* [Using Terraform and Ansible via `kubespray`](https://zonca.dev/2020/06/kubernetes-jetstream-kubespray.html)

Once Kubernetes is available, it is quite easy to deploy `JupyterHub` configuring properly [`zero-to-jupyterhub`](https://zero-to-jupyterhub.readthedocs.io/en/latest/)

See the [Gateways 2020 paper (open-access) on OSF](https://osf.io/zyhwt/), ([direct link to PDF](https://osf.io/gkz9v/download))

Here is the recording of the presentation and the questions:

> youtube: https://www.youtube.com/watch?v=D5ZrbB2KtXw

or [here just the video recording (better quality, no questions/answers)](https://www.youtube.com/watch?v=1ECTVNpvaoo&feature=youtu.be)


On the same topic I also gave a 1-hour webinar focused on introducing how to use Openstack and Kubernetes for people with no previous experience, [the video (April 2020 ECSS Symposium) is available on Youtube](https://www.youtube.com/watch?v=jiYw4g4RX-w)

If you have any question/feedback, reply to this tweet:

> twitter: https://twitter.com/andreazonca/status/1319145344089767936
