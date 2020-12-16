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

<iframe width="560" height="315" src="https://www.youtube.com/embed/D5ZrbB2KtXw" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

or [here just the video recording (better quality, no questions/answers)](https://www.youtube.com/watch?v=1ECTVNpvaoo&feature=youtu.be)


On the same topic I also gave a 1-hour webinar focused on introducing how to use Openstack and Kubernetes for people with no previous experience, [the video (April 2020 ECSS Symposium) is available on Youtube](https://www.youtube.com/watch?v=jiYw4g4RX-w)

If you have any question/feedback, reply to [this tweet](https://twitter.com/andreazonca/status/1319145344089767936):

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Tomorrow (Thursday 22nd), I&#39;ll be presenting a paper about:<br><br>&quot;Deployment of <a href="https://twitter.com/hashtag/Kubernetes?src=hash&amp;ref_src=twsrc%5Etfw">#Kubernetes</a> and <a href="https://twitter.com/ProjectJupyter?ref_src=twsrc%5Etfw">@ProjectJupyter</a> Hub on <a href="https://twitter.com/xsede?ref_src=twsrc%5Etfw">@XSEDE</a> <a href="https://twitter.com/jetstream_cloud?ref_src=twsrc%5Etfw">@jetstream_cloud</a>&quot;<br><br>at <a href="https://twitter.com/hashtag/gateways2020?src=hash&amp;ref_src=twsrc%5Etfw">#gateways2020</a> by <a href="https://twitter.com/sciencegateways?ref_src=twsrc%5Etfw">@sciencegateways</a>, 11:45am PDT, see <a href="https://t.co/myaNoUe3s0">https://t.co/myaNoUe3s0</a> <a href="https://t.co/t06kzt1CEe">pic.twitter.com/t06kzt1CEe</a></p>&mdash; Andrea Zonca (@andreazonca) <a href="https://twitter.com/andreazonca/status/1319145344089767936?ref_src=twsrc%5Etfw">October 22, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
