---
layout: post
title: Install a Linux Rootkit to Test Security Systems With
categories:
header_image: "/img/roots.jpg"
header_permalink: "https://unsplash.com/photos/hW11fwjzVfA"

---

# {{ page.title }}

Let's say you wnat to install a rootkit to test with. There could be various reasons, maybe you are testing some kind of specialized security system. Regardless of the reason, you need a rootkit to test out. 

The first place I ended up at is [Awesome Rootkits](https://github.com/milabs/awesome-linux-rootkits), a big list of rootkits.

The one that I picked from that list, somewhat randomly, was [LilyOfTheValley](https://github.com/En14c/LilyOfTheValley). I have no good reason as to why I picked this particular one, other than it seemed to suggest it would work on several kernel versions, and was a small amount of code.

## WARNING

Obviously this is a rootkit. Don't install it on your local machine. Get a temporary, disposable virtual machine. Also, please review the code in the rootkit files. You've been warned!

## Install

The kernel version in this example is 4.4.0-22 on Ubuntu 16.04.

```
# uname -a
Linux old-kernely 4.4.0-22-generic #40-Ubuntu SMP Thu May 12 22:03:46 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```

Clone the rootkit repo.

```
# git clone https://github.com/En14c/LilyOfTheValley
# cd LilyOfTheValley
```

Install build-essential.

```
# apt install build-essential -y
```

Get headers too.

```
# apt install linux-headers-4.4.0-22-generic
```

Make.

```
# make
make -C /lib/modules/4.4.0-22-generic/build M=/root/LilyOfTheValley modules
make[1]: Entering directory '/usr/src/linux-headers-4.4.0-22-generic'
  CC [M]  /root/LilyOfTheValley/lilyofthevalley.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /root/LilyOfTheValley/lilyofthevalley.mod.o
  LD [M]  /root/LilyOfTheValley/lilyofthevalley.ko
make[1]: Leaving directory '/usr/src/linux-headers-4.4.0-22-generic'
```

Now there is a kernel module, lilyofthevalley.ko.

```
# ls -1
client.c
lilyofthevalley.c
lilyofthevalley.ko
lilyofthevalley.mod.c
lilyofthevalley.mod.o
lilyofthevalley.o
Makefile
modules.order
Module.symvers
README.md
```

Install that module.

```
# insmod lilyofthevalley.ko 
```

Once it's loaded /proc can be used to find commands that can be sent to the module.

```
# cat /proc/lilyofthevalleyr00tkit 
###########################
LilyOfTheValley Commands
###########################

	* [givemerootprivileges] -->> to gain root access
	* [hidepidPID] -->> to hide a given pid. replace (PID) with target pid
	* [unhidepidPID] -->> to unhide a given pid. replace (PID) with target pid
	* [hidingfiles] -->> just prepend lilyofthevalley to the file or dir name that u want to hide
```

Also, note how any files in the directory that started with the string "lilyofthevalley" now no longer appear in ls.

```
# ls -1
client.c
Makefile
modules.order
Module.symvers
README.md
```

This is because the rootkit makes those files "invisble."

See line 409 of lilyofthevalley.c.

```
	/*
	hide any file in  the root filesystem, 
	if first chars of it's name == r00tkit_name
	*/
	if (strncmp(name,R00TKIT_NAME,R00TKIT_NAMELEN) == 0)
		return 0;
```

Create a file that starts with that particular string. Note how it's not visible in ls.

```
# touch lilyofthevalley.txt
# ls
client.c  Makefile  modules.order  Module.symvers  README.md
```

After rebooting the node the kernel module is no longer loaded and the missing files are again visible..

```
# ls -1
client.c
lilyofthevalley.c
lilyofthevalley.ko
lilyofthevalley.mod.c
lilyofthevalley.mod.o
lilyofthevalley.o
lilyofthevalley.txt
Makefile
modules.order
Module.symvers
README.md
```

With the module in place secuirty systems can be tested to see if they can find it.