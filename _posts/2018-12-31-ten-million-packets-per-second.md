---
layout: post
title: Ten Million Packets per Second with Moongen
categories:
header_image: "/img/moon.jpg"
header_permalink: "https://unsplash.com/photos/VIcTzkzNZR8"

---

# {{ page.title }}

## Network Performance Testing

Over the last few years I have been doing a lot of work in the Network Function Virtualization (NFV) world, specifically Virtual Network Function (VNF) on-boarding. When on-boarding any application, it's important to test it, and one part of testing is performance testing. But how do we effectively performance test network applications?

We need to push performance to the breaking point. Many of these applications are designed to be extremely fast, thus it's difficult to even push them to their limits without some kind of specialized traffic generator. Further, traffic generators themselves are just applications, and have the same issues getting high performance as the systems that are under test. Commercial traffic generators available but are typically esoteric, expensive, and inflexible.

## How Much of Our Network Do We Really Use?

I'm going to make a statement that is perhaps controversial: we don't really use much of our networks...they're rarely pushed to their limits. In fact, I think we build many of our systems assuming that the network will hardly be used. Frankly I think it would be fair to apply this to much of what we do: compute, network, and storage. This is in part why virtualization is so successful. But, as usual, I digress...

We should push our networks, and more specifically our network interfaces and the applications that create and manage packets, to the limit. Throughput is relatively easy, packets per second, however, is a bit harder--a good 10GB interface should be able to send about 10 million 64 byte packets per second (PPS). That's what hardware can do, but what about software?

## The Linux Kernel and DPDK

The Linux Kernel, and presumably all other OS kernels, just can't deal with that many packets. Here's what [CloudFlare](https://blog.cloudflare.com/kernel-bypass/) has to say:

>Unfortunately the speed of vanilla Linux kernel networking is not sufficient for more specialized workloads. For example, here at CloudFlare, we are constantly dealing with large packet floods. Vanilla Linux can do only about 1M pps. This is not enough in our environment, especially since the network cards are capable of handling a much higher throughput. Modern 10Gbps NIC's can usually process at least 10M pps.

This performance issue is well known and usually doesn't come into play in most networks and applications. However, when we need a high level of performance we can get around the kernel with something like DPDK, where we pull most the networking into userland. While the Data Plane Development Kit (DPDK) is available to allow vastly increased performance, it still requires some complex programming...unless you use Moongen!

## Moongen

[Moongen](https://github.com/emmericp/MoonGen) is truly an amazing--and more importantly accessible--piece of software for generating extreme amounts of traffic.  

>MoonGen is a fully scriptable high-speed packet generator built on DPDK and LuaJIT. It can saturate a 10 Gbit/s connection with 64 byte packets on a single CPU core while executing user-provided Lua scripts for each packet. Multi-core support allows for even higher rates. It also features precise and accurate timestamping and rate control.

Using Moongen one can access DPDK though simply Lua scripts. I was able to easily create a custom DNS UDP packet generator in less than a hundred lines of Lua. Using that code I could push about 6.5GB of DNS queries towards a DNS server to test its performance capability, all using an older x86 server with a common, and inexpensive 10GB Intel 82599ES network card. 

A quick example of running that code:

```
# ./build/MoonGen ./examples/dns-query.lua -r 10000
[INFO]  Initializing DPDK. This will take a few seconds...
EAL: Detected 16 lcore(s)
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
EAL: PCI device 0000:03:00.0 on NUMA socket 0
EAL:   probe driver: 8086:1521 net_e1000_igb
EAL: PCI device 0000:03:00.1 on NUMA socket 0
EAL:   probe driver: 8086:1521 net_e1000_igb
EAL: PCI device 0000:81:00.0 on NUMA socket 1
EAL:   probe driver: 8086:10fb net_ixgbe
EAL: PCI device 0000:81:00.1 on NUMA socket 1
EAL:   probe driver: 8086:10fb net_ixgbe
[INFO]  Found 3 usable devices:
   Device 0: 0C:C4:7A:92:63:7D (Intel Corporation I350 Gigabit Network Connection)
   Device 1: 0C:C4:7A:BB:70:82 (Intel Corporation 82599ES 10-Gigabit SFI/SFP+ Network Connection)
   Device 2: 0C:C4:7A:BB:70:83 (Intel Corporation 82599ES 10-Gigabit SFI/SFP+ Network Connection)
[WARN]  device.config() without named arguments is deprecated and will be removed. See documentation for libmoons device.config().
PMD: ixgbe_dev_link_status_print(): Port 1: Link Up - speed 0 Mbps - half-duplex
[INFO]  Waiting for devices to come up...
[INFO]  Device 1 (0C:C4:7A:BB:70:82) is up: 10000 MBit/s
[INFO]  1 device is up. 
starting arp 
starting send
starting counter
in send
in counter
[INFO]  starting in moongen loop
[nil] TX: 5.48 Mpps, 4562 Mbit/s (5439 Mbit/s with framing)
[nil] TX: 6.56 Mpps, 5455 Mbit/s (6504 Mbit/s with framing)
[nil] TX: 6.73 Mpps, 5601 Mbit/s (6679 Mbit/s with framing)
[nil] TX: 6.74 Mpps, 5606 Mbit/s (6684 Mbit/s with framing)
[nil] TX: 6.77 Mpps, 5636 Mbit/s (6719 Mbit/s with framing)
[nil] TX: 6.77 Mpps, 5631 Mbit/s (6714 Mbit/s with framing)
[nil] TX: 6.79 Mpps, 5645 Mbit/s (6731 Mbit/s with framing)
[nil] TX: 6.77 Mpps, 5633 Mbit/s (6717 Mbit/s with framing)
[nil] TX: 6.76 Mpps, 5624 Mbit/s (6706 Mbit/s with framing)
[nil] TX: 6.79 Mpps, 5647 Mbit/s (6733 Mbit/s with framing)
[nil] TX: 6.81 Mpps, 5664 Mbit/s (6753 Mbit/s with framing)
SNIP!
```

<br />

Pretty incredible how easy it is to write a little bit of Lua and generate (or process) such a huge number of packets! One server. One NIC. One CPU. 6Mpps of traffic.

## Use All Your Network (With Much Less CPU)

What I found was that very few network applications, including the Linux kernel, can deal with that much traffic. In my simplistic testing it was easy to get the Linux kernel and network applications to drop packets by the millions, or get applications to just plain fall over and either crash or stop processing packets (looking at you named). Ultimately, dealing with this many packets is difficult, and a well known problem, but using things like DPDK and Moongen makes it much more accessible to solve.

In situations that call for high performance using considerably fewer CPU resources, DPDK, possibly with Moongen, becomes an easy choice, and that goes double for performance testing with a relatively limited budget.

## Thanks Moongen (and CloudFlare too)

Moongen is truly an amazing piece of software. Thanks to the relatively few people who put this project together, especially Mr. Paul Emmerich.

I also want to give a shout out to the CloudFlare folks. I don't know if they know how important their network performance blog posts are. I also used some of their example code. Thanks greatly!