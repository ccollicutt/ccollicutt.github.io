---
layout: post
title: OCZ Z-Drive R4 Installation and Performance  
categories:
---

# {{ page.title }}

In a previous [post](http://serverascode.com/2012/04/13/11-ocz-zdrive-r4s) I mentioned how we had purchased 11 300GB [OCZ Z-Drive R4 PCIe-SSD cards](http://www.oczenterprise.com/ssd-products/z-drive-c-series.html). (Please note that this was a special case purchase--the cards didn't meet any specific requirements we had other than that they were easily available, PCIe-SSD, and low profile.)

We bought the low profile version because these drives are going into the [Supermicro SC847E16-R1400LPB](http://www.supermicro.com/products/chassis/4U/847/SC847E1-R1400LP.cfm) chassis (the subject of future posts), which have room for seven low profile cards. I believe the full height zdrive R4s are faster, so this is a compromise.

## Installation

Each of our servers is going to get one zdrive. I placed them in a x8 slot.

Once the OS is up and installed (these cards are not bootable, ie. the OS can't be installed onto the cards) the [proprietary kernel module](http://www.oczenterprise.com/drivers.html) needs to be loaded.

There is an [installation guide](http://www.oczenterprise.com/downloads/solutions/z-driver4-installation-guide.pdf).

I'm running Centos 6.2 on these servers.

<pre>
<code># cat /etc/redhat-release 
CentOS release 6.2 (Final)
# uname -a
Linux ocz_server 2.6.32-220.el6.x86_64 #1 SMP Tue Dec 6 19:48:22 GMT 2011 x86_64 x86_64 x86_64 GNU/Linux

</code>
</pre>

I'm using the:

<pre>
<code>Red Hat Enterprise Linux 6.x, CentOS 6.x 64-bit	1.0.0.1480	Mar 2, 2012
</code>
</pre>

version of the driver.

When that tar file is downloaded and unzipped all there is inside is the _ocz10xx.ko_ kernel module.

<pre>
<code># wget http://www.oczenterprise.com/files/drivers/OCZ%20RHEL-Centos_6.x_64-Bit_r1480.tar.gz 
--2012-05-08 21:40:23--  http://www.oczenterprise.com/files/drivers/OCZ%20RHEL-Centos_6.x_64-Bit_r1480.tar.gz
Resolving www.oczenterprise.com... 74.52.187.58
Connecting to www.oczenterprise.com|74.52.187.58|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4072553 (3.9M) [application/x-gzip]
Saving to: “OCZ RHEL-Centos_6.x_64-Bit_r1480.tar.gz”

100%[======================================>] 4,072,553   1.03M/s   in 4.0s    

2012-05-08 21:40:27 (991 KB/s) - “OCZ RHEL-Centos_6.x_64-Bit_r1480.tar.gz” saved [4072553/4072553]

# tar zxvf OCZ\ RHEL-Centos_6.x_64-Bit_r1480.tar.gz 
ocz10xx.ko
</code>
</pre>

which can be loaded by:

<pre>
<code># insmod ocz10xx.ko
# lsmod | grep ocz
ocz10xx               479350  1 
</code>
</pre>

When that module is loaded the following is reported to dmesg:

<pre>
<code>ocz10xx: module license 'Proprietary' taints kernel.
Disabling lock debugging due to kernel taint
ocz10xx module is older than RHEL 6.2 ... applying fixups
  alloc irq_desc for 26 on node -1
  alloc kstat_irqs on node -1
ocz10xx 0000:07:00.0: PCI INT A -> GSI 26 (level, low) -> IRQ 26
ocz10xx 0000:07:00.0: setting latency timer to 64
OCZ Storage Controller is found, using IRQ 26, driver version 2.0.0.1480.
OCZ Linux driver ocz10xx, driver version 2.0.0.1480.
OCZ DRIVE LEVEL=OCZ_FAST, STATE=ONLINE
scsi5 : OCZ Storage Controller
scsi 5:0:126:0: Direct-Access     ATA      OCZ Z-DRIVE R4 C 2.15 PQ: 0 ANSI: 5
sd 5:0:126:0: Attached scsi generic sg8 type 0
sd 5:0:126:0: [sdg] 586135549 512-byte logical blocks: (300 GB/279 GiB)
sd 5:0:126:0: [sdg] Write Protect is off
sd 5:0:126:0: [sdg] Mode Sense: 41 00 00 00
sd 5:0:126:0: [sdg] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
 sdg: unknown partition table
sd 5:0:126:0: [sdg] Attached SCSI disk
</code>
</pre>

and I now have a _/dev/sdg_ to use. 

## Loading the kernel module at boot

First, let me say that I don't have a lot of experience with kernel modules. I'm hoping that if I've made a mistake that someone will alert me in the comments. Or perhaps I missed where this is documented by OCZ.

Running _insmod_ is great for the first time one tries out the zdrive, but what happens after a reboot?

Usually kernel modules go in _/lib/modules/`uname -r`_ but this module doesn't seem to be tied to a particular kernel version. While I could put it in that directory, each time I get a new kernel I'd have to move it. This would not be good for maintainability. Assuming the module works with all 6.x kernels--which is what the OCZ drivers page suggests--it should be Ok to put this module in a more permanent location.

What I did was build and RPM with three files:

<pre>
<code># rpm -qf /etc/depmod.d/ocz-zdrive-r4.conf 
ocz-zdrive-r4-r1480-2.el6.x86_64
# rpm -qf /etc/modprobe.d/ocz-zdrive-r4.conf 
ocz-zdrive-r4-r1480-2.el6.x86_64
# rpm -qf /usr/share/ocz-zdrive-r4/module/ocz10xx.ko 
ocz-zdrive-r4-r1480-2.el6.x86_64
</code>
</pre>

The _.conf_ files contain:

<pre>
<code># cat /etc/depmod.d/ocz-zdrive-r4.conf 
search /usr/share/ocz-zdrive-r4/module
# cat /etc/modprobe.d/ocz-zdrive-r4.conf 
alias ocz10xx ocz
</code>
</pre>

which will ensure that the _ocz10xx.ko_ module is loaded with all the other kernel modules, so that you can put file systems on the zdrive into fstab and have them mounted at boot:

<pre>
<code># uptime
 23:05:24 up 15 min,  2 users,  load average: 0.00, 0.02, 0.00
# lsmod | grep ocz
ocz10xx               479350  1 
# mount | grep ocz
/dev/mapper/ocz-test on /mnt/ocz-xfs-test type xfs (rw)
# cat /etc/fstab | grep ocz
/dev/mapper/ocz-test /mnt/ocz-xfs-test          xfs    defaults        1
</code>
</pre>

Please let me know if there is something wrong with the methodology. :)

## Performance testing

As I've said before, good performance testing is hard to do. All I can really do at this point is run the same tests that [FusionIO](https://support.fusionio.com/kb/verifying-linux-system-performance/) (GAH! Behind a support login now! Bad FusionIO, bad!) suggests running on their drives.

%{color:red}WARNING:% The write tests will destroy data on the drive! 

%{color:blue}NOTE:% A little bird told me that you need to run the write tests first, otherwise the flash drive--if it's empty--may (depending on the vendor, perhaps) know it's empty and return zeroes and you'll be testing RAM instead of the card.

### Write bandwidth test

<pre>
<code># fio --filename=/dev/sdg --direct=1 --rw=randwrite --bs=1m \
--size=5G --numjobs=4 --runtime=10 --group_reporting --name=file1
file1: (g=0): rw=randwrite, bs=1M-1M/1M-1M, ioengine=sync, iodepth=1
...
file1: (g=0): rw=randwrite, bs=1M-1M/1M-1M, ioengine=sync, iodepth=1
fio 2.0.7
Starting 4 processes
Jobs: 4 (f=4): [wwww] [100.0% done] [0K/1021M /s] [0 /974  iops] [eta 00m:00s]
file1: (groupid=0, jobs=4): err= 0: pid=2444
  write: io=9281.0MB, bw=948572KB/s, iops=926 , runt= 10019msec
    clat (usec): min=679 , max=81086 , avg=4111.26, stdev=4974.85
     lat (usec): min=848 , max=81251 , avg=4313.21, stdev=4974.67
    clat percentiles (usec):
     |  1.00th=[ 1704],  5.00th=[ 1928], 10.00th=[ 2064], 20.00th=[ 2672],
     | 30.00th=[ 2736], 40.00th=[ 2800], 50.00th=[ 2960], 60.00th=[ 3408],
     | 70.00th=[ 3568], 80.00th=[ 3760], 90.00th=[ 4960], 95.00th=[ 9664],
     | 99.00th=[35584], 99.50th=[36608], 99.90th=[38144], 99.95th=[38656],
     | 99.99th=[81408]
    bw (KB/s)  : min=139912, max=273338, per=25.14%, avg=238498.74, stdev=28729.62
    lat (usec) : 750=0.09%, 1000=0.03%
    lat (msec) : 2=8.60%, 4=76.87%, 10=9.69%, 20=2.80%, 50=1.90%
    lat (msec) : 100=0.03%
  cpu          : usr=4.53%, sys=2.98%, ctx=9287, majf=0, minf=120
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=9281/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
  WRITE: io=9281.0MB, aggrb=948572KB/s, minb=948572KB/s, maxb=948572KB/s, mint=10019msec, maxt=10019msec

Disk stats (read/write):
  sdg: ios=83/18421, merge=578/0, ticks=13/69730, in_queue=69712, util=99.21%
</code>
</pre>

### Read IOPS test

<pre>
<code># fio --filename=/dev/sdg --direct=1 --rw=randread --bs=4k \
--size=5G --numjobs=64 --runtime=10 --group_reporting --name=file1
file1: (g=0): rw=randread, bs=4K-4K/4K-4K, ioengine=sync, iodepth=1
...
file1: (g=0): rw=randread, bs=4K-4K/4K-4K, ioengine=sync, iodepth=1
fio 2.0.7
Starting 64 processes
Jobs: 64 (f=64): [rrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrr
Jobs: 64 (f=64): [rrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrr
Jobs: 64 (f=64): [rrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrr
Jobs: 64 (f=64): [rrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrr
Jobs: 64 (f=64): [rrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrr
Jobs: 64 (f=64): [rrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrr
Jobs: 64 (f=64): [rrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrr
Jobs: 64 (f=64): [rrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrr
Jobs: 64 (f=64): [rrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrr]
 [100.0% done] [374.1M/0K /s] [91.4K/0  iops] [eta 00m:00s]
file1: (groupid=0, jobs=64): err= 0: pid=2465
  read : io=3589.4MB, bw=367442KB/s, iops=91860 , runt= 10003msec
    clat (usec): min=100 , max=283036 , avg=693.42, stdev=1539.29
     lat (usec): min=101 , max=283036 , avg=693.61, stdev=1539.29
    clat percentiles (usec):
     |  1.00th=[  262],  5.00th=[  378], 10.00th=[  438], 20.00th=[  506],
     | 30.00th=[  556], 40.00th=[  604], 50.00th=[  652], 60.00th=[  700],
     | 70.00th=[  756], 80.00th=[  828], 90.00th=[  948], 95.00th=[ 1064],
     | 99.00th=[ 1400], 99.50th=[ 1592], 99.90th=[ 2288], 99.95th=[ 2832],
     | 99.99th=[56064]
    bw (KB/s)  : min=  816, max= 7032, per=1.55%, avg=5706.40, stdev=599.67
    lat (usec) : 250=0.81%, 500=17.90%, 750=50.29%, 1000=23.77%
    lat (msec) : 2=7.05%, 4=0.16%, 10=0.01%, 20=0.01%, 50=0.01%
    lat (msec) : 100=0.01%, 250=0.01%, 500=0.01%
  cpu          : usr=0.56%, sys=6.49%, ctx=919527, majf=0, minf=2240
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=918880/w=0/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
   READ: io=3589.4MB, aggrb=367441KB/s, minb=367441KB/s, maxb=367441KB/s, mint=10003msec, maxt=10003msec

Disk stats (read/write):
  sdg: ios=914696/0, merge=0/0, ticks=612829/0, in_queue=607382, util=98.84%
</code>
</pre>

### Read bandwidth test

<pre>
<code># fio --filename=/dev/sdg --direct=1 --rw=randread --bs=1m --size=5G \
 --numjobs=4 --runtime=10 --group_reporting --name=file1
file1: (g=0): rw=randread, bs=1M-1M/1M-1M, ioengine=sync, iodepth=1
...
file1: (g=0): rw=randread, bs=1M-1M/1M-1M, ioengine=sync, iodepth=1
fio 2.0.7
Starting 4 processes
Jobs: 4 (f=4): [rrrr] [100.0% done] [1599M/0K /s] [1524 /0  iops] [eta 00m:00s]
file1: (groupid=0, jobs=4): err= 0: pid=2543
  read : io=16475MB, bw=1647.2MB/s, iops=1647 , runt= 10002msec
    clat (usec): min=828 , max=79515 , avg=2423.79, stdev=1154.98
     lat (usec): min=828 , max=79515 , avg=2424.04, stdev=1154.98
    clat percentiles (usec):
     |  1.00th=[ 1528],  5.00th=[ 1768], 10.00th=[ 1912], 20.00th=[ 2064],
     | 30.00th=[ 2160], 40.00th=[ 2256], 50.00th=[ 2320], 60.00th=[ 2416],
     | 70.00th=[ 2544], 80.00th=[ 2736], 90.00th=[ 2992], 95.00th=[ 3280],
     | 99.00th=[ 3856], 99.50th=[ 4128], 99.90th=[ 6176], 99.95th=[13120],
     | 99.99th=[78336]
    bw (KB/s)  : min=369211, max=526336, per=25.10%, avg=423322.49, stdev=33254.52
    lat (usec) : 1000=0.18%
    lat (msec) : 2=14.73%, 4=84.40%, 10=0.63%, 20=0.04%, 100=0.02%
  cpu          : usr=0.19%, sys=5.89%, ctx=16488, majf=0, minf=1151
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=16475/w=0/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
   READ: io=16475MB, aggrb=1647.2MB/s, minb=1647.2MB/s, maxb=1647.2MB/s, mint=10002msec, maxt=10002msec

Disk stats (read/write):
  sdg: ios=32621/0, merge=0/0, ticks=71360/0, in_queue=71316, util=99.09%
</code>
</pre>

### Write IOPS test

<pre>
<code># fio --filename=/dev/sdg --direct=1 --rw=randwrite --bs=4k --size=5G \
--numjobs=64 --runtime=10 --group_reporting --name=file
file: (g=0): rw=randwrite, bs=4K-4K/4K-4K, ioengine=sync, iodepth=1
...
file: (g=0): rw=randwrite, bs=4K-4K/4K-4K, ioengine=sync, iodepth=1
fio 2.0.7
Starting 64 processes
Jobs: 64 (f=64): [wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwJobs: 
 64 (f=64): [wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwJobs: 64
 (f=64): [wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwJobs: 64
 (f=64): [wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwJobs: 64
 (f=64): [wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwJobs: 64
 (f=64): [wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwJobs: 64
 (f=64): [wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwJobs: 64
 (f=64): [wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwJobs: 64
 (f=64): [wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwww] 
[100.0% done] [0K/408.8M /s] [0 /99.8K iops] [eta 00m:00s]
file: (groupid=0, jobs=64): err= 0: pid=2556
  write: io=3670.1MB, bw=374777KB/s, iops=93694 , runt= 10030msec
    clat (usec): min=40 , max=302579 , avg=677.08, stdev=1765.33
     lat (usec): min=40 , max=302580 , avg=678.03, stdev=1765.34
    clat percentiles (usec):
     |  1.00th=[  117],  5.00th=[  390], 10.00th=[  450], 20.00th=[  506],
     | 30.00th=[  548], 40.00th=[  580], 50.00th=[  620], 60.00th=[  652],
     | 70.00th=[  692], 80.00th=[  748], 90.00th=[  820], 95.00th=[  892],
     | 99.00th=[ 1064], 99.50th=[ 1144], 99.90th=[31616], 99.95th=[32640],
     | 99.99th=[33536]
    bw (KB/s)  : min= 2208, max= 9448, per=1.56%, avg=5834.54, stdev=562.51
    lat (usec) : 50=0.25%, 100=0.58%, 250=1.49%, 500=16.44%, 750=62.05%
    lat (usec) : 1000=16.98%
    lat (msec) : 2=1.91%, 4=0.06%, 10=0.09%, 20=0.02%, 50=0.11%
    lat (msec) : 100=0.01%, 250=0.01%, 500=0.01%
  cpu          : usr=0.68%, sys=6.54%, ctx=942753, majf=0, minf=2070
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=939753/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
  WRITE: io=3670.1MB, aggrb=374776KB/s, minb=374776KB/s, maxb=374776KB/s, mint=10030msec, maxt=10030msec

Disk stats (read/write):
  sdg: ios=609/926539, merge=2759/0, ticks=337/599297, in_queue=594561, util=99.08%
</code>
</pre>

## Conclusion

From a cursory look these drives seem to perform well. At least when they are brand new. :) We'll see how they perform over time.

If anyone would like to see specific tests, please let me know.

