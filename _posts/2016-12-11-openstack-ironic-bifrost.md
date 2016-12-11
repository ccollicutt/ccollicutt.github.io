---
layout: post
title: OpenStack Bifrost
categories:
header_image: /img/bifrost.jpg
---

# {{ page.title }}

No matter how you deploy OpenStack, you need some kind of software that manages the operating system that runs directly on the baremetal. Sure, maybe you have all kinds of exotic container management systems, but one of the problems with running your own private cloud is that you still need to manage the physical hosts and their base operating system. Those containers have to run somewhere!

So most OpenStack deployment systems come with some kind of baremetal installer. For example Mirantis uses Cobber and Tripleo uses Ironic (within the concept of an undercloud). 

There are other baremetal installers too, such as Ubuntu MaaS, I think Foreman also, and if you feel like it you can even roll your own based on PXE booting with dnsmasq or similar. The point is you want to automatically deploy the baremetal OS (BOS).

## Bifrost

I already mentioned OpenStack Ironic as a way to manage the BOS. [Bifrost](http://docs.openstack.org/developer/bifrost/) also uses Ironic:

>Bifrost (pronounced bye-frost) is a set of Ansible playbooks that automates the task of deploying a base image onto a set of known hardware using ironic. It provides modular utility for one-off operating system deployment with as few operational requirements as reasonably possible.

Basically it installs a standalone Ironic system, and then also uses Ansible playbooks to generate OS images and deploys them to baremetal nodes. It is a combination of Ironic and Disk Image Builder with Ansible to install and orchestrate them.

## Installation

[Installation](http://docs.openstack.org/developer/bifrost/readme.html#installation) is straight forward. I would repeat the docs here, but they are quite good in terms of installation. The only thing to really remember is to set the *network_interface* variable to the correct interface on the bifrost node. You will need at least one server to run this from. I'm using a virtual machine. One of the interfaces on the bifrost node must be on the same network as the DHCP interfaces of the physical nodes you want to manage with bifrost. Like most BOS installers, Ironic installs via PXE booting.

Once the install completes you should be able to run Ironic commands.

~~~bash
ubuntu@p3-bifrost-01:~$ ironic node-list
+--------------------------------------+----------+---------------+-------------+--------------------+-------------+
| UUID                                 | Name     | Instance UUID | Power State | Provisioning State | Maintenance |
+--------------------------------------+----------+---------------+-------------+--------------------+-------------+
+--------------------------------------+----------+---------------+-------------+--------------------+-------------+
~~~

But of course you won't have any Ironic nodes because you have not enrolled servers into Ironic.

## Disk Image

The installation process also creates a disk image that will be deployed to the physical node.

This image is in */httpboot*.

~~~bash
ubuntu@p3-bifrost-01:~$ ls /httpboot/*.qcow2
/httpboot/deployment_image.qcow2
~~~

By default the image is named *deployment_image.qcow2*.

If you don't change any defaults it will be Debian Jessie.

You can change what OS is build using a couple of options. For example if you wanted Ubuntu Trusty:

~~~bash
dib_os_release: "trusty"
dib_os_element: "ubuntu"
~~~

Note that I had some issues with Ubuntu Xenial and Bifrost which I will discuss later in the post.

But just be aware that part of the installation process also creates this image, and the image is obviously very important as it is what gets deployed to the physical node. Likely you will want to customize that image.

## Environment

I've added sourcing a couple of bifrost files to the .profile of the local user:

~~~bash
# Bifrost
. ~/bifrost/env-vars
. /opt/stack/ansible/hacking/env-setup
~~~

Just makes it a bit easier to use Bifrost.

## Hardware Enrollment

Again, the docs are good on this.

Basically you need to setup a file that details your inventory of physical servers. The inventory can be defined in the old CSV manner, or in a newer method with either JSON or YAML.

Here's an example of a single server in YAML. Obviously your real inventory should include tens or hundreds of physical servers.

~~~bash
---
  node01:
    uuid: "00000000-0000-0000-0000-000000000001"
    driver_info:
      power:
        ipmi_username: "PXE_USER"
        ipmi_address: "10.10.0.10"
        ipmi_password: "PXE_PASSWORD"
    nics:
      -
        mac: "48-8C-83-E7-A5-D5"
    driver: "agent_ipmitool"
    properties:
      cpu_arch: "x86_64"
      ram: "128375"
      disk_size: "400"
      cpus: "48"
    name: "node01"
~~~

When you start a deployment, the dnsmasq server running on the node will wait to see DHCP requests from the above mac address, and when it receives them proceed to install a specific OS image to that physical node.

## Deployment

Deployment of nodes is a single Ansible command. Again the docs are good on this.

Here's an example deployment of a single node. I've put it into a deploy script just to make it a bit easier to use. :)

~~~bash
ubuntu@p3-bifrost-01:~$ cat deploy.sh 
#!/bin/bash

export BIFROST_INVENTORY_SOURCE=~/node01.yml

pushd ~/bifrost/playbooks
  ansible-playbook -vvvv -i inventory/bifrost_inventory.py deploy-dynamic.yaml
popd 
~~~

Once the deployment completes, you should have some nodes listed as active in Ironic.

~~~bash
ubuntu@p3-bifrost-01:~$ ironic node-list
+--------------------------------------+----------+---------------+-------------+--------------------+-------------+
| UUID                                 | Name     | Instance UUID | Power State | Provisioning State | Maintenance |
+--------------------------------------+----------+---------------+-------------+--------------------+-------------+
| 00000000-0000-0000-0000-000000000001 | node01 | None          | power on    | active             | False       |
+--------------------------------------+----------+---------------+-------------+--------------------+-------------+
~~~

Hopefully in your deployment you will have more than one node. :)

## What IP address did the node get?

You can specify what IP address the node will get on the DHCP/PXE network as a static entry in the nodes inventory information, but I have not done that. Thus it will get a random IP from the DHCP pool.

~~~bash
ubuntu@p3-bifrost-01:~$ cat /var/lib/misc/dnsmasq.leases 
1481507199 48-8C-83-E7-A5-D5 10.100.0.31 node01 01:48-8C-83-E7-A5-D5
~~~

I can ssh into that node:

~~~bash
ubuntu@p3-bifrost-01:~$ ssh 10.100.0.31
Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-53-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


Last login: Sun Dec 11 13:54:29 2016 from 10.100.0.3
ubuntu@node01:~$ 
~~~

Bwoop!

## Ubuntu Xenial and building your own images

Bifrost will try to help out and generate a working image for you. This is nice. However, it did not work with Ubuntu Xenial. I ran into some kind of Python related error which I will investigate further. I believe it's related to the default use of simple-init instead of cloud-init, but I'm not completely sure at this time.

But, you an also generate your own image using [Disk Image Builder](http://docs.openstack.org/developer/diskimage-builder/) (dib).

Dib is very easy to install. Using it takes a bit more work as there are many options.

I am building a working Xenial image like so:

~~~bash
ubuntu@p3-bifrost-01:~/dib$ cat build.sh 
#!/bin/bash

cd /home/ubuntu/dib
export PATH=$PATH:/home/ubuntu/dib/diskimage-builder/bin:/home/ubuntu/dib/dib-utils/bin
export DIB_RELEASE=xenial
export DIB_DEV_USER_PASSWORD="SOME_USER"
export DIB_DEV_USER_USERNAME="SOME_PASS"
export DIB_DEV_USER_PWDLESS_SUDO="Yes"
export DIB_DEV_USER_PASSWORD="/bin/bash"
export DIB_CLOUD_INIT_DATASOURCES="ConfigDrive"

# dib elements to use in creating the image
EL="ubuntu vm cloud-init devuser serial-console cloud-init-datasources"

disk-image-create -a amd64 -t qcow2 -o xenial.qcow2 \
-p python2.7,python-simplejson \
$EL
~~~

This creates a Xenial image that has cloud-init installed but set to only use ConfigDrive as a data source. The bifrost installation does not have a metadata service that the instance can call back to, and instead uses ConfigDrive to add things to the instance dynamically, such as ssh keys and the like.

Then I just copy the resulting image to */httpboot/deployment_image.qcow2* and that is what will be deployed to the physical node.

I should note that:

* serial-console does not seem to work in Xenial
* Adding a devuser in production is probably not a great idea

I also add python2.7 and python-simplejson for use with Ansible.

## ConfigDrive

The drive is attached to the physical node and has a json file that defines some dynamic changes to the host. Below it is just hostname and an ssh key for the ubuntu user's authorized_keys file.

~~~bash
ubuntu@node01:~$ sudo mount /dev/sda2 /mnt
mount: /dev/sda2 is write-protected, mounting read-only
ubuntu@node01:~$ find /mnt
/mnt
/mnt/openstack
/mnt/openstack/2012-08-10
/mnt/openstack/2012-08-10/meta_data.json
/mnt/openstack/content
/mnt/openstack/latest
/mnt/openstack/latest/meta_data.json
ubuntu@node01:~$ cat /mnt/openstack/latest/meta_data.json 
{
    "availability_zone": "",
    "files": [],
    "hostname": "node01",
    "name": "node01",
    "meta": {},
    "public_keys": {
        "mykey": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC0tdpS1S83ZQPMzFVwJ603CiKyapIOSfjmofqlYExYm+UCcFuC1eUF+xYA/dwFucKbd6shdLxC/cSLtHilQjolyg32jKw8G0LwittPH7Npi1BSCmLg5xnwUML6Hh/g/r3Xjj0NfuqIMBiiwQR/XkCyWHt5tE8ztGCm14Mz4SSL8qFhPdZXF0r5O9iBsWCJ6OsabuPK3lZQUqeMnKynARocnqXa4KHUr1yEOM/VHMNnUuJbRPEJoFHPrqS+vHOwKPWBvfv8Eia6GxsCXt3Z+ioWYA4Ejed/Csv1kRAWLDA4xuExD4VjgHdpHPoPt1M3nv3BdhwJpDzhSrXumGdFZz79 ubuntu@p3-bifrost-01"
    },
    "uuid": "00000000-0000-0000-0000-000000000001"
}
~~~

If you have not used OpenStack or any IaaS that uses metadata it can be a bit confusing. Suffice it to say that in image based deployment systems there is often some dynamic configuration you need to make, at least inject ssh keys, and this is done with a combination of information from some datasource, and cloud-init in the image.

I should note that Bifrost generates this ConfigDrive automatically.

## Conclusion

This is a basic overview of using Bifrost. I have quite a bit more work to do in terms of understanding and customizing Bifrost, but in a few hours I had a basic system going that works well. If you are using Ubuntu Trusty you will probably have a bit of an easier time as opposed to Xenial, but I expect that will either improve or I will discover that I was doing something silly. :)

The ability to automatically deploy a custom image to many baremetal servers is a basic requirement for a successful deployment of OpenStack or any other large, complicated system.
