---
layout: post
title:  OpenBSD 6.9 on Protecli 6 Port Firewall
categories:
header_image: "/img/protecli2.jpg"
summary: "Time to implement some homelab network isoliation"

---

# {{ page.title }}

I've had a homelab for a while. And an old printer. And a wifi network. And a office network. Up until now it's been a bit of a free for all, everything was connected to everything else with no limitations or isolation. That's probably not great security-wise. Also...all this ransomware talk...it's concerning. Looks like it's time to implement some network isolation. So I bought a six port fanless firewall and put OpenBSD on it.

## Protecli 6 Port

![protecli box](/img/protecli-box.jpg)

The device was about $500, which is pricey. But I wanted 6 ports.

What I bought:

```
Protectli Vault 6 Port, Firewall Micro Appliance/Mini PC - Intel Dual Core, AES-NI, 4GB RAM, 32GB mSATA SSD 
```

What it has:

```
THE VAULT: Secure your network with a compact, fanless & silent firewall. Comes with US-based Support & 30-day money back guarantee!
CPU: Intel Dual Core Celeron 3865U, 64 bit, 1.8GHz, 2MB Smart Cache, Intel AES-NI hardware support
PORTS: 6x Intel Gigabit Ethernet NIC ports, 4x USB 3.0, 1x RJ-45 COM, 1x HDMI
COMPONENTS: 4GB DDR4 RAM, 32GB mSATA SSD
COMPATIBILITY: Firewalls tested with pfSense, untangle, OPNsense and other popular open-source software solutions.
```

The CPU isn't great, there are definitely better options, but it started to get to expensive. I think this CPU will be fine for my purposes.

## Install OpenBSD

I installed via the com port. I used a USB to serial adapter connected to my Linux workstation.

To connect: 

```
sudo screen /dev/ttyUSB0 115200
```

I downloaded the OpenBSD 6.9 img file and ``dd``ed it to a USB device.

Then plugged that device into the Protecli.

Next I set the BIOS in the Protecli to be "legacy only", othewise OpenBSD will give this error:

```
probing: pc0 com0 mem[352K 280K 2153M 83M 1M 1776M]
disk: hd0 hd1*
>> OpenBSD/amd64 BOOTX64 3.57
boot>
cannot open hd0a:/etc/random.seed: No such file or directory
booting hd0a:/6.9/amd64/bsd.rd: 3818189+1590272+3878376+0+704512 [109+288+28]=0x989530
entry point at 0x1001000
```

So get into the BIOS by pressing the `DEL` key when the box is booting up. Then change to "legacy only" and OpenBSD should boot.

Once OpenBSD boots up, enter the following to setup the com port.

```
stty com0 11500
set tty com0
```

From there you should be good to install. I just use all the defaults for now.

Here you can see all six ports:

```
# ifconfig | grep em
em0: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
em1: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
em2: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
em3: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
em4: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
em5: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
```

## Conclusion

I haven't put this into use yet, but I should soon. I'm assuming it's going to work fine, but I'll update this post after I've used it for a while.