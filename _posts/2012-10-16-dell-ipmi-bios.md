---
layout: post
title: ipmitool and BIOS Access
categories:
---

# {{ page.title }}

When I put my systems administrator hat on one of the things that bothers me is getting remote access to the BIOS. I love to use the command line, and any time I have to fire up a GUI of some kind to do work I get an icky feeling.

So when I had to remotely access some Dell C6220s to turn on virtualization, which is off by default in these Dell servers, I didn't want to have to go through the rigamarole of getting the right OS configuration that would allow me to remotely access the console via a web-based Java GUI run out of Firefox. Even writing that sentence takes too long. So I thought I would try out serial over lan (SOL) access.

And it works! 

<pre>
<code>server2 $ ipmitool -I lanplus -H server1-ipmi -U root -P password sol activate
[SOL Session operational.  Use ~? for help]

Ubuntu 12.04.1 LTS server1 ttyS1

server1 login: 
</code>
</pre>

If we reboot the server we can see all the usual boot screens and can access them all without having to fire up the Java based console, instead I can just ssh into a second server that has access to the ipmi network and connect with ipmitool.

Here are some of the key mappings that will help to use ipmitool. The main thing to note is how to exit from the SOL session, and to do that you basically hit ENTER the a tilde, then a period. (For some reason this will usually log me not only out of the SOL session, but also the ssh session. Something to look into.)

<pre>
<code>server2 $ ipmitool -I lanplus -H 10.10.10.10 -U username -P password sol activate
[SOL Session operational.  Use ~? for help]
Here are some useful commands:
        KEY MAPPING FOR CONSOLE REDIRECTION:
         Use the <ESC><0> key sequence for <F10>
    	Use the <ESC><!> key sequence for <F11>
    	Use the <ESC><@> key sequence for <F12>
        Use the <ESC><Ctrl><M> key sequence for <Ctrl><M>
    	Use the <ESC><Ctrl><H> key sequence for <Ctrl><H>
    	Use the <ESC><Ctrl><I> key sequence for <Ctrl><I>
    	Use the <ESC><Ctrl><J> key sequence for <Ctrl><J>
        Use the <ESC><X><X> key sequence for <Alt><x>, where x is any letter
    	key, and X is the upper case of that key
        Use the <ESC><R><ESC><r><ESC><R> key sequence for <Ctrl><Alt><Del>
Help commands from ~?:
Supported escape sequences:
    	~.  - terminate connection
    	~^Z - suspend ipmitool
    	~^X - suspend ipmitool, but don't restore tty on restart
    	~B  - send break
    	~?  - this message
    	~~  - send the escape character by typing it twice
    	(Note that escapes are only recognized immediately after newline.)
</code>
</pre>

There are all kinds of things one can do with ipmitool other than terminal based BIOS access, and I won't list all of them here, but one nice thing is the ability to set the boot device, for example setting the boot device to disk and doing so for all future reboots.

<pre>
<code>server2 $ ipmitool -H 10.10.10.10 -U root -P password chassis bootdev disk \
options=persistent
</code>
</pre>

Here are the kernel configs I used to tell Ubuntu to use serial. Note that I added the "\" in the text below, so you can't just cut and paste it; needs to be all on one line.

<pre>
<code>pxeboot-server:/tftp/pxelinux.cfg# cat default 
# D-I config version 2.0
DEFAULT server
prompt 0
timeout 1
# serial console
console 0
serial 0 115200 0

LABEL server
kernel ubuntu-installer/amd64/linux
append ramdisk_size=14984 locale=en_US keyboard-configuration/layoutcode=us \
console-keymaps-at/keymap=us locale=en_US console-setup/layoutcode=en_US \
netcfg/wireless_wep= netcfg/choose_interface=eth0 netcfg/get_hostname=c01-07 \
url=http://10.10.10.10/node.preseed vga=normal \
initrd=ubuntu-installer/amd64/initrd.gz -- \
console=ttyS1,115200 earlyprint=serial,ttyS1,115200
</code>
</pre>

Finally I'll show a screenshot of a text install happening, and I am watching that via text-based SOL access.

![](https://raw.github.com/ccollicutt/ccollicutt.github.com/master/img/sol_install.png)

So give SOL a shot, and let me know if I've made any mistakes in this post, or if there are other interesting things that can be done with ipmitool. :)
