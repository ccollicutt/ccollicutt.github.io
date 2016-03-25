---
layout: post
title: Packaged OpenVSwitch with DPDK on Ubuntu Trusty
categories:
header_image: /img/murano.png
---

_Murano by Mathieu St-Pierre_

# {{ page.title }}

This is just a quick post on getting OpenVSwitch with DPDK installed on Ubuntu 14.04/Trusty using a package instead of compiling it on your own.

## tl;dr

Install the Ubuntu Liberty Cloud Archive. Then you can install openvswitch-switch-dpdk. However I have not completed the configuration component because there are some bugs, as well I'm lacking a baremetal server at the moment as I was working in AWS. So really this post is just to say that there is a DPCK OpenVSwitch for Ubuntu Trusty. I'll follow up with more information in a later post.

## Installation

First ensure python3-software-properties is installed.

```bash
$ apt-get install python3-software-properties
```

Then install the cloud archive.

```bash
$ add-apt-repository cloud-archive:liberty
```

Now apt-get update and check the policy/version of the openvswitch-switch-dpdk package.

```bash
$ apt-cache policy openvswitch-switch-dpdk
openvswitch-switch-dpdk:
  Installed: 2.4.0-0ubuntu1~cloud0
  Candidate: 2.4.0-0ubuntu1~cloud0
  Version table:
 *** 2.4.0-0ubuntu1~cloud0 0
        500 http://ubuntu-cloud.archive.canonical.com/ubuntu/ trusty-updates/liberty/main amd64 Packages
        100 /var/lib/dpkg/status
```

Now it can be installed.

```bash
$ apt-get update
$ apt-get install openvswitch-switch-dpdk
```
<br/>

## Configuring to use DPDK

Set alternatives.

```bash
$ update-alternatives --set ovs-vswitchd /usr/lib/openvswitch-switch-dpdk/ovs-vswitchd-dpdk
update-alternatives: using /usr/lib/openvswitch-switch-dpdk/ovs-vswitchd-dpdk to provide /usr/sbin/ovs-vswitchd (ovs-vswitchd) in manual mode
```

At this point you could follow the rest of the [instructions for using OVS](http://openvswitch.org/support/dist-docs/INSTALL.DPDK.md.txt) however I will warn you that it's not that straightforward at this time, though I'm sure that will improve in the future. This [bug report](https://bugs.launchpad.net/ubuntu/+source/openvswitch-dpdk/+bug/1547463) has some hints. I expect this will be much easier soon. At any rate, at least OVS with DPDK is installed and you can start messing around with getting it working.
