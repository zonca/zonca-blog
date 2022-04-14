---
layout: post
title: Custos authentication for JupyterHub
categories: [kubernetes, openstack, jetstream2, jupyterhub]
---

[Custos](https://airavata.apache.org/custos/) is a security middleware used to
authenticate users to Airavata-based Science Gateways.
It is relevant to the Science Gateways community to unify authentication and also
authenticate users to JupyterHub using the same framework.

Custos is a hosted solution managed by Indiana University, therefore it has no maintenance burden
and supports CILogon so that users can login with credentials from almost all US Higher Education
Institutions.

## Requirements

I have been testing on Jetstream 2, however it should work easily on any Kubernetes deployment.
If you also are testing on Jetstream 2, you can follow my previous tutorials to:

* [Deploy Kubernetes](https://zonca.dev/2022/03/kubernetes-jetstream2-kubespray.html)
* [Deploy JupyterHub](https://zonca.dev/2022/03/jetstream2-jupyterhub.html)
* [Deploy Cert-Manager for SSL support](https://zonca.dev/2020/03/setup-https-kubernetes-letsencrypt.html)

## Configure Custos

First you can request a new tenant on the Custos hosted service, for testing you can use the development version at:

<https://dev.portal.usecustos.org/>

then, for production, switch to:

<https://portal.usecustos.org/>

* Login with CILogon (I tested using my XSEDE account)
* Click on create new tenant
* Redirect url <https://your.jupyterhub.com/hub/oauth_callback>
* Domain `your.jupyterhub.com`
* Client URI <https://your.jupyterhub.com>
* Logo URI, anything, for example I used my Github avatar

After having completed the process, you should see that your tenant is in the "Requested" state,
wait for the Custos admins to approve it.

## Configure JupyterHub

Custom authenticators for JupyterHub need to be installed in the Docker image used to run the `hub` pod.
The Custos Authenticator for JupyterHub has [a package on PyPI](https://pypi.org/project/custos-jupyterhub-authenticator/) so it is easy to add it to the JupyterHub image.
See [this issue on zero-to-jupyterhub for more details](https://github.com/jupyterhub/zero-to-jupyterhub-k8s/issues/2265)

The Custos developers maintain Docker images which have this patch already applied, see the [repository on DockerHub]( https://hub.docker.com/r/apachecustos/jupyter-hub-k8/tags).

We can therefore modify the JupyterHub configuration (`config_standard_storage.yaml` in my tutorials) and add:

```
hub:
  image:
    name: apachecustos/jupyter-hub-k8
    tag: 1.2.0
```

Consider that this will need to be updated if we change the version of the Helm recipe (currently 1.2.0).

## Configure the Custos Authenticator

Once the Custos tenant has been approved, we can proceed to configure JupyterHub with the right credentials,
again, modify `config_standard_storage.yaml` to add:

```
hub:
  extraConfig:
      00-custos: |
        from custosauthenticator.custos import CustosOAuthenticator
        c.JupyterHub.authenticator_class = CustosOAuthenticator
        c.CustosOAuthenticator.oauth_callback_url = "https://your.jupyterhub.com/hub/oauth_callback"
        c.CustosOAuthenticator.client_id = "custos-xxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        c.CustosOAuthenticator.client_secret = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        c.CustosOAuthenticator.login_service = "Custos Login"
        c.CustosOAuthenticator.custos_host= "custos.scigap.org"
```

Finally you can test in your browser, you probably need to test with an account different than the one you used to setup the tenant, so for example if you used XSEDE, now use CILogon with your institution or use ORCID.

With this default configuration, any user that can login to Custos can also login to JupyterHub, so if you already have permissions setup for your Custos gateway, those will be also applied to JupyterHub.

## Work in progress

* [Understand if we can login with owner account of tenant](https://github.com/apache/airavata-custos/issues/265).
* [Fix the logout button](https://github.com/apache/airavata-custos/issues/264)
