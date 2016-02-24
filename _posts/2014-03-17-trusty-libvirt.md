---
layout: post
title: Boot Ubuntu Trusty Tahr 14.04 with libvirt
categories: 
---

# {{ page.title }}

Then next Ubuntu long term service (LTS) operating system, [Trusty Tahr or version 14.04](https://wiki.ubuntu.com/TrustyTahr/ReleaseSchedule), is currently expected to be released on April 27th of 2014, about a month away from the writing of this post. So note that right now it is still beta.

Below I'll take a quick look at how to deploy the cloud image prepared by Ubuntu with libvirt. We'll use libvirt + kvm on Ubuntu Precise to boot the Trusty Tahr image. The server running libvirt is Ubuntu 12.04, and is setup in a default manner, which means vms should IPs from dnsmasq and get natted access to the Internet. 

## Download the image

First we download the cloud image from Ubuntu's site. Thankfully Ubuntu provides ready-to-go images.

<pre>
<code>root# cd /var/lib/libvirt/images
root# wget https://cloud-images.ubuntu.com/trusty/current/trusty-server-cloudimg-amd64-disk1.img
root# mv trusty-server-cloudimg-amd64-disk1.img trusty-server-cloudimg-amd64-disk1.img.dist
root# qemu-img convert -O qcow2 trusty-server-cloudimg-amd64-disk1.img.dist \
trusty-server-cloudimg-amd64-disk1.img
root# qemu-img resize trusty-server-cloudimg-amd64-disk1.img +8G
</code>
</pre>

Above I downloaded the image, converted it from compressed to uncompressed, and then increase it's size by 8G, making it about a 10G image. If it's not resized it will be about 2G.

Next create a snapshotted backing file which will allow us to keep the original image pristine.

<pre>
<code>root# qemu-img create -f qcow2 -b trusty-server-cloudimg-amd64-disk1.img trusty1.img
</code>
</pre>

That's all it takes to grab a pre-built Ubuntu Trusty Tahr image!

## Checkout cloud-localds

We need to get cloud-utils from launchpad which includes a cloud-localds script. Using this script we can easily create an ISO file that can be used locally by the vm with cloud-init to configure the vm.

<pre>
<code>root# cd ~
root# bzr branch lp:cloud-utils
root# ls cloud-utils/bin/cloud-localds 
cloud-utils/bin/cloud-localds
</code>
</pre>

Using that script we can create a user-data image that can be attached to our virtual machine.

## Create a user-data file

Essentially what this allows us to do is have a local disk file attached to the vm which [cloud-init](http://cloudinit.readthedocs.org/en/latest/) can use to setup the system as we would like it, at least by setting a password and/or an ssh key. 

I'm not showing my public key below just because it doesn't fit well on the page. But just paste your public ssh key in after the "-". The password for the ubuntu user is set to "passw0rd" which is in fact not a good password. It might be better to either not set the password at all and just use and ssh key, or to set a very good password. This is just an example.

<pre>
<code>#cloud-config
password: passw0rd
chpasswd: { expire: False }
ssh_pwauth: True
ssh_authorized_keys:
  - <enter your public key>
</code>
</pre>

Cloud-init has a ton more features and options so I suggest checking out the documentation. That said, it is a rather new and fast moving project.

## Build the user-data image

Now we convert the user-data file to an ISO file.

<pre>
<code>root# ~/cloud-utils/bin/cloud-localds user-data.img user-data
root# cp user-data.img /var/lib/libvirt/images
</code>
</pre>

With that file created, we can now setup a libvirt xml file to boot the virtual machine.

## Prepare a libvirt.xml file

Here is an example libvirt xml file, in this case called trusty1.xml.

<pre>
<code>root# cat trusty1.xml 
<domain type='kvm'>
    <name>trusty1</name>
    <memory>1048576</memory>
    <os>
        <type>hvm</type>
        <boot dev="hd" />
    </os>
    <features>
        <acpi/>
    </features>
    <vcpu>1</vcpu>
    <devices>
        <disk type='file' device='disk'>
            <driver type='qcow2' cache='none'/>
            <source file='/var/lib/libvirt/images/trusty1.img'/>
            <target dev='vda' bus='virtio'/>
        </disk>
        <disk type='file' device='disk'>
            <source file='/var/lib/libvirt/images/user-data.img'/>
            <target dev='vdb' bus='virtio'/>
        </disk>
        <interface type='network'>
            <source network='default'/>
                <model type='virtio'/>
        </interface>
    </devices>
</domain>
</code>
</pre>

Now that we have a libvirt xml file we can define and start the vm.

## Define and start the virtual machine

Lets define and start the vm based on the images we created and the libvirt xml file.

<pre>
<code>root# virsh define trusty1.xml
Domain trusty1 defined from trusty1.xml
root# virsh start trusty1
</code>
</pre>

Now that the vm is booted, by default it should get an IP from dnsmasq, and has a random mac address. Figuring out the IP is not that easy with libvirt (if you know a way, let me know) but generally I look at the leases file or would set specific mac addresses to get specific IPs.

<pre>
<code>root# cat /var/lib/libvirt/dnsmasq/default.leases 
1395108553 52:54:00:6a:f1:c8 192.168.122.103 cloudimg *
</code>
</pre>

From dnsmasq know that the instance has the 192.168.122.103 IP, we can ssh in. Note that because I put my ssh public key in I can ssh into the server without a password. I removed some content for brevity.

<pre>
<code>root# ssh ubuntu@192.168.122.103
Warning: Permanently added '192.168.122.103' (ECDSA) to the list of known hosts.
Welcome to Ubuntu Trusty Tahr (development branch) (GNU/Linux 3.13.0-14-generic x86_64)
SNIP!
Last login: Sun Mar 16 18:46:59 2014 from 192.168.122.1
ubuntu@cloudimg:~$ cat /etc/lsb-release 
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=14.04
DISTRIB_CODENAME=trusty
DISTRIB_DESCRIPTION="Ubuntu Trusty Tahr (development branch)"
ubuntu@cloudimg:~$ df -h | grep vda1
/dev/vda1        11G  898M  8.7G  10% /
</code>
</pre>

That is about the minimum required to get a Trusty image up and running via libvirt. Certainly there are other considerations one might have to make in a production environment, such as passwords and keys, actual production Trusty Tahr image, etc. 

Now to explore the new features of Trusty Tahr! 

Don't forget it won't be officially released until late April of 2014.
