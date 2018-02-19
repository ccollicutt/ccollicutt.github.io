---
layout: post
title: Juniper vSRX MPLS Lab
categories:
header_image: /img/mpls-juniper.jpg
---

*(network diagram of the lab)*

# {{ page.title }}

I've wrote up a [lab](https://github.com/ccollicutt/mpls-networking/blob/master/JUNIPER-MPLS.md) (available on github) on deploying a MPLS core network using Juniper vSRX routers. It's completely based on [this](https://juniperlabs.wordpress.com/2014/01/16/simple-juniper-mpls-core-with-l3vpn/) blog post, entitled "Simple Juniper MPLS Core."

My goal with replicating the simple MPLS network was to try to get a better understanding of how MPLS will work in NFV and 5G where there is a desire to do "network slicing."

I'm not sure who wrote the initial blog post, but I am thankful they did because I was able to completely replicate it. I'm indebted to them because I feel like I now have a better understanding of Juniper and MPLS networks, which was my goal. This lab provides a base that I can build off of.

I don't want to say much more in this blog post because I put a lot of work into the lab, so please take a look at the [github repo](https://github.com/ccollicutt/mpls-networking) where it resides.

Here's a listing of all the virtual routers running an a baremetal KVM node.

```
$ virsh list
 Id    Name                           State
----------------------------------------------------
 86    ce-ny                          running
 120   pe-lo4                         running
 122   p2-njh                         running
 123   p1-ny8                         running
 124   pe-nj2                         running
 125   ce-uk                          running
```

Here's a session of ce-ny pinging/tracerouting ce-uk.

```
$ ssh root@ce-ny
Password:
--- JUNOS 17.3R1.10 built 2017-08-23 06:47:03 UTC
root@ce-ny%
root@ce-ny% cli
root@ce-ny> ping 6.6.6.6
PING 6.6.6.6 (6.6.6.6): 56 data bytes
64 bytes from 6.6.6.6: icmp_seq=0 ttl=61 time=6.536 ms
64 bytes from 6.6.6.6: icmp_seq=1 ttl=61 time=3.840 ms
^C
--- 6.6.6.6 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max/stddev = 3.840/5.188/6.536/1.348 ms

root@ce-ny> traceroute 6.6.6.6
traceroute to 6.6.6.6 (6.6.6.6), 30 hops max, 40 byte packets
 1  10.3.5.0 (10.3.5.0)  4.759 ms  4.345 ms  4.514 ms
 2  10.2.3.0 (10.2.3.0)  6.033 ms  6.077 ms  5.291 ms
     MPLS Label=299776 CoS=0 TTL=1 S=0
     MPLS Label=299776 CoS=0 TTL=1 S=1
 3  10.2.4.1 (10.2.4.1)  6.693 ms  6.012 ms  8.049 ms
     MPLS Label=299776 CoS=0 TTL=1 S=1
 4  6.6.6.6 (6.6.6.6)  4.826 ms  5.409 ms  5.222 ms

root@ce-ny>
```

It works! :)

*NOTE: On the diagram each router has a .17x IP address (full address is the management network, 192.168.122.17x). The x for each is also their router ID and loopback address. So ce-uk, .176, is 6.6.6.6.*

## Conclusion

I'm really happy with how this worked out, and as mentioned, I now have a base to do more exploration around MPLS, BGP, OSPF and other protocols, hopefully ending with me having a good understanding of what network slicing could or should look like in a service provider.

One of the major things I learned is that building a virtual lab like this is really about understanding nodes and edges, just like what one would do with `graphviz` dot diagrams. This is why I've listed the "links" (aka edges) and interfaces on the diagram, to help me automate and validate. Building a network diagram like the above should be completely automatable, based on well understood graphing. But that is another story... :)
