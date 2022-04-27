---
layout: post
title: Backup Kubernetes volumes to OpenStorageNetwork object store
categories: [jetstream, kubernetes]
---

In my specific scenario, I have users running JupyterHub on top of Kubernetes
on the Jetstream XSEDE Cloud resouce.
Each user has a persistent volume as their home folder of a few GB.
Instead of snapshotting the entire volume, I would like to only backup the data offsite to OpenStorageNetwork and being able to restore them.

In this tutorial I'll show how to configure [Stash](https://stash.run) for this task.
Stash is has a lot of other functionality, so it is really easy to get lost in their
documentation. This tutorial is for an advanced topic, it assumes good knowledge of Kubernetes.

Stash under the hood uses [`restic`](https://restic.net/) to backup the data, so that we can also manage the backups outside
of Kubernetes, see further down the tutorial. It also automatically decuplicates the data, so if the same file is unchanged in
multiple backups, as it is often the case, it is just stored once and referenced by multiple backups.

All the configuration files are available in the `backup_volumes` folder of [zonca/jupyterhub-deploy-kubernetes-jetstream](https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream/tree/master/backup_volumes)

## Install Stash

First we need to request a free license for the community edition of the software,
I tested with `2021.03.17`, replace as needed with a newer version:

* <https://stash.run/docs/v2021.03.17/setup/install/community/>

Rename it to `license.txt`, then install Stash via Helm:

```
helm repo add appscode https://charts.appscode.com/stable/
helm repo update
bash install_stash.sh
```

## Test object store

I have used object store from OpenStorageNetwork, which is nice as it is offsite, but
also using the Jetstream object store is an option.
Both support the AWS S3 protocol.

It would be useful at this point to test the S3 credentials:

Install the AWS cli `pip install awscli awscli-plugin-endpoint`

Then create a configuration profile at `~/.aws/config`:

```
[plugins]
endpoint = awscli_plugin_endpoint

[profile osn]
aws_access_key_id=
aws_secret_access_key=
s3 =
    endpoint_url = https://xxxx.osn.xsede.org
s3api =
    endpoint_url = https://xxxx.osn.xsede.org
```

Then you can list the content of your bucket with:

    aws s3 --profile osn ls s3://your-bucket-name --no-sign-request

## Configure the S3 backend for Stash

See the [Stash documentation about the S3 backend](https://stash.run/docs/v2021.03.17/guides/latest/backends/s3/). In summary, we should create 3 text files:

* `RESTIC_PASSWORD` with a random password to encrypt the backups
* `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` with the S3 style credentials

Then we can create a Secret in Kubernetes that holds the credentials:

    bash create_aws_secret.sh

Then, customize `stash_repository.yaml` and create the Stash repository with:

    kubectl create -f stash_repository.yaml

Check it was created:

```
> kubectl -n jhub get repository
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
osn-repo                                                                2d15h
```

## Configuring backup for a standalone volume

Automatic and batch backup require a commercial Stash license. With the community version, we can only use the "standalone volume" functionality, which is enough for our purposes.

See the [relevant documentation](https://stash.run/docs/v2021.03.17/guides/latest/volumes/overview/)

Next we need to create a [BackupConfiguration](https://stash.run/docs/v2021.03.17/concepts/crds/backupconfiguration/)

Edit `stash_backupconfiguration.yaml`, in particular you need to specify which `PersistentVolumeClaim` you want to backup, for JupyterHub user volumes, these will be `claim-username`. For testing better leave "each minute" for the schedule, if a backup job is running, the following are skipped.
You can also customize excluded folders.

In order to pause backups, set `paused` to `true`:

    kubectl -n jhub edit backupconfiguration test-backup

`BackupConfiguration` should create a `CronJob` resource:

```
> kubectl -n jhub get cronjob
NAME                       SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-test-backup   * * * * *   True      0        2d15h           2d15h
```

`CronJob` then launches a `BackupSession` for each trigger of the backup:

```
> kubectl -n jhub get backupsession
NAME                     INVOKER-TYPE          INVOKER-NAME   PHASE     AGE
test-backup-1618875244   BackupConfiguration   test-backup    Succeeded   3m13s
test-backup-1618875304   BackupConfiguration   test-backup    Succeeded   2m13s
test-backup-1618875364   BackupConfiguration   test-backup    Succeeded   73s
test-backup-1618875425   BackupConfiguration   test-backup    Running     12s
```

## Monitor and debug backups

You can check the logs of a backup with:

```
> kubectl -n jhub describe backupsession test-backup-1618869996
> kubectl -n jhub describe pod stash-backup-test-backup-1618869996-0-rdcdq
> kubectl -n jhub logs stash-backup-test-backup-1618861992-0-kj2r6
```

Once backups succeed, they should appear on object store:

```
> aws s3 --profile osn ls s3://your-bucket-name/jetstream-backup/snapshots/
2021-04-19 16:34:11        340 1753f4c15da9713daeb35a5425e7fbe663e550421ac3be82f79dc508c8cf5849
2021-04-19 16:35:12        340 22bccac489a69b4cda1828f9777677bc7a83abb546eee486e06c8a8785ca8b2f
2021-04-19 16:36:11        340 7ef1ba9c8afd0dcf7b89fa127ef14bff68090b5ac92cfe3f68c574df5fc360e3
2021-04-19 16:37:12        339 da8f0a37c03ddbb6c9a0fcb5b4837e8862fd8e031bcfcfab563c9e59ea58854d
2021-04-19 16:33:10        339 e2369d441df69bc2809b9c973e43284cde123f8885fe386a7403113f4946c6fa
```

## Restore from backup

Backups are encrypted, so it is not possible to access the data directly from object store. We need to restore it to a volume.

For testing purposes, login to the volume via JupyterHub and delete some files.
Then stop the single user server from the JupyterHub dashboard.

Configure and launch the restoring operation:

    kubectl -n jhub create -f stash_restore.yaml

This overwrites the content of the target volume with the content of the backup. See the Stash documentation on how to restore to a different volume.

```
> kubectl -n jhub get restoresession
NAME      REPOSITORY   PHASE       AGE
restore   osn-repo     Succeeded   2m18s
```

Then login back to JupyterHub and check that the files previously deleted.

In the default configuration `stash_restore.yaml` restores the last backup, independently of username, so if you are backing up volumes of different users, you should tag by usernames, see below, and then restore a specific id (just replace `latest` in the YAML file with the first 10 or so characters of the ID).
See an example of the full restore workflow with screenshots at the end of [this Github issue](https://github.com/det-lab/jupyterhub-deploy-kubernetes-jetstream/issues/44).

## Setup for production in a small deployment

In a small deployment with tens of users, we can individually identify which users we want to backup, and choose a schedule.
The backup service works even the user is currently logged in, anyway, it is good practice to schedule a daily backup at 3am or 4am in the appropriate timezone.
We should create 1 `BackupConfiguration` object for each user, 10 minutes apart, each targeting a different PersistentVolumeClaim.

## Template backup configuration creation

If you like danger, you can also automate the creation of the `BackupConfiguration` objects.
You can create a text file named `users_to_backup.txt` with 1 username per line of the JupyterHub users you want to backup.

Then customize the `stash_backupconfiguration_template.yaml` configuration file, make sure you decide a retention policy, for more information see the Stash or Restic documentation.
Unfortunately Stash considers all backups together under 1 retention policy, so if I set to keep 1 weekly backup, it will retain 1 weekly backup of just **one of the users** instead of all of them.
I worked around this issue tagging myself the backups after the fact using the `restic` command line tool, see the next section.

Then you can launch it:

    bash setup_backups.sh

```
******** Setup xxxxxxx at 8:0
backupconfiguration.stash.appscode.com/backup-xxxxxxx created
******** Setup xxxxxxx at 8:10
backupconfiguration.stash.appscode.com/backup-xxxxxxx created
******** Setup xxxxxxx at 8:20
backupconfiguration.stash.appscode.com/backup-xxxxxxx created
******** Setup xxxxxxx at 8:30
backupconfiguration.stash.appscode.com/backup-xxxxxxx created
******** Setup xxxxxxx at 8:40
backupconfiguration.stash.appscode.com/backup-xxxxxxx created
```

There is no chance this will work the first time, so:

    kubectl delete backupconfiguration --all

## Categorize the backups by username

Unfortunately I couldn't find a way to tag the backups with the username which own the volume.
So I added this line:

    echo $JUPYTERHUB_USER > ~/.username;

to the `zero-to-jupyterhub` configuration YAML under:

```
singleuser:
  lifecycleHooks:
    postStart:
      exec:
        command:
```

So when the user logs in, we write their username into the volume.
Then we can use `restic` outside of Kubernetes to tag the backups once in a while with the correct usernames,
see the `restic_tag_usernames.sh` script.

Once we have tags, we can handle pruning old backups manually using the `restic forget` command.

## Manage backups outside of Kubernetes

Stash manages backups with `restic`.
It is also possible to access and manage the backups using `restic` on a machine outside of Kubernetes.

Install `restic` [from the official website](https://restic.readthedocs.io/en/stable/020_installation.html#stable-releases)

Export the AWS variables:

```
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
```

Have the RESTIC password ready for the prompt:

```
restic -r s3:https://ncsa.oss-data/jetstream-backup/ snapshots
enter password for repository: 
repository 18a1c421 opened successfully, password is correct
created new cache in /home/zonca/.cache/restic
ID        Time                 Host        Tags        Paths
------------------------------------------------------------------
026bcce3  2021-05-10 13:17:17  host-0                  /stash-data
4f71a384  2021-05-10 13:18:16  host-0                  /stash-data
34ff4677  2021-05-10 13:19:18  host-0                  /stash-data
9f7337fe  2021-05-10 13:20:08  host-0                  /stash-data
c130e039  2021-05-10 13:21:08  host-0                  /stash-data
------------------------------------------------------------------
5 snapshots
```

You can even browse the backups without downloading the data:

    sudo mkdir /mnt/temp
    sudo chown $USER /mnt/temp
    restic -r s3:https://ncsa.osn.xsede.org/xxxxxx/jetstream-backup/ mount /mnt/temp

```
/mnt/temp/snapshots/latest/stash-data $ ls
a  b  Healpix_3.70_2020Jul23.tar.gz  MosfireDRP-2018release.zip  plot_cl_TT.ipynb  Untitled1.ipynb  Untitled2.ipynb  Untitled.ipynb
```

## Troubleshooting

* Issue: Volume available but also attached in Openstack, works fine on JupyterHub but backing up fails, this can happen while testing.
* Solution: Delete the PVC, the PV and the volume via Openstack, login through JupyterHub to get another volume assigned.

* Issue: Volumes cannot be mounted because they are in "Reserved" state in Openstack
* Solution: Run `openstack volume set --state available <uuid>`, this is an [open issue affecting Jetstream](https://github.com/zonca/jupyterhub-deploy-kubernetes-jetstream/issues/40)

## Setup monitoring

See the new tutorial on [how to setup a system to monitor that the backups are being executed](https://zonca.dev/2022/04/monitor-restic-backups-kubernetes.html)
