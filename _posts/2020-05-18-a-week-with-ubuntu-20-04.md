---
layout: post
title: A Week with Ubuntu 20.04
categories:
header_image: "/img/fossa-drawing.jpg"
header_permalink: "https://commons.wikimedia.org/wiki/File:Fossa-drawing.jpg"
summary: "7 days with Focal Fossa on the Desktop"

---

# {{ page.title }}

I know a lot of people like to make fun of running Linux on the desktop, and maybe it deserves the criticism that has built up year over year, like a thousand paper cuts (which never seem to heal, but I digress). I'll be honest and say that I love running Linux as my main OS, and am much more proficient with it than any other operating system. Some people like Linux on the desktop and some don't. That's fine with me!

Because I feel like I'm going to be doing a lot more work from home, I decided it was time to get a new linux workstation/desktop. So I did. Given that Ubuntu 20.04 recently came out, I thought that sounded like a good distribution of Linux to run, and I especially liked the idea of using ZFS as the boot file system.

* Hardware

The hardware I'm using is really entry level, but boy, does it feel faster than my mac laptop and, of course, my old circa 2012 desktop. (Amazing to have 1TB NVMe drive for $170 CDN.)

```
AMD Ryzen™ 5 3600 Processor, 3.6GHz w/ 35MB Cache 
Corsair Vengeance LPX 64GB DDR4 2666MHz CL16 Dual Channel Kit (4x 16GB), Black 
Western Digital Blue SN550 M.2 PCI-E NVMe SSD, 1TB 
Asus TUF B450-PRO GAMING w/ DDR4-2666, 7.1 Audio, Gigabit LAN, CrossFire 
```

At least with 64G of main memory I can keep some tabs open.

I have not seen any errors, issues, etc, with the hardware so far.

* ZFS

When I installed 20.04 I decided vto install via the [experimental](https://linuxconfig.org/install-ubuntu-20-04-with-zfs) use of ZFS as the boot file system. The interesting thing about ZFS here is that you can get [snapshots](https://www.phoronix.com/scan.php?page=news_item&px=Trying-Ubuntu-20.04-ZFS-Snaps) of your OS.  When a new package is added via apt a zfs snapshot will be taken. It's possible to revert by booting from the snapshot. For example, here I install wireguard. Note the "INFO" lines regarding creating a snapshot and updating grub.

```
$ sudo apt install wireguard
[sudo] password for curtis: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  wireguard-tools
The following NEW packages will be installed:
  wireguard wireguard-tools
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 85.3 kB of archives.
After this operation, 341 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://ca.archive.ubuntu.com/ubuntu focal/universe amd64 wireguard-tools amd64 1.0.20200319-1ubuntu1 [82.4 kB]
Get:2 http://ca.archive.ubuntu.com/ubuntu focal/universe amd64 wireguard all 1.0.20200319-1ubuntu1 [2,912 B]
Fetched 85.3 kB in 0s (544 kB/s)      
INFO Requesting to save current system state      
Successfully saved as "autozsys_hku7gp"
Selecting previously unselected package wireguard-tools.
(Reading database ... 177462 files and directories currently installed.)
Preparing to unpack .../wireguard-tools_1.0.20200319-1ubuntu1_amd64.deb ...
Unpacking wireguard-tools (1.0.20200319-1ubuntu1) ...
Selecting previously unselected package wireguard.
Preparing to unpack .../wireguard_1.0.20200319-1ubuntu1_all.deb ...
Unpacking wireguard (1.0.20200319-1ubuntu1) ...
Setting up wireguard-tools (1.0.20200319-1ubuntu1) ...
Setting up wireguard (1.0.20200319-1ubuntu1) ...
Processing triggers for man-db (2.9.1-1) ...
INFO Updating GRUB menu     
```

There is a boot pool and a root pool.

```
$ zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
bpool  1.88G   260M  1.62G        -         -     0%    13%  1.00x    ONLINE  -
rpool   920G   110G   810G        -         -     5%    11%  1.00x    ONLINE  -
```

* Kubernetes in Docker

I do a lot of work with k8s, and would prefer to use Kind locally, but it doesn't work with ZFS. Minikube is fine because it runs in its own VM, which is perhaps better security-wise anyways.

* kvm, virt-manager

I run several VMs to keep some separation. I have work Windows, work Linux demo vm, etc. With 64G of ram I have enough room for several vms and many tabs. 12 cpus helps too. So far no problems with KVM.

* Crashes

I have had a couple of crashes. One when I installed Docker. Not sure what happened there. That concerns me. That said, every desktop I've used in the last year crashes. Windows crashes. OSX crashes. Linux vms crash on OSX. Linux workstation crashes. Crash crash crash crash crash crash. No OS stays up 100% of the time. Brutal out there.

* GUI

A lot of the features mentioned for Fossa are related to the Gnome GUI. I use i3, which pretty much abstracts away the underlying GUI so I don't see it. It's to the point where my workflow is just 100% i3 based.

* Snaps

I haven't used snaps much. I use vscode and you can install it from snaps but I just did the package based install. That said I don't have a problem with Snaps either. There does seem to be some controversy around it, but packaging is hard, and I'm all for options being available.

```
$ snap list
Name               Version             Rev   Tracking         Publisher   Notes
core18             20200427            1754  latest/stable    canonical✓  base
gnome-3-34-1804    0+git.3009fc7       33    latest/stable/…  canonical✓  -
gtk-common-themes  0.1-36-gc75f853     1506  latest/stable/…  canonical✓  -
snap-store         3.36.0-74-ga164ec9  433   latest/stable/…  canonical✓  -
snapd              2.44.3              7264  latest/stable    canonical✓  snapd
```

* Wireguard

I do use Wireguard, so having it back ported to the Fossa kernel is useful,no compiling for kernel module necessary.

* Encryption

I usually encrypt the OS device. But I haven't in this workstation, I guess because of ZFS I just didn't think of it at the time. Something to consider in the future.

* xfreerdp and i3

I run a couple of vms for demos and work, and this xfreerdp command works great and still allows me to flit around virtual desktops with $MOD+NUM without having to escape the virtual session. That said I haven't tested the mic yet, but the "speaker" works great for audio in the vm. What I mean is that I can use i3 on the desktop and windows in the vm, and jump back to linux with $MOD+NUM.

```
xfreerdp /u:curtis /p:$PASS /v:$IP /f +fonts /floatbar /smart-sizing -grab-keyboard /sound /microphone
```

That is working really well and I'm super happy with it.

## Updates 

zfs and docker issue...I could not remove a container, and had to use the below script to temporarily recreate the pool and then rm the container.

```
#!/bin/bash

stuck=$(docker ps -a | grep Removal | cut -f1 -d' ')
echo $stuck
for container in $stuck; do
	zfs_path=$(docker inspect $container | jq -c '.[] | select(.State | .Status == "dead")|.GraphDriver.Data.Dataset')
	zfs_path=$(echo $zfs_path|tr -d '"')
	sudo zfs destroy -R $zfs_path
	sudo zfs destroy -R $zfs_path-init
    sudo zfs create $zfs_path
    sudo zfs create $zfs_path-init
	docker rm $container
done
```



## Conclusion

So far, so good. Other than ZFS as the boot file system, I don't actually see that much different with Focal Fossa. Everything works fine like usual, yes, even sound!