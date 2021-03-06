---
layout: post
title: Booting partitions bigger than 2TB on a HP DL160 G6 with RHEL5
categories:
---

# {{ page.title }}

Yesterday I was working on a HP DL160 G6 server. Originally it had two 160GB hard-drives, but of course, I wanted more storage...a lot more. :) So we ordered four 2TB drives to put in it. Then I realized the backplane would only support two drives, so I had to order a backplane that can support four drives.

Once all the parts arrived I replaced the backplane and put the four drives in. It was fairly simple actually. Then when I booted the server with the new backplane and disks the P410 RAID card noticed the new drives, and suggested configuring RAID 1+0, a suggestion I accepted. That leaves me with about 4TB usable.

By default, the system creates one large drive of 4TB, which Redhat Enterprise 5 sees as `/dev/sda`. 

*However*, mbr, the default partition type on RHEL5, cannot boot partitions larger than 2TB. So, after the first install via kickstart I was missing 2TB. Not cool! Well, actually it's fine, the computer did what it was told, but I wanted to use the rest.

The solution? It was actually fairly easy. Maybe too easy. But it's working.

Because, it seems, the HP BIOS in this server supports UEFI/EFI/GPT/whatever, in the `%pre` section of the kickstart we can create a gpt partition.

<pre>
<code>%pre
# B/c sda on this server is 4.0TB we need to try to use gpt instead of mdr.
/usr/sbin/parted -s /dev/sda mklabel gpt
</code>
</pre>

Also make sure that if you have a `clearpart` command in your kickstart to comment it out or delete it.

<pre>
<code># Removing b/c of the parted in %pre
#clearpart --drives=sda --all --initlabel
</code>
</pre>

Then run your kickstart as usual and hopefully your system will boot with whatever partitions you configured. In my case, I created two partitions, one 60GB for the system and the rest for virtual machines, and placed logical volumes over top:

<pre>
<code>4TB_RAID10_SERVER$ pvs
  PV         VG     Fmt  Attr PSize  PFree 
  /dev/sda2  system lvm2 a-   58.56g 36.66g
  /dev/sda3  vm     lvm2 a-    3.58t  3.58t
</code>
</pre>

And running `parted` we can see that it is indeed a gpt layout:

<pre>
<code>4TB_RAID10_SERVER$ parted /dev/sda print
Model: HP LOGICAL VOLUME (scsi)
Disk /dev/sda: 4001GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  525MB   524MB   ext4               boot
 2      525MB   63.4GB  62.9GB                     lvm
 3      63.4GB  4001GB  3937GB                     lvm
</code>
</pre>

So now I can create that rsync backup server I've always wanted, and I have up to 3.58TB to store the backups on. Good times and I'm glad it all worked out. Now if only they were 3TB drives... *;)*

