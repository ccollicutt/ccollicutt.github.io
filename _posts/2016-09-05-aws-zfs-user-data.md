---
layout: post
title: Installing ZFS in an AWS EC2 Instance Using User-Data
categories:
header_image: /img/syd_mead.jpg
header_permalink: https://twitter.com/70sscifiart/status/770925365124530176 
---

# {{ page.title }}

Quick post on installing ZFS into an AWS instance using user-data and cloud-init.

I'm doing some work with Kubernetes, which uses Docker, and Docker can use ZFS as a backing store. I want ZFS configured before I do anything else, and the best way to do that is to either create a specific Amazon machine image (AMI) or to use user-data. I chose the former.

Note that this is an instance-store AMI and has two block devices, xvdb and xvdc.

~~~bash
#cloud-config
 
packages:
 - zfsutils-linux

runcmd:
 - umount -f /mnt
 - zpool create -f zpool-docker /dev/xvdb /dev/xvdc
 - zfs create -o mountpoint=/var/lib/docker zpool-docker/docker
~~~

That's it.

Once the instance is booted up, you can login and see that there is a /var/lib/docker file system mounted from the zpool-docker zpool.

~~~bash
root@docker0:/home/ubuntu# zfs list /var/lib/docker
NAME                  USED  AVAIL  REFER  MOUNTPOINT
zpool-docker/docker   265M  29.0G  1.59M  /var/lib/docker
~~~
