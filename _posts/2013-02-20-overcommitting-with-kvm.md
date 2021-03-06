---
layout: post
title: Over committing with KVM
categories:
---

# {{ page.title }}

It's quite possible to over commit resources with the KVM hypervisor.

I should say first that most of the work I've been doing around over committing in KVM is based on a project I am working in where the virtual machines are stateless. This makes over committing easier because if we run out of resources on a compute node and this causes a vm to crash, it's not the end of the world--the end-user can just restart their session.

To me over committing means having virtual machines running on a node where the total CPU, main memory (ie. ram), and disk that is available to the virtual machines is much larger than the actual physical resources available on the node. Hopefully five or ten times larger.

## Hardware

I am running these tests on a single Dell C6220 node. It has 32 cores (including hyperthreading), 128GB of main memory, and SSD drives.

<pre>
<code>test_host:~$ cat /proc/cpuinfo | grep processor | wc -l
32
</code>
</pre>

The SSD drive types we are testing with are:

- 2x Standard Dell 300GB SSD
- 2x Samsung 830 512GB
- 2x Intel 520 480GB

Each of them is in a stripe/RAID0 configuration, so we have three striped devices, md2 (Samsung) and the poorly named md126 (Intel) and md127 (Dell).

<pre>
<code>test_host:~$ cat /proc/mdstat 
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4]
[raid10] 
md0 : active raid1 sdb1[1] sda1[0]
      307136 blocks [2/2] [UU]
      
md2 : active raid0 sdb5[1] sda5[0]
      882155520 blocks super 1.2 512k chunks
      
md1 : active raid1 sdb2[1] sda2[0]
      41910144 blocks super 1.2 [2/2] [UU]
      
md126 : active raid0 sdd[1] sdc[0]
      937702400 blocks super 1.2 512k chunks
      
md127 : active raid0 sdf[1] sde[0]
      586072064 blocks super 1.2 512k chunks
      
unused devices: <none>
</code>
</pre>

The striped Samsungs are really fast--84K IOPS from the stripe!

## Memory over committing: KSM (not KVM)

KSM, or Kernel Samepage Merging, is a memory de-duplication feature that is present in most Linux systems that support KVM. I wrote a bit about it in a [previous blog post](http://serverascode.com/2012/11/11/ksm-kvm.html). Basically if you up the scanning rate it will de-duplicate more memory. This means if I'm running 300 Ubuntu Precise images there is a lot of memory that can be de-duplicated.

## CPU over committing

I don't have a lot of information on how over committing CPU works in KVM, but there are some best practices documented by a couple organizations, which I have listed below. This is an area I need to do more research in.

- [IBM says:](http://publib.boulder.ibm.com/infocenter/lnxinfo/v3r0m0/index.jsp?topic=%2Fliaat%2Fliaatbpprocmem.htm)
** Target system use at max 80%--ie. each vm shouldn't be using 100% of their CPU, and in fact should max out at 80%
** Allocate minimum VCPUs per vm--ie. if the vm doesn't need four CPUs, rather say, only one, then just give it one
- [RedHat says](https://access.redhat.com/knowledge/docs/en-US/Red_Hat_Enterprise_Linux/6/html/Virtualization_Administration_Guide/sect-Virtualization-Tips_and_tricks-Overcommitting_with_KVM.html:)
** _Virtualized CPUs are over committed best when each guest only has a single VCPU. The Linux scheduler is very efficient with this type of load. KVM should safely support guests with loads under 100% at a ratio of five VCPUs. Over committing single VCPU guests is not an issue._

Notice that RedHat is saying a 5:1 ratio for CPU over committing.

## Disk space over committing

This is straight forward: image snapshots. qcow2 images can be created as snapshots of a backing file.

<pre>
<code>test_host:/mnt/intel$ file 263.img 
263.img: QEMU QCOW Image (v2), has backing file 
(path /mnt/intel/precise-server-cloudimg-amd64-disk1.qcow2), 2147483648 bytes
</code>
</pre>

This is also how OpenStack handles images. It puts one base image on the compute node and each instance based on that same image is backed by a qcow2 snapshot. I suppose we could do the same with LVM, if desired, using snapshots.

## Can't over commit IOPS

The part that is difficult in terms of running hundreds of vms on a single node is the amount of IOPS each vm takes up. In my experiments the Ubuntu cloud image doesn't really do anything at all in terms of IOPS--maybe one or two IOPS per image when they are idle. So if I'm running 300 images, they don't even use 300 IOPS when idle. Even during boot they hardly do anything. (Of course when they are working they could be using a lot of IOPS.)

But, if I boot 100 Windows 7 images five minutes apart, I need 5000 IOPS. 

Yup: *5000 IOPS*. 

Six SATA drives in RAID10 are going to barely provide 400 IOPS let alone 5000. So, SSDs, or at least some kind of faster storage, are required. Pretty much any SSD is capable of 5000 IOPS. 

IMHO, it's a lot more efficient to run the Ubuntu cloud image than a standard Windows image.

## Load

Right now I have 300 Ubuntu images running, each being given 2048MB of memory. 

<pre>
<code>test_host:~$ ps ax | grep "kvm -drive" | wc -l
300
</code>
</pre>

And load and memory usage are just fine. A bit of swapping, but that's Ok.

<pre>
<code>top - 10:58:13 up 8 days, 23:55,  2 users,  load average: 1.35, 1.29, 1.92
Tasks: 845 total,   2 running, 843 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.8%us,  3.1%sy,  0.0%ni, 95.9%id,  0.1%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:  131997760k total, 122223236k used,  9774524k free,   142448k buffers
Swap: 33554424k total,   242172k used, 33312252k free, 42323472k cached
SNIP!
</code>
</pre>

They have been running for a few days, and are not using much in terms of IOPS as they are completely idle. Below is the output from [iostat](http://linuxcommand.org/man_pages/iostat1.html). All the vms are running off qcow2 images on the Intel 520 based stripe, md126.

<pre>
<code>avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.67    0.00    3.08    0.05    0.00   96.20

Device:            tps    MB_read/s    MB_wrtn/s    MB_read    MB_wrtn
sda              16.40         0.00         0.17          0          1
sdb              16.40         0.00         0.17          0          1
sdc              44.70         0.00         0.16          0          1
sde               0.00         0.00         0.00          0          0
sdd              44.20         0.00         0.15          0          1
sdf               0.00         0.00         0.00          0          0
md127             0.00         0.00         0.00          0          0
md126            60.10         0.00         0.31          0          3
md1              35.20         0.00         0.17          0          1
md2               0.00         0.00         0.00          0          0
md0               0.00         0.00         0.00          0          0
dm-0              0.00         0.00         0.00          0          0
nb0               0.00         0.00         0.00          0          0
</code>
</pre>

It would be great to have IOPS usage on a per-process basis, but I couldn't find a tool that would do that. The best I could do is iostat with IOPS per storage device.

## Running tests

Each vm gets its IPs from dnsmasq running on an [Openvswitch](http://openvswitch.org/) bridge (topic of a future blog post). So we have 300 tap devices.

<pre>
<code>test_host:~/performance_testing$ sudo ovs-vsctl show
SNIP!
        Port "tap166"
            Interface "tap166"
        Port "tap0"
            Interface "tap0"
    ovs_version: "1.4.0+build0"

test_host:~/performance_testing$ ifconfig | grep tap | wc -l
300
</code>
</pre>

I like using the [Ansible](http://ansible.cc) configuration management system. I've written a custom inventory script that pulls the IPs out of the dnsmasq lease file.

<pre>
<code>test_host:~/performance_testing$ ansible all -c ssh -i ./inventory.py \
-m ping -u ubuntu
SNIP 298 hosts!
192.168.100.98 | success >> {
    "changed": false, 
    "ping": "pong"
}

192.168.100.99 | success >> {
    "changed": false, 
    "ping": "pong"
}
</code>
</pre>

Then, using that inventory and Ansible I can run whatever load tests I want across however many virtual instances I want. For example I could run [fio](http://linux.die.net/man/1/fio) tests across 10% of the vms and watch the IOPS usage on the compute node.

If you have any questions, criticisms or concerns, please let me know by posting in the comments. :)
