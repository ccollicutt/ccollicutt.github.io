---
layout: post
title: Installing IBM high-iops FusionIO Cards in Redhat/Centos 6
categories:
---

# {{ page.title }}

In a previous [post](http://serverascode.com/serverascode/storage/2011/06/27/fusionio-drives-on-redhat-enterprise-5.html) I had described how I deployed a IBM branded FusionIO drive on Redhat Enterprise 5.

I am now running that same card on CentOS 6, and am using the new version (2.2.3) of [IBM's version of the driver](http://www-947.ibm.com/support/entry/portal/docdisplay?lndocid=MIGR-5085137).

Actually I think there is a [new-new](http://www.mysqlperformanceblog.com/2012/05/07/testing-fusion-io-iodrive-now-with-driver-3-1/) version (3) of the driver now out for some people. I'm not sure if IBM has put out this driver or not for their high-iops cards.


CentOS version:

<pre>
<code>[root@srv ~]# cat /etc/redhat-release 
CentOS release 6.2 (Final)
</code>
</pre>


The kernel I am running is stock RHEL 6:

<pre>
<code>[root@srv ~]# uname -a
Linux example.com 2.6.32-220.17.1.el6.x86_64 #1 \
SMP Wed May 16 00:01:37 BST 2012 x86_64 x86_64 x86_64 GNU/Linux
</code>
</pre>

This is what I see in terms of PCI devices for the FusionIO cards:

<pre>
<code>[root@srv ~]# lspci | grep -i fusion
8f:00.0 Mass storage controller: Fusion-io ioDimm3 (rev 01)
90:00.0 Mass storage controller: Fusion-io ioDimm3 (rev 01)
</code>
</pre>

So the card is physically installed in the server, but the driver has not been loaded, so they are not usable at this point. Also should note that one 640GB cards actually looks like 2x 320GB devices to the OS.

First, we download the zip file containing the RPMs from IBM.

%{color:red}Warning:% These drivers are for the IBM version of the FusionIO cards. If you are not running the IBM version you probably need different drivers and RPMs.

<pre>
<code># wget ftp://download2.boulder.ibm.com/ecc/sar/CMA/XSA/ibm_dd_highiop_ssd-2.2.3_rhel6_x86-64.zip
SNIP!
</code>
</pre>

Inside that zip are several RPMs:

<pre>
<code>[root@srv tmp]# mkdir fio
[root@srv tmp]# cd fio/
[root@srv fio]# unzip ../ibm_dd_highiop_ssd-2.2.3_rhel6_x86-64.zip 
Archive:  ../ibm_dd_highiop_ssd-2.2.3_rhel6_x86-64.zip
  inflating: rhel6/fio-common-2.2.3.66-1.0.el6.x86_64.rpm  
  inflating: rhel6/fio-firmware-highiops-101583.6-1.0.noarch.rpm  
  inflating: rhel6/fio-snmp-agentx-1.1.1.5-1.0.el6.x86_64.rpm  
  inflating: rhel6/fio-sysvinit-2.2.3.66-1.0.el6.x86_64.rpm  
  inflating: rhel6/fio-util-2.2.3.66-1.0.el6.x86_64.rpm  
  inflating: rhel6/high_iops-gui-2.3.1.1874-1.1.noarch.rpm  
  inflating: rhel6/iomemory-vsl-2.2.3.66-1.0.el6.el6.src.rpm  
  inflating: rhel6/iomemory-vsl-2.6.32-71.el6.x86_64-2.2.3.66-1.0.el6.el6.x86_64.rpm  
  inflating: rhel6/libfio-2.2.3.66-1.0.el6.x86_64.rpm  
  inflating: rhel6/libfusionjni-1.1.1.5-1.0.el6.x86_64.rpm 
</code>
</pre>

So far when I've been running these servers I haven't installed all of those RPMs, only a subset.

So lets install those RPMs:

<pre>
<code>[root@srv rhel6]# yum localinstall --nogpg \
 fio-common-2.2.3.66-1.0.el6.x86_64.rpm \
 libfio-2.2.3.66-1.0.el6.x86_64.rpm fio-util-2.2.3.66-1.0.el6.x86_64.rpm \
 fio-sysvinit-2.2.3.66-1.0.el6.x86_64.rpm \
 fio-firmware-highiops-101583.6-1.0.noarch.rpm \
 iomemory-vsl-2.6.32-71.el6.x86_64-2.2.3.66-1.0.el6.el6.x86_64.rpm
SNIP!
Transaction Test Succeeded
Running Transaction
  Installing     : fio-util-2.2.3.66-1.0.el6.x86_64                   1/6 
  Installing     : fio-common-2.2.3.66-1.0.el6.x86_64                 2/6 
  Installing     : iomemory-vsl-2.6.32-71.el6.x86_64-2.2.3.66-1.0.e   3/6 
  Installing     : libfio-2.2.3.66-1.0.el6.x86_64                     4/6 
  Installing     : fio-sysvinit-2.2.3.66-1.0.el6.x86_64               5/6 
  Installing     : fio-firmware-highiops-101583.6-1.0.noarch          6/6 

Installed:
  fio-common.x86_64 0:2.2.3.66-1.0.el6                                    
  fio-firmware-highiops.noarch 0:101583.6-1.0                             
  fio-sysvinit.x86_64 0:2.2.3.66-1.0.el6                                  
  fio-util.x86_64 0:2.2.3.66-1.0.el6                                      
  iomemory-vsl-2.6.32-71.el6.x86_64.x86_64 0:2.2.3.66-1.0.el6.el6         
  libfio.x86_64 0:2.2.3.66-1.0.el6                                        

</code>
</pre>

As you can see the sysvinit RPM contains a couple of init.d files.

<pre>
<code>[root@srv rhel6]# rpm -qpl fio-sysvinit-2.2.3.66-1.0.el6.x86_64.rpm 
/etc/init.d/iomemory-vsl
/etc/sysconfig/iomemory-vsl
</code>
</pre>

Let's _chkconfig_ this on permanently.

<pre>
<code>[root@srv rhel6]# chkconfig iomemory-vsl on
</code>
</pre>

We *also* need to enable _iomemory-vsl_ in _/etc/sysconfig/iomemory-vsl_.

<pre>
<code>[root@srv init.d]# cd /etc/sysconfig
[root@srv sysconfig]# grep ENABLED iomemory-vsl 
# If ENABLED is not set (non-zero) then iomemory-vsl init script will not be
#ENABLED=1
[root@srv sysconfig]# vi iomemory-vsl 
[root@srv sysconfig]# grep ENABLED iomemory-vsl 
# If ENABLED is not set (non-zero) then iomemory-vsl init script will not be
ENABLED=1
[root@srv sysconfig]#
</code>
</pre>

And we can start or restart iomemory-vsl:

<pre>
<code>[root@srv sysconfig]# service iomemory-vsl restart
Stopping iomemory-vsl: 
Unloading module iomemory-vsl
                                                           [FAILED]
Starting iomemory-vsl: 
Loading module iomemory-vsl
Attaching: [                    ] (  0%) /Attaching:
[                    
Attaching: [====================] (100%) \
fioa
Attaching: [====================] (100%)
fiob
                                                           [  OK  ]
</code>
</pre>

At this point I'm going to *reboot* the server as well, just to make sure everything is going to get loaded if the server spontaneously restarts, which they have been known to do. ;)

<pre>
<code>[root@srv sysconfig]# reboot
</code>
</pre>

Now after the reboot there are a couple more block storage devices on this server:

<pre>
<code>[root@srv ~]# ls /dev/fio?
/dev/fioa  /dev/fiob
</code>
</pre>

We want to create a lvm physical volume (pv) on that block device:

<pre>
<code>[root@srv ~]# pvcreate /dev/fioa
  Device /dev/fioa not found (or ignored by filtering).
</code>
</pre>

Ooops. Error message. What went wrong? Well, the "or ignored by filtering" is where to start looking. [This](https://support.fusionio.com/kb/enabling-an-iomemory-device-for-lvm-use/) FusionIO knowledge base entry (which you have to login to see, how annoying is that) shows that we need to add an entry to the _lvm.conf_ on the server:

<pre>
<code>Locate and edit the /etc/lvm/lvm.conf configuration file.
Add an entry similar to the following to that file:
types = [ "fio", 16 ]
</code>
</pre>

That is precisely what I will do.

<pre>
<code>[root@srv lvm]# grep types lvm.conf
    # List of pairs of additional acceptable block device types found 
    # types = [ "fd", 16 ]
    types = [ "fio", 16 ]
</code>
</pre>

And now:

<pre>
<code># let's see if the types were loaded
[root@srv ~]# lvm dumpconfig | grep types
  	types=["fio", 16]
[root@srv ~]# pvcreate /dev/fioa
  Physical volume "/dev/fioa" successfully created
[root@srv ~]# pvcreate /dev/fiob
  Physical volume "/dev/fiob" successfully created
</code>
</pre>

And create a volume group and add the pvs to it.

<pre>
<code>[root@srv ~]# vgcreate hiops /dev/fioa
  Volume group "hiops" successfully created
[root@srv ~]# vgextend hiops /dev/fiob
  Volume group "hiops" successfully extended
[root@srv ~]# vgs
  VG     #PV #LV #SN Attr   VSize   VFree  
  hiops    2   0   0 wz--n- 504.91g 504.91g
  system   1   9   0 wz--n-  58.56g  36.66g
  vm       1  11   2 wz--n-   1.31t 228.09g
</code>
</pre>

I should note at this point that there is only 504g in the hiops volume group when there should be about 600g. 

Previously, using the fio-format command, I had formatted these drives to only 80% capacity. But that was on another server, and I'm not sure it's really necessary to do that unless you are looking for extreme performance or perhaps additional reliability. 

I believe that in some cases with SSD, PCIe or otherwise, it's not a bad idea to use less than 100% of the drive. That said, if you are looking to max out these drives performance-wise, I'd suggest talking to your vendor rather than just listening to me. :)

(AFAIK, these cards can actually take an external power source to increase performance even more. But we don't use that functionality.)

So I'm going to reformat these drives to 100% usage. Just for fun. Why not get back that 100g because the performance/endurance at 100% is going to be fine for our usage.

%{color:blue}Note:% Brand new drives won't have to be formatted. I'm only doing this because I had formatted the drives when they were in the other server. 

%{color:red}Warning:% Reformatting will obviously delete any data on these drives!

<pre>
<code># first detach the /dev/fioa
[root@srv ~]# fio-detach /dev/fct0
Detaching: [====================] (100%) -
[root@srv ~]# fio-format -s 100% /dev/fct0
Creating a device of size 322.55GBytes (300.40GiBytes).
  Using block (sector) size of 512 bytes.

WARNING: Formatting will destroy any existing data on the device!
Do you wish to continue [y/n]? y
Formatting: [====================] (100%) \
Formatting: [====================] (100%)
Format successful.
# then attach...
[root@srv ~]# fio-attach /dev/fct0
Attaching: [====================] (100%) -
fioa
</code>
</pre>

And we can add that device back with _pvcreate_ and then we should see a larger drive:

<pre>
<code>[root@srv ~]# pvcreate /dev/fioa
  Physical volume "/dev/fioa" successfully created
[root@srv ~]# pvs /dev/fioa
  PV         VG    Fmt  Attr PSize   PFree  
  /dev/fioa  hiops lvm2 a-   300.40g 300.40g
</code>
</pre>

I reformatted the other side of the drive back to 100% as well. (With new drives this shouldn't be necessary.)

And the fio-status now is:

<pre>
<code>[root@srv ~]# fio-status

Found 2 ioDrives in this system with 1 ioDrive Duo
Fusion-io driver version: 2.2.3 build 66

Adapter: ioDrive Duo
	IBM 640GB High IOPS MD Class PCIe Adapter, Product Number:68Y7381 SN:XXXXX
	External Power: NOT connected
	PCIE Power limit threshold: 24.75W
	Sufficient power available: Unknown
	Connected ioDimm modules:
	  fct0:	IBM 640GB High IOPS MD Class PCIe Adapter, Product Number:68Y7381 SN:XXXXX
	  fct1:	IBM 640GB High IOPS MD Class PCIe Adapter, Product Number:68Y7381 SN:XXXXX

fct0	Attached as 'fioa' (block device)
	IBM 640GB High IOPS MD Class PCIe Adapter, Product Number:68Y7381 SN:XXXXX
	Alt PN:68Y7382
	Located in slot 0 Upper of ioDrive Duo SN:XXXXX
	PCI:8f:00.0
	Firmware v5.0.6, rev 101583
	322.55 GBytes block device size, 396 GBytes physical device size
	Sufficient power available: Unknown
	Internal temperature: avg 50.2 degC, max 51.2 degC
	Media status: Healthy; Reserves: 100.00%, warn at 10.00%

fct1	Attached as 'fiob' (block device)
	IBM 640GB High IOPS MD Class PCIe Adapter, Product Number:68Y7381 SN:XXXXX
	Alt PN:68Y7382
	Located in slot 1 Lower of ioDrive Duo SN:XXXXX
	PCI:90:00.0
	Firmware v5.0.6, rev 101583
	322.55 GBytes block device size, 396 GBytes physical device size
	Sufficient power available: Unknown
	Internal temperature: avg 46.3 degC, max 46.8 degC
	Media status: Healthy; Reserves: 100.00%, warn at 10.00%


</code>
</pre>

Finally we can create a logical volume (lv) to use.

<pre>
<code>[root@srv ~]# vgs hiops
  VG    #PV #LV #SN Attr   VSize   VFree  
  hiops   1   0   0 wz--n- 300.40g 300.40g
[root@srv ~]# lvcreate -n test -L10.0G /dev/hiops
  Logical volume "test" created
</code>
</pre>

If you have any corrections or other comments, please let me know!

