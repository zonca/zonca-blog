---
layout: post
title: Renew letsencrypt certificate with NGINX
categories: [letsencrypt]
---

If you have a default domain already configured with NGINX,
the easiest way to renew manually a domain is to temporarily
disable your custom domain and then use `certbot-auto` renew the certificate.

<script src="https://gist.github.com/zonca/4c930f36177fb5f083aede76f3f0d61b.js"></script>
