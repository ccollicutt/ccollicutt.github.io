---
layout: post
title: Using the Cisco 1000v CSR with Libvirt and KVM
categories:

---
 
# {{ page.title }}

This blog post is going to cover installing and booting the [Cisco 1000v Cloud Services Router](http://www.cisco.com/c/en/us/products/routers/cloud-services-router-1000v-series/index.html) with KVM on Ubuntu Trusty 14.04. 

It's important to note that I haven't touched a Cisco device in over a decade. At least until the blog post that is. I've been an open source based systems administrator for all of my career, and now I would like to learn a bit more about networking, which is considerably more closed source than I am used to. Thankfully, however, Cisco offers a free download of their cloud router.

## Getting the Cisco 1000v CSR images and ISOs

First, if you don't have an account with [Cisco](http://cisco.com), then create one and login.

To find the download page, this [link](http://software.cisco.com/download/release.html?mdfid=284364978&softwareid=282046477&release=3.12.0S&flowid=39582) might work. If if doesn't work:

1. [CSR Index page](http://www.cisco.com/c/en/us/products/routers/cloud-services-router-1000v-series/index.html)
2. Right menu, "Download Software for this Product"
3. Right panel, "Cisco Cloud Services outer 1000V"
4. Select "IOS XE Software"
5. Pick your download, ISO, OVA, qcow2, etc.

In this post I'm going to work from the ISO.

## Install from ISO into qcow2 image

Using the ISO process, we boot the ISO using KVM, which will automatically install the router software onto the disk image specified. Once that's done you can boot the qcow2 image like any regular virtual machine.

I've downloaded the ISO file.

<pre>
<code># ls *.iso
csr1000v-universalk9.03.12.00.S.154-2.S-std.iso
</code>
</pre>

Next, create a backing image to install the software onto. It has to be at least 8 gigs.

<pre>
<code># qemu-img create -f qcow2 csr.img 8G
Formatting 'csr.img', fmt=qcow2 size=8589934592 encryption=off cluster_size=65536 lazy_refcounts=off 
</code>
</pre>

Now we can boot the ISO with KVM and set the backing image on which the ISO will install the 1000v router. Note that you need to hit a key pretty quickly to get to the GRUB boot menu.

<pre>
<code># kvm -boot d csr.img -enable-kvm -m 4096M -cpu Nehalem -smp 4,sockets=4,cores=1,threads=1 -cdrom csr1000v-universalk9.03.12.00.S.154-2.S-std.iso -nographic
</code>
</pre>

After hitting the "any" key, you should see the below. Select "Serial Console" and hit enter.

<pre>
<code>   GNU GRUB  version 0.97  (639K lower / 3144696K upper memory)

 +-------------------------------------------------------------------------+
 | CSR 1000V Virtual Console -- Wed-26-Mar-14-15:35                        |  
 | CSR 1000V Serial Console -- Wed-26-Mar-14-15:35                         |
 |                                                                         |
 |                                                                         |
 |                                                                         |
 |                                                                         |
 |                                                                         |
 |                                                                         |
 |                                                                         |
 |                                                                         |
 |                                                                         |
 |                                                                         |  
 +-------------------------------------------------------------------------+
      Use the ^ and v keys to select which entry is highlighted.
      Press enter to boot the selected OS, or 'c' for a command-line.
</code>
</pre>

At this point the ISO should install the 1000v CSR router into the csr.img qcow2 file, and some text should fly by, such as the below. It should only take a minute or two to install the CSR onto the hd image.

<pre>
<code>Booting 'CSR 1000V Serial Console -- Wed-26-Mar-14-15:35'

root (cd)
 Filesystem type is iso9660, using whole disk
kernel /boot/csr1000v-universalk9.03.12.00.S.154-2.S-std.SPA.bin rw root=/dev/r
am quiet console= max_loop=64 HARDWARE=virtual SR_BOOT=cdrom:csr1000v-universal
k9.03.12.00.S.154-2.S-std.iso
package header rev 1 structure detected
Calculating SHA-1 hash...done
SHA-1 hash:
        calculated   f51efee9:bfc569d7:9a732dee:4af42ccc:7003719d
        expected     f51efee9:bfc569d7:9a732dee:4af42ccc:7003719d
Package type:0x7530, flags:0x0
   [Linux-bzImage, setup=0x2e00, size=0x11706720]
   [isord @ 0x6fe6c000, 0x10183000 bytes]
SNIP!
%IOSXEBOOT-4-BOOT_CDROM: (rp/0): Installing GRUB
%IOSXEBOOT-4-BOOT_CDROM: (rp/0): Copying super package csr1000v-universalk9.03.12.00.S.154-2.S-std.SPA.bin
%IOSXEBOOT-4-BOOT_CDROM: (rp/0): Expanding super package on /bootflash
%IOSXEBOOT-4-BOOT_CDROM: (rp/0): Creating /boot/grub/menu.lst
%IOSXEBOOT-4-BOOT_CDROM: (rp/0): CD-ROM Installation finished
%IOSXEBOOT-4-BOOT_CDROM: (rp/0): Ejecting CD-ROM tray
%IOSXEBOOT-4-BOOT_CDROM: (rp/0): Rebooting from HD
SNIP!
Press any key to continue.
Press any key to continue.
Press any key to continue.
</code>
</pre>

Again you have to be quick on hitting a key when the "Press any key to continue" message comes up. (Must be a better way to do this.) The vm is now booting off of the hd image instead of the ISO image.

Select virtual console once more and the router should boot up and ask if we want to configure from a dialog. 

<pre>
<code>SNIP!
cisco CSR1000V (VXE) processor with 2170596K/6147K bytes of memory.
Processor board ID 9W17YZL34P2
1 Gigabit Ethernet interface
32768K bytes of non-volatile configuration memory.
4194304K bytes of physical memory.
7774207K bytes of virtual hard disk at bootflash:.


         --- System Configuration Dialog ---

Would you like to enter the initial configuration dialog? [yes/no]:
</code>
</pre>

I enter no, though you may want to enter yes. The router continues booting, eventually stops and we can just hit enter to get the "Router>" prompt.

<pre>
<code>SNIP!
*Jul 15 01:40:39.304: %VMAN-5-PACKAGE_SIGNING_LEVEL_ON_INSTALL: F0: vman:  Package 'csrmgmt.1_3_1.20140213_121708.ova
Building configuration...
' for service container 'csr_mgmt' is 'Cisco signed', signing level cached on original install is 'Cisco signed'
*Jul 15 01:40:39.745: Not MO, application name is csr_mgmt
*Jul 15 01:40:39.745: %VIRT_SERVICE-5-INSTALL_STATE: Successfully installed virtual service csr_mgmt
*Jul 15 01:40:39.748: IOS-FIREWALL-POLICY-SHIM-REGISTER[OK]
*Jul 15 01:40:42.273: %CONFIG_CSRLXC-5-CONFIG_DONE: Configuration was applied and saved to NVRAM. See bootflash:/csrlxc-cfg.log for more details.
# hit enter!
Router> #here we are at the router prompt!
</code>
</pre>

Now we have a full fledged Cisco router!

At this point I usually kill the KVM process from another terminal, and the installation is complete. (Killing from another terminal is a little awkward, so let me know if you find a better way, which probably involves not using "-serial stdio", but I like that it just streams in the terminal. Lots of ways to do this.)

## Using the image file

Now that the ISO has finished installing the software, we have an image file to work with.

<pre>
<code># du -hs csr.img
1.6G	csr.img
# file csr.img
csr.img: QEMU QCOW Image (unknown version)
</code>
</pre>

Copy that image to /var/lib/libvirt/images.

<pre>
<code># cp csr.img /var/lib/libvirt/images/
</code>
</pre>

In /var/lib/libvirt/images, create a qcow2 snapshot.

<pre>
<code># qemu-img create -f qcow2 -b csr.img csr-01.img
Formatting 'csr-01.img', fmt=qcow2 size=8589934592 backing_file='csr.img' encryption=off cluster_size=65536 lazy_refcounts=off 
</code>
</pre>

Now we can use that image with libvirt.

## Libvirt XML file

Create an XML file like the below, ensuring to replace the image file location if necessary. I believe the CSR requires 4GB of memory and 4 VCPUS.

Note that if you want the CSR to have more than one interface, you'll have to add it to the XML file, and perhaps add networks to libvirt.

<pre>
<code># cat csr-01.xml 
<domain type='kvm'>
  <name>csr-01/name>
  <memory unit='KiB'>4194304</memory>
  <currentMemory unit='KiB'>4194304</currentMemory>
  <vcpu placement='static'>4</vcpu>
  <os>
    <type arch='x86_64' machine='pc-0.14'>hvm</type>
    <boot dev='hd'/>
  </os>
  <cpu>
    <model>Nehalem</model>
  </cpu>
  <features>
    <acpi/>
    <apic/>
    <pae/>
  </features>
  <clock offset='utc'/>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <devices>
    <emulator>/usr/bin/kvm</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/var/lib/libvirt/images/csr-01.img'/>
      <target dev='vda' bus='virtio'/>
    </disk>
    <controller type='usb' index='0'>
    </controller>
  <interface type='network'>
     <source network='default'/>
            <model type='virtio'/>
  </interface>
    <serial type='pty'>
      <target port='0'/>
    </serial>
    <console type='pty'>
      <target type='serial' port='0'/>
    </console>
    <input type='tablet' bus='usb'/>
    <input type='mouse' bus='ps2'/>
    <graphics type='vnc' port='-1' autoport='yes'/>
    <sound model='ich6'>
    </sound>
    <video>
      <model type='cirrus' vram='9216' heads='1'/>
    </video>
    <memballoon model='virtio'>
    </memballoon>
  </devices>
</domain>
</code>
</pre>

Define the vm and start it.

<pre>
<code># virsh define csr-01.xml
Domain csr-01 defined from csr-01.xml

# virsh start csr-01
Domain csr-01 started
</code>
</pre>

You can use "virsh console csr-01" to access the console. To exit (at least when using OSX's terminal) hit "CTRL-5."

<pre>
<code># virsh console csr-01
  Booting 'CSR1000v - packages.conf'

root (hd0,0)
 Filesystem type is ext2fs, partition type 0x83
kernel /packages.conf rw quiet root=/dev/ram console= max_loop=64 HARDWARE=virt
ual SR_BOOT=bootflash:packages.conf
Calculating SHA-1 hash...done
SHA-1 hash:
        calculated   514e2831:94ee1441:2404193c:f37dac1e:4c196e19
        expected     514e2831:94ee1441:2404193c:f37dac1e:4c196e19
package header rev 1 structure detected
Calculating SHA-1 hash...done
SHA-1 hash:
        calculated   134e1e2e:319d85c6:34a4d2b3:965dcb75:dc20afef
        expected     134e1e2e:319d85c6:34a4d2b3:965dcb75:dc20afef
Package type:0x7531, flags:0x0
   [Linux-bzImage, setup=0x2e00, size=0xd1c0720]
   [isord @ 0x743b2000, 0xbc3d000 bytes]
   SNIP!
</code>
</pre>

Check the interfaces and show version.

<pre>
<code>SNIP!
*Jul 15 02:46:04.712: %CONFIG_CSRLXC-5-CONFIG_DONE: Configuration was applied and saved to NVRAM. See bootflash:/csrlxc-cfg.log for more details.
Router>enable
Router#show int desc
Interface                      Status         Protocol Description
Gi1                            admin down     down     
Gi2                            admin down     down     
Gi3                            admin down     down     
Router#show version
Cisco IOS XE Software, Version 03.12.00.S - Standard Support Release
Cisco IOS Software, CSR1000V Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 15.4(2)S, RELEASE SOFTWARE (fc2)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2014 by Cisco Systems, Inc.
Compiled Wed 26-Mar-14 21:09 by mcpre
SNIP!
</code>
</pre>

Again, you can hit "CTRL-5" to exit "virsh console."

## Conclusion

Now that we have a base image and a working libvirt XML file, we can create all kinds of interesting network configurations and learn how to use a Cisco router without actually having any Cisco hardware. Nothing is stopping you from booting several CSR virtual machines and configuring them to work together.


