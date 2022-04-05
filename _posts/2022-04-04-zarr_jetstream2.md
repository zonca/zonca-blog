---
layout: post
title: Use the distributed file format Zarr on Jetstream 2 object storage
date: 2022-04-04 18:00
categories: [jupyter, jetstream2, zarr]
slug: zarr-jetstream2
---

## Zarr

Zarr is a file format designed for cloud computing, see [documentation](http://zarr.readthedocs.io).

Zarr is also supported by [dask](http://dask.pydata.org), the parallel computing framework for Python,
and the Dask team implemented storage backends for [Google Cloud Storage](https://github.com/dask/gcsfs) and
[Amazon S3](https://github.com/dask/s3fs).

## Use OpenStack swift on Jetstream for object storage

Jetstream 2, like Jetstream 1, offers access to object storage via OpenStack Swift.
This is a separate service from the Jetstream Virtual Machines, so you do not need to spin
any Virtual Machine dedicated to storing the data but just use the object storage already
provided by Jetstream. When you ask for an allocation, you can ask for volume storage and object store storage.

## Read Zarr files from object store

If somebody else has already made available some files on object store and set their visibility
to "public", anybody can read them.

See the [example Notebook to read Zarr files](https://gist.github.com/4172aab52ef0cc12623364765e0030f5)

OpenStack Swift already provides an endpoint which has an interface compatible with Amazon S3, therefore
we can directly use the `S3FileSystem` provided by `s3fs`.

Then we can build a `S3Map` object which `zarr` and `dask.array` can access.

In this example I am using the `distributed` scheduler on a single node, you can scale up your computation
having workers distributed on multiple nodes, just make sure that all the workers have access to the
`zarr`, `dask`, `s3fs` packages.

## Write Zarr files or read private files

In this case we need authentication.

First you need to ask to the XSEDE helpdesk API access to Jetstream, this also gives access
to the Horizon interface, which has many advanced features that are not available in Atmosphere.

### Create a bucket

Object store systems are organized on buckets, which are like root folders of our filesystem.
From the Horizon interface, we can choose Object Store -> Containers (quite confusing way of referring to buckets in OpenStack).
Here we can check content of existing buckets or create a new one.

**Make sure you create the bucket on the right project**

### Get credentials

Once you have Jetstream 2 application credentials on your system,
you can first test you can check the content of the bucket we created above:

    openstack object list my_bucket

Now create ec2 credentials with:

	openstack ec2 credentials create

This is going to display AWS access key and AWS secret, we can save credentials in `~/.aws/credentials`
in the machine we want then use to write to object store.
```
[default]
region=RegionOne
aws_access_key_id=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
aws_secret_access_key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Test access

We can check if we can successfully login using `s3fs`, notice we **do not use** `anon=True` as
we did before:

```
import s3fs
fs = s3fs.S3FileSystem(client_kwargs=dict(endpoint_url="https://js2.jetstream-cloud.org:8001/"))
fs.ls("my_bucket")
```

### Generate a file and write it to object store

See a [Notebook example of creating a random array in dask and saving it in Zarr format to Object Store](https://gist.github.com/33b51f74d9252cc3e5d18d290393c33c).
