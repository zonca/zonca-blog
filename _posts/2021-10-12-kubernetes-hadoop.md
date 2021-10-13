---
layout: post
title: Deploy Hadoop on Kubernetes on Jetstream
categories: [kubernetes, jetstream]
slug: hadoop-kubernetes-jetstream
---

We are deploying the good old Hadoop on top of Kubernetes on Jetstream. Don't ask why.

As usual we start with a full-fledged Kubernetes deployment on Jetstream (1) [deployed via Kubespray](https://zonca.dev/2021/01/kubernetes-jetstream-kubespray.html)

## Deploy Hadoop via helm

Fortunately we have a Helm chart which deploys all the Hadoop components.
It is deprecated since November 2020, but it still works fine on Kubernetes 1.19.7.

Clone the usual repository with [`gh`](https://cli.github.com/):

    gh repo clone zonca/jupyterhub-deploy-kubernetes-jetstream
    cd hadoop/

Verify the configuration in `stable_hadoop_values.yaml`, I'm currently keeping it simple,
so no persistence.

Install Hadoop via Helm:

    bash install_hadoop.sh

Once the pods are running, you should see:

```
> kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
hadoop-hadoop-hdfs-dn-0   1/1     Running   0          144m
hadoop-hadoop-hdfs-nn-0   1/1     Running   0          144m
hadoop-hadoop-yarn-nm-0   1/1     Running   0          144m
hadoop-hadoop-yarn-rm-0   1/1     Running   0          144m
```

## Launch a test job

Get a terminal on the YARN node manager:

    bash login_yarn.sh

You have now access to the Hadoop 2.9.0 cluster.
Launch a test MapReduce job to compute pi:

    bin/yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.0.jar pi 16 1000

## Access the YARN Dashboard

You can also export the YARN dashboard from the cluster to your local machine.

    bash expose_yarn.sh

Connect locally to port 8088 to check the status of the jobs.

Make sure this port is never exposed publicly. I learned the hard way that there are botnets scanning the internet and compromising the YARN service for crypto-mining, see [this article for details](https://tolisec.com/yarn-botnet/).
