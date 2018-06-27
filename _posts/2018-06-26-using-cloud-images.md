---
layout: post
title: Using Cloud Images With KVM
categories:
header_image: /img/cloud-img.jpg
header_permalink: https://unsplash.com/photos/xe-ss5Tg2mo
---

# {{ page.title }}

Most Linux distributions offer some kind of binary image that is typically used in "cloudy" situations--such as public cloud or with OpenStack. These images have been pre-built, use cloud-init, and are expecting to be configured based on metadata provided through a combination of the user and the infrastructure as a service (IaaS) system, either via network metadata or a "config drive" (a specially created ISO image that is attached to the vm).

However, one doesn't always have an IaaS system available, and sometimes we just want to use KVM and not have to attach a custom cloud init ISO image, so here are some quick instructions on using a cloud image with plain KVM, and manually configuring the image.

## Step 1: Download the image

Download from Ubuntu's cloud image cache.

```
wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
```

Other bigger distros should offer a "cloud image" in some fashion.

## Enlarge Virtual Disk

By default it's only 2.2G. That's not very many Gs. :)

```
$ qemu-img info bionic-server-cloudimg-amd64.img
image: bionic-server-cloudimg-amd64.img
file format: qcow2
virtual size: 2.2G (2361393152 bytes)
disk size: 322M
cluster_size: 65536
Format specific information:
    compat: 0.10
    refcount bits: 16
```

Resize it to 20G.

```
$ qemu-img resize bionic-server-cloudimg-amd64.img 20G
Image resized.
$ qemu-img info bionic-server-cloudimg-amd64.img
image: bionic-server-cloudimg-amd64.img
file format: qcow2
virtual size: 20G (21474836480 bytes)
disk size: 322M
cluster_size: 65536
Format specific information:
    compat: 0.10
    refcount bits: 16
```

Once we boot it we'll resize inside the OS.

## Set Root Password

virt-customize is an easy way to "edit" images.

```
$ virt-customize -a bionic-server-cloudimg-amd64.img --root-password password:coolpass
[   0.0] Examining the guest ...
[  22.3] Setting a random seed
virt-customize: warning: random seed could not be set for this type of
guest
[  22.3] Setting the machine ID in /etc/machine-id
[  22.4] Setting passwords
[  24.4] Finishing off
```

<br />

## Remove Cloud Init

Let's uninstall cloud-init as well.

(NOTE: These two virt-customize steps could be put into one command.)

```
$ virt-customize -a bionic-server-cloudimg-amd64.img --uninstall cloud-init
[   0.0] Examining the guest ...
[  12.1] Setting a random seed
virt-customize: warning: random seed could not be set for this type of
guest
[  12.1] Uninstalling packages: cloud-init
[  17.1] Finishing off
```

<br />

## Boot the VM

Create a vm definition file for virsh. Replace the various variables indicated by double braces with your choices of resources and file locations.

Note that you could use virt-manager and load the backing image and do this graphically if you like that sort of thing.

```
{% raw %}<domain type='kvm'>
  <name>{{ vm_name }}</name>
  <memory unit='KiB'>{{ vm_required_memory }}</memory>
  <currentMemory unit='KiB'>{{ vm_required_memory }}</currentMemory>
  <vcpu>{{ vm_required_vcpu }}</vcpu>
  <os>
    <type arch='x86_64'>hvm</type>
    <boot dev='hd'/>
  </os>
  <clock offset='utc'/>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <devices>
    <emulator>/usr/bin/qemu-kvm</emulator>
  <disk type='file' device='disk'>
       <driver name='qemu' type='qcow2'/>
       <source file='{{ location_of_backing_image }}'/>
       <target dev='vda' bus='virtio'/>
  </disk>
  <interface type='bridge'>
    <source bridge='virbr0'/>
    <model type='virtio'/>
  </interface>
  <serial type='pty'>
    <target port='0'/>
  </serial>
  <console type='pty'>
    <target type='serial' port='0'/>
  </console>
  </devices>
</domain>{% endraw %}
```

<br />

## Define and Start the VM

Define and start.

```
virsh define vm.xml
virsh start vm_name
```

<br />

## Access Console

Handy dandy virsh console.

```
virsh console vm_name
```

Login with username: root password: coolpass (or whatever you set it to).

## Setup DHCP (Ubuntu Bionic only example)

Networking will likely not be configured at all (b/c we removed cloud-init).

Create the below. Note the interface name. Could probably add this with virt-customize.

*NOTE: This is Ubuntu Bionic/18.04 and it has new networking configuration.*

```
root@ubuntu:/etc/netplan# cat 01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s2:
      dhcp4: true
```

Apply it.

```
root@ubuntu:/etc/netplan# sudo netplan --debug apply
** (generate:930): DEBUG: 19:24:16.431: Processing input file //etc/netplan/01-netcfg.yaml..
** (generate:930): DEBUG: 19:24:16.433: starting new processing pass
** (generate:930): DEBUG: 19:24:16.434: enp0s2: setting default backend to 1
** (generate:930): DEBUG: 19:24:16.435: Generating output files..
** (generate:930): DEBUG: 19:24:16.436: NetworkManager: definition enp0s2 is not for us (backend 1)
DEBUG:netplan generated networkd configuration exists, restarting networkd
DEBUG:no netplan generated NM configuration exists
DEBUG:device lo operstate is unknown, not replugging
DEBUG:netplan triggering .link rules for lo
DEBUG:replug enp0s2: unbinding virtio0 from /sys/bus/virtio/drivers/virtio_net
DEBUG:replug enp0s2: rebinding virtio0 to /sys/bus/virtio/drivers/virtio_net
```

Now interface should have an IP. (Assuming the hypervisor was setup with a default network that does DHCP.)

## Disk size

Note that cloud-init usually does this. But b/c we don't have it, we'll perform the same basic steps manually.

Growpart.

```
root@ubuntu:/etc/netplan# growpart /dev/vda 1
CHANGED: partition=1 start=227328 old: size=4384735 end=4612063 new: size=41715679,end=41943007
```

Resize.

```
root@ubuntu:/etc/netplan# resize2fs /dev/vda1
resize2fs 1.44.1 (24-Mar-2018)
Filesystem at /dev/vda1 is mounted on /; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 3
The filesystem on /dev/vda1 is now 5214459 (4k) blocks long.

root@ubuntu:/etc/netplan# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            985M     0  985M   0% /dev
tmpfs           200M  460K  199M   1% /run
/dev/vda1        20G  959M   19G   5% /
tmpfs           997M     0  997M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           997M     0  997M   0% /sys/fs/cgroup
/dev/vda15      105M  3.4M  102M   4% /boot/efi
tmpfs           200M     0  200M   0% /run/user/0
```

Should have a 20G / now.

## Conclusion

One should really use cloud-init properly, but if you are stuck or just plain don't feel like it, then this is one way to go about getting a vm setup in KVM using a cloud image.
