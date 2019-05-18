---
layout: post
title: Install and Boot an Older Kernel in Ubuntu
categories:

---

# {{ page.title }}

Stangely it was hard to find good instructions on installing an older kernel and setting it to boot on Ubuntu 16.04. Here are some quick instructions.

First, we have a Ubuntu 16.04 instance (in digital ocean) with the following kernel:

```
root@old-kernely:~# uname -a
Linux old-kernely 4.4.0-148-generic #174-Ubuntu SMP Tue May 7 12:20:14 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
root@old-kernely:~# 
```

Let's install 4.4.0-22-generic.

```
root@old-kernely:~# apt install linux-image-4.4.0-22-generic
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following package was automatically installed and is no longer required:
  grub-pc-bin
Use 'apt autoremove' to remove it.
Suggested packages:
  fdutils linux-doc-4.4.0 | linux-source-4.4.0 linux-tools linux-headers-4.4.0-22-generic
The following NEW packages will be installed:
  linux-image-4.4.0-22-generic
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 18.7 MB of archives.
After this operation, 55.4 MB of additional disk space will be used.
Get:1 http://mirrors.digitalocean.com/ubuntu xenial-updates/main amd64 linux-image-4.4.0-22-generic amd64 4.4.0-22.40 [18.7 MB]
Fetched 18.7 MB in 0s (69.0 MB/s)                 
Selecting previously unselected package linux-image-4.4.0-22-generic.
(Reading database ... 54537 files and directories currently installed.)
Preparing to unpack .../linux-image-4.4.0-22-generic_4.4.0-22.40_amd64.deb ...
Done.
Unpacking linux-image-4.4.0-22-generic (4.4.0-22.40) ...
Setting up linux-image-4.4.0-22-generic (4.4.0-22.40) ...
Running depmod.
update-initramfs: deferring update (hook will be called later)
Examining /etc/kernel/postinst.d.
run-parts: executing /etc/kernel/postinst.d/apt-auto-removal 4.4.0-22-generic /boot/vmlinuz-4.4.0-22-generic
run-parts: executing /etc/kernel/postinst.d/initramfs-tools 4.4.0-22-generic /boot/vmlinuz-4.4.0-22-generic
update-initramfs: Generating /boot/initrd.img-4.4.0-22-generic
W: mdadm: /etc/mdadm/mdadm.conf defines no arrays.
run-parts: executing /etc/kernel/postinst.d/unattended-upgrades 4.4.0-22-generic /boot/vmlinuz-4.4.0-22-generic
run-parts: executing /etc/kernel/postinst.d/update-notifier 4.4.0-22-generic /boot/vmlinuz-4.4.0-22-generic
run-parts: executing /etc/kernel/postinst.d/x-grub-legacy-ec2 4.4.0-22-generic /boot/vmlinuz-4.4.0-22-generic
Searching for GRUB installation directory ... found: /boot/grub
Searching for default file ... found: /boot/grub/default
Testing for an existing GRUB menu.lst file ... found: /boot/grub/menu.lst
Searching for splash image ... none found, skipping ...
Found kernel: /boot/vmlinuz-4.4.0-148-generic
Found kernel: /boot/vmlinuz-4.4.0-148-generic
Found kernel: /boot/vmlinuz-4.4.0-22-generic
Updating /boot/grub/menu.lst ... done

run-parts: executing /etc/kernel/postinst.d/zz-update-grub 4.4.0-22-generic /boot/vmlinuz-4.4.0-22-generic
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-4.4.0-148-generic
Found initrd image: /boot/initrd.img-4.4.0-148-generic
Found linux image: /boot/vmlinuz-4.4.0-22-generic
Found initrd image: /boot/initrd.img-4.4.0-22-generic
done
root@old-kernely:~# 
```

OK now that we have the new kernel, how do we make it default. One way would be to reboot and chose that kernel at the grub menu but that is not that easy in a cloud environment, and wouldn't be a permanent change.

Figuring out what to put in /etc/default/grub isn't that easy. Let's look at the submenus and menuentries ([hat tip](https://unix.stackexchange.com/questions/198003/set-default-kernel-in-grub)).

```
root@old-kernely:~# awk '/menuentry/ && /class/ {count++; print count-1"****"$0 }' /boot/grub/grub.cfg                                            
0****menuentry 'Ubuntu' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-3bfdf15c-91ab-470e-a04a-6d95c9a1fbac' {
1****	menuentry 'Ubuntu, with Linux 4.4.0-148-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-4.4.0-148-generic-advanced-3bfdf15c-91ab-470e-a04a-6d95c9a1fbac' {
2****	menuentry 'Ubuntu, with Linux 4.4.0-148-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-4.4.0-148-generic-recovery-3bfdf15c-91ab-470e-a04a-6d95c9a1fbac' {
3****	menuentry 'Ubuntu, with Linux 4.4.0-22-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-4.4.0-22-generic-advanced-3bfdf15c-91ab-470e-a04a-6d95c9a1fbac' {
4****	menuentry 'Ubuntu, with Linux 4.4.0-22-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-4.4.0-22-generic-recovery-3bfdf15c-91ab-470e-a04a-6d95c9a1fbac' {
```

Under the submenu at entry 3 is what we want. But numbering restarts with the submenu. So if we want to boot the 4.4.0-22 kernel we need submenu 1 and menuentry 2.

We add this to /etc/default/grub.

Currently it's set to 0.

```
root@old-kernely:~# grep GRUB_DEFAULT /etc/default/grub
GRUB_DEFAULT=0
```

Lets set it to "1>2".

```
root@old-kernely:~# grep GRUB_DEFAULT /etc/default/grub
GRUB_DEFAULT="1>2"
```

And update grub.

```
root@old-kernely:~# update-grub
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-4.4.0-148-generic
Found initrd image: /boot/initrd.img-4.4.0-148-generic
Found linux image: /boot/vmlinuz-4.4.0-22-generic
Found initrd image: /boot/initrd.img-4.4.0-22-generic
done
```

Then reboot.

```
root@old-kernely:~# reboot
```

When the node reboots it's using the older kernel we specified.

```
root@old-kernely:~# uname -a
Linux old-kernely 4.4.0-22-generic #40-Ubuntu SMP Thu May 12 22:03:46 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
root@old-kernely:~# 
```

Presumably there is a better way to do this. Let me know in the comments!
