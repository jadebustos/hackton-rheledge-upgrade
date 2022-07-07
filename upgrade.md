# Upgrade

## Initial scenario

A RHEL 8.6 (Edge) image has been created with the following packages:

* **podman**
* **slirp4netns**
* **net-tools**
* **setroubleshoot-server**

The image has been deployed using a [kickstartfile](rhel8edge.ks) which deploys a [containerized application](quay.io/rhte_2019/2048-demoday:latest)

A RHEL 8 server was used to create the image.

After RHEL8 server boots, ssh to it using **core** user and **edge** password.

The pre-pull systemd unit is not working properly but hackaton purpouse is not to fix that.

## RHEL 9 image

To upgrade the above a RHEL 9 image has been built with the same packages.

 A RHEL 9 server was used to create the image.

## Apache Server

An apache server has been used to serve:

* Kickstart file
* RHEL 8 image
* RHEL 9 image

## Upgrade process

[Upgrading RHEL for Edge systems](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/composing_installing_and_managing_rhel_for_edge_images/index#upgrading_rhel_for_edge_systems)

Show current configuration:

```
[root@upgrade ~]# cat /etc/ostree/remotes.d/edge.conf 
[remote "edge"]
url=http://192.168.122.1/ostree/repo/
gpg-verify=false
[root@upgrade ~]# ostree remote show-url edge
http://192.168.122.1/ostree/repo/
[root@upgrade ~]# ostree remote refs edge
error: Remote refs not available; server has no summary file
[root@upgrade ~]# 
```

Add new repo:

```
[root@upgrade ~]# ostree remote add --set=url=http://192.168.122.1/ostree-rhel9/repo/ --set=gpg-verify=false rhel9 http://192.168.122.1/ostree-rhel9/repo/
[root@upgrade ~]# cat /etc/ostree/remotes.d/edge.conf 
[remote "edge"]
url=http://192.168.122.1/ostree/repo/
gpg-verify=false
[root@upgrade ~]# ostree remote show-url rhel9
http://192.168.122.1/ostree-rhel9/repo/
[root@upgrade ~]# ostree remote list
edge
rhel9
```

```
[root@upgrade ~]# rpm-ostree status
State: idle
AutomaticUpdates: stage; rpm-ostreed-automatic.timer: no runs since boot
Deployments:
● edge:rhel/8/x86_64/edge
                   Version: 8.6 (2022-07-07T08:54:42Z)
                    Commit: 94062cfbbd3af0c586738ab71507057cbadc7fbf6ffd1512b22b85713627fcba
[root@upgrade ~]# rpm-ostree rebase rhel9:rhel/9/x86_64/edge
  util-linux-core-2.37.4-3.el9.x86_64
  wireless-regdb-2020.11.20-6.el9.noarch
  yajl-2.1.0-20.el9.x86_64
Changes queued for next boot. Run "systemctl reboot" to start a reboot
[root@upgrade ~]# Connection to 192.168.122.124 closed by remote host.
Connection to 192.168.122.124 closed.
[jadebustos@archimedes ~]$ 
```

Where:

* **rhel9** is the repo name.
* **rhel/9/x86_64/edge** is the reference that can be found on **compose.json** file within the ostree repository.

When the server boots:

Check if the upgrade was successful and delete the old repository:

```
[root@upgrade ~]# ostree remote delete edge
```

Check the current ostree version:

```
[core@upgrade ~]$ rpm-ostree status
State: idle
AutomaticUpdates: stage; rpm-ostreed-automatic.timer: no runs since boot
Deployments:
● edge:rhel/8/x86_64/edge
        OstreeRemoteStatus: Remote "edge" not found
                   Version: 8.6 (2022-07-07T08:54:42Z)
                    Commit: 94062cfbbd3af0c586738ab71507057cbadc7fbf6ffd1512b22b85713627fcba

  rhel9:rhel/9/x86_64/edge
                   Version: 9.0 (2022-07-07T09:34:18Z)
                    Commit: b56cc17c4b524d1a9b1ea6cf870bf457dcb86e0e47f25656b770dcf3061aa2b0
[core@upgrade ~]$ 
```

