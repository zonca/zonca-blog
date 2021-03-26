---
layout: post
title: Stream video from object store on Jetstream
categories: [jetstream]
slug: video-streaming-jetstream
---

In this tutorial we will serve video stored in Jetstream's object store (like Amazon S3)
as HTTP live streaming to a user' browser. This makes it difficult for the user to save
the whole video as it is served in chunks.

## Load a test video on object store

Login to [Horizon](https://iujetstream.atlassian.net/wiki/spaces/JWT/pages/44826638/Using+the+OpenStack+Horizon+GUI+Interface)

Go to Project > Object store

Create a container, for example `videostream`, make it public

Upload a test video, for example:

<https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/720/Big_Buck_Bunny_720_10s_30MB.mp4>

Wait 5 minutes then test you can access the video from your machine and note the base url.

## Deploy the VM

Login to Atmosphere and launch the newest Ubuntu 20.04 image

### Configure SSL

The streaming server will ask for the SSL certificates on setup.

So, install certbot with:

    sudo apt install certbot

and get certificates:

    certbot certonly --standalone -d js-xxx-yyy.jetstream-cloud.org

## Install the streaming server

Kaltura maintains packages for Ubuntu, so it is easy to install it following <https://github.com/kaltura/nginx-vod-module#debianubuntu-deb-package>

For the configuration interactive prompts (I might have set them out of order here):

* Use port 80 instead of 88 (unless you plan to have another NGINX instance on port 80)
* Use port 443 instead of 8443
* For the mode choose "Remote", we will load from object store
* For the remote url set the base url of the object store (before `/swift`) and the port
* Say "No" when it asks if you use the Kaltura media service
* For SSL certificate set `/etc/letsencrypt/live/js-xxx-yyy.jetstream-cloud.org/fullchain.pem`
* For SSL key set `/etc/letsencrypt/live/js-xxx-yyy.jetstream-cloud.org/privkey.pem`

By default it sets the streaming from HTTP instead of HTTPS,
so edit `/opt/kaltura/nginx/conf/vod-remote.conf`,
modify:

    proxy_pass http://media/$1;

into:

    proxy_pass https://media/$1;

If anything goes wrong in the configuration, check the error log at:

    /opt/kaltura/log/nginx/

and reset it:

    sudo apt purge kaltura-nginx
    rm -fr /opt/kaltura
    sudo apt install kaltura-nginx

## Test the streaming

First try to access the stream directly from your browser,
browsers don't know how to display a HLS stream, but if you get to download a file named "index.m3u8" that is
a good sign:

    https://js-xxx-yyy.jetstream-cloud.org/hls/swift/v1/videostream/Big_Buck_Bunny_720_10s_30MB/index.m3u8

In the URL, `videostream` is the name of the bucket (container), then the filename, `index.m3u8` tells the server
that we want to stream that into HLS format.

If you get an error message instead, check the Kaltura NGINX logs.

Finally we can test it into a player, for example VideoJS, go to <https://videojs-http-streaming.netlify.app/>
and paste the URL of your stream.
