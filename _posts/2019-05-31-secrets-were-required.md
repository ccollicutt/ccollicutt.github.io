---
layout: post
title: WIFI - Error Connection activation failed, Secrets were required, but not provided.
categories:

---

# {{ page.title }}

Prior to a presentation, I was trying to connect to a Cisco Meraki based wireless network with a Lenovo Gen3 X1 Fedora 30 laptop, and kept getting this error:

```
$ nmcli dev wifi connect <SSID> password <password>
Error: Connection activation failed: (7) Secrets were required, but not provided.
```

Obviously I provided the password. I was totally stumped. Then I saw [this](https://unix.stackexchange.com/questions/420640/unable-to-connect-to-any-wifi-with-networkmanager-due-to-error-secrets-were-req/444433) Stackexchange post and decided to give it a try.

This is my wireless device:

```
$ lspci -vv -s 04:00.0
04:00.0 Network controller: Intel Corporation Wireless 7265 (rev 59)
	Subsystem: Intel Corporation Dual Band Wireless-AC 7265
	Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+
	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
	Latency: 0, Cache Line Size: 64 bytes
	Interrupt: pin A routed to IRQ 49
	Region 0: Memory at f1000000 (64-bit, non-prefetchable) [size=8K]
	Capabilities: <access denied>
	Kernel driver in use: iwlwifi
	Kernel modules: iwlwifi
```

I should note that I've connected to several WIFI networks since installing Fedora 30 and have had no issues. Just this one Meraki based network so far.

I added the below:

```
$ tail -2 /etc/NetworkManager/NetworkManager.conf
[device]
 wifi.scan-rand-mac-address=no
```

And restarted NetworkManager, and lo behold was able to connect to the WIFI. I am not sure why this works. The MAC address randomization is a good thing to have security/privacy-wise. I'd prefer that it was enabled. But I really do need to connect to WIFI as easily as possible. This caused me major issues as this cropped up prior to a presentation (ugh). Previously I was on Fedora 26 for a (too) long time. Presumably this feature is new. Or it is some other related bug.

However, that said, I don't know why this would cause the problem.

* Related [bug report](https://bugs.launchpad.net/ubuntu/+source/network-manager/+bug/1681513)
* Related [post](https://askubuntu.com/questions/902992/ubuntu-gnome-17-04-wi-fi-not-working-mac-address-keeps-changing)
* Article on [WIFI MAC randomization](https://arstechnica.com/gadgets/2014/06/ios8-to-stymie-trackers-and-marketers-with-mac-address-randomization/)

The bug report seems to be related to USB WIFI devices but there are a few onboard as well.

The reality is turning off the randomization allowed me to connect. *sigh*
