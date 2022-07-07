# Suggestions to improve the offcial docummentation

[Upgrading RHEL for Edge systems](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/composing_installing_and_managing_rhel_for_edge_images/index#upgrading_rhel_for_edge_systems)

## 10.3.1. Upgrading your RHEL 8 system to RHEL 9

In the prerequisistes:

**You have a RHEL 9 OSTree repository running Podman container** What means that?

We should include a way to handle RHEL lifecycle updates using the httpd repository having different branches, ...

In the procedure instead of **Adjust the /etc/ostree/remotes.d/*.repo to point to the httpd server.** the new repo should be added using the specific commands:

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
[root@upgrade ~]#
```

and the old ostree repo can be deleted if the upgrade was successful:

```
[root@upgrade ~]# ostree remote delete edge
```