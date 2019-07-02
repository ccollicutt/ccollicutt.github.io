---
layout: post
title: Mikrotik RB32011 Won't Reset to Default Configuration
categories:
header_image: "/img/rb2011.png"

---

# {{ page.title }}

## tl;dr

If you can't get your RB32011 router to go back to default configuration, ie. it just comes up with a blank configuration, you'll likely need to netinstall it, which wipes everything.

## Mikrotik

I have an old [Mikrotik RB2011](https://mikrotik.com/product/RB2011UiAS-2HnD-IN) and it's been sitting unused in a box for several years. But I'm back on my Mikrotik thing as I'm wiring up my house with more than one WIFI device. I don't know if this will actually improve my home WIFI, but I'm doing it anyways. 

I like Mikrotik because they support MPLS and I like to pretend I know what I'm doing. Not that I'll use MPLS for a wireless network, but I like the possibility of using it...even if I don't.

Mikrotik was in the [news](https://www.zdnet.com/article/thousands-of-mikrotik-routers-are-snooping-on-user-traffic/) last year for some security issues, which I have not paid enough attention to. 

## Default Configuration

As I try to figure out how to configure CAPsMAN, Mikrotik's multi AP management system, I needed to put my RB2011 back into factory default configuration more than once (as I make mistakes and lose access to the router). But, after an upgrade to the OS and a reset, it wouldn't come back up with the default settings, rather it boots with a completely blank configuration, which makes it challenging to work with as you can't access it easily over the network (of course, the serial console works). I only found one [post](https://forum.mikrotik.com/viewtopic.php?t=78638) about it and it didn't help.

At any rate, my assumption is the old default script stored in the device wasn't compatible with the new OS version. Unfortunately I didn't keep a copy of the old default-config script, which you can view with:

```
/system default-configuration print
```

Believe me that the most recent default config file, the one that exists after a netinstall, is much different than the original one I had on my RB2011 with the older OS.

To ge the new script installed I had to reinstall via NetInstall, which actually worked. I had to use my Windows computer, but I was able to reinstall in less than 10 minutes, which is nice.

So, after netinstalling, I could again run the reset command.

```
/system reset-configuration
```

Once the node came back up, it has this default config.

## PS. Serial Access

Also, serial access works just fine using a USB serial console device and cable to the back of the RB2011.

```
sudo screen /dev/ttyUSB0 115200
```