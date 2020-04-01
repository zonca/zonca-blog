---
layout: post
title: Use the Jetstream object store
categories: [openstack, jetstream]
---

I plan to collect here notes about using the Openstack object store
on Jetstream.

## Get Amazon-style credentials

Most of the ecosystem is used to Amazon S3, so Openstack Swift provides
Amazon compatible APIs, to access those, make sure your openstack client
can authenticate with the correct allocation, then run:

    openstack ec2 credentials create

This prints on the screen the Access and the Secret keys, those can be
used in all tools which expects Amazon APIs.

## Command line access to object store with s3cmd

One of the most convenient tools is `s3cmd`, which allows to list, upload
and download files from object store. It is included in most linux distributions.

First run the interactive configuration tool:

    s3cmd --configure

And set:

* Region: `RegionOne`
* Any password for encryption
* Use HTTPS: Yes
* Do not test
* Save the configuration

Now edit `~/.s3cfg`:

* set `check_ssl_certificate` and `check_ssl_hostname` to `False`
* set `host_base=JETSTREAM_SWIFT_ENDPOINT` where `JETSTREAM_SWIFT_ENDPOINT` is just the hostname, without `https://` and without `/swift/v1`, I prefer not to post publicly check on your Openstack dashboard or email me.

Now you can list the content of buckets/containers:

```
> s3cmd ls
2020-03-11 23:25  s3://data_store

> s3cmd ls s3://data_store
                       DIR   s3://data_store/bbb/
                       DIR   s3://data_store/data/
                       DIR   s3://data_store/fff/
2020-03-27 01:39       500   s3://data_store/nginx-cinder.yaml

> s3cmd put local_file.txt s3://data_store/fff/ggg
```
