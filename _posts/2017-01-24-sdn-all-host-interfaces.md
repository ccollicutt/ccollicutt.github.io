---
layout: post
title: SDN on All Interfaces 
categories:
header_image: /img/octopus.jpg
header_permalink: https://flic.kr/p/ePzuj9 
---

# {{ page.title }}

Recently I have been involved in some discussions surrounding network segmentation, specifically around the concept of PVLAN...private VLAN. Note that this is different from the idea of a private network, or a tenant network in a virtualization system, and is instead a form of network isolation, and I believe is described in [RFC 5517](https://tools.ietf.org/html/rfc5517).

>Private VLAN, also known as port isolation, is a technique in computer networking where a VLAN contains switch ports that are restricted such that they can only communicate with a given "uplink". The restricted ports are called "private ports". Each private VLAN typically contains many private ports, and a single uplink. The uplink will typically be a port (or link aggregation group) connected to a router, firewall, server, provider network, or similar central resource. - [ Wikipedia](https://en.wikipedia.org/wiki/Private_VLAN)

PVLAN is based on physical switch ports, and only works on access ports, not trunk ports. This is a problem in environments where we have bonds on hosts that are accessing multiple VLANs and, of course, virtual switches in a virtualization environment. (That said, some distributed virtual switches support PVLANs, but not OVS which is common in most open source virtualization systems, such as OpenStack.)

## Micro-segmentation

What we are really talking about, as far as I'm concerned, is micro-segmentation. This concept has become popular recently, mostly due to a desire for *zero-trust networks* and the proliferation of containers. 

One way to deploy containers is to give them all an IP, be it IPv4 or IPv6, and to not be concerned as to what IP they get, and where this IP is actually located in the datacenter. This suggests that we need powerful network policies to control what these containers need to connect to, and thus we get micro-segmentation. Frankly I'm not sure how [layer 2 networks](http://blog.ipspace.net/2014/06/why-is-ipv6-layer-2-security-so-complex.html) make sense in a situation where we have zero-trust, but if it does, then the virtualization hosts will have to have their interfaces, especially virtual switches, controlled by SDN, or perhaps some kind of centralized firewall management (where firewall means managing flows, not blocking ports).

See [this fascinating Facebook presentation](https://www.nanog.org/sites/default/files/20161018_Lapukhov_Internet-Scale_Virtual_Networking_v1.pdf) on *Internet-scale Virtual Networking Using Identifier-Locator Addressing* for some mindblowing ideas. *cough* IPv6 *cough*. There is a [video](https://www.youtube.com/watch?v=xy3DPkwcbvA) as well.

## Zero-trust Networks

East/west traffic has grown considerably. 

>The threat from inside is bigger than ever before and is further exacerbated by the fact that around 80 percent of traffic in data centers is now of east-west nature - [SDX Central](https://www.sdxcentral.com/articles/contributed/data-center-security-shiv-agarwal/2016/02/)

Organizations of all kinds are getting cracked left and right, or...er...east and west. Clearly security is broken, most often because we build networks that are secure on the perimeter but wide open internally (also humans make mistakes and write software). 

>The Zero Trust Model is simple: cybersecurity professionals must stop trusting packets as if they were people. Instead, they must eliminate the idea of a trusted network (usually the internal network) and an untrusted network (external networks). In Zero Trust, all network traffic is untrusted. - [Forrester](http://csrc.nist.gov/cyberframework/rfi_comments/040813_forrester_research.pdf)

I believe Ronald Reagan popularized the phrase "trust but verify" which unfortunately has become a common refrain, and one I have never liked. Clearly we cannot automatically trust any network activity, and frankly I don't know how much network verification has ever occurred.

Some organizations use PVLAN to try to deter lateral movement of bad actors, but in a modern, virtualized, or even worse, containerized, datacenter PVLAN is of no use. So we need another technology to achieve zero trust.

## Separation of Duties

Clearly one area PVLAN may have an edge is that it is performed by physical network hardware. If micro-segmentation is managed by the local vswitch on a physical host, or (ugh) firewall rules in a VM, then there is not as much separation of duty. Further these systems are typically managed out of band of the network (though things like routing protocols would not be).

That said, virtualization is so important that it likely cancels out any security issues; it's just too useful. Even [OpenBSD is getting a hypervisor](https://www.openbsd.org/papers/asiabsdcon2016-vmm-slides.pdf). If we are going to do large scale virtualization, ie. IaaS, then we have no choice but to trust the hypervisors, and thus the vswitches. If you work in a place that believes hypervisor breakouts are too risky, then I would imagine your infrastructure is quite different than most, and you have unique requirements.

Further, it could be that the fact that *all* hypervisors are secured via micro-segmentation is helpful should some of them become compromised. Unless the centralized "micro-segmentation" provider is also compromised, then all of the other micro-segmentation rules would still be in place, making it hard to hop from one compromised node to the other because the security polices are highly distributed and exist in some fashion on all nodes at all times.

## Routing on the Host

I want to mention "routing on the host" because it is a model that is becoming more popular, though still not that common. The idea is to allow hosts to announce what IPs they own, ie. typically become BGP speakers. This certainly comes with its own set of issues and benefits, which I won't get into here. Certainly this model would have an effect on implementing micro-segmentation.

Some network companies, such as Cumulus Linux, are making routing on the host easier. I should note that I am a big fan of Cumulus, though no one I've ever worked with seems to like them. Cumulus, at least started out, trying to make [routing on the host](https://cumulusnetworks.com/routing-on-the-host/) more commonplace. But I am not sure how well they have succeeded. They have gone so far as to create a custom [Quagga](https://github.com/CumulusNetworks/quagga) package to enable routing on the host.

[Project Calico](https://www.projectcalico.org/) is similar:

>Based on the same scalable IP network principles as the Internet, Calico leverages the existing Linux kernel forwarding engine without the need for virtual switches or overlays. Each host propagates workload reachability information (routes) to the rest of the data center – either directly in small scale deployments or via infrastructure route reflectors to reach Internet level scales in large deployments.

Calico seems to be picking up steam regarding Kubernetes (k8s), where k8s has made the major architectural decision to give each container its own IP.

![https://flic.kr/p/aiLeiD](/img/cosmic.jpg)

## Virtual Switches are Part of Your Network Infrastructure

I think if there is a valuable idea in this post it is to consider virtual switches as part of your **network** infrastructure. For example in a small OpenStack deployment you might have four physical switches, but then you will have one virtual switch for every OpenStack Neutron DHCP/L3 server and of course all the compute nodes. That adds up to a lot of switches. If you have 100 hypervisors then you have 100+ switches.

These vswitches can break--they can get hit by cosmic rays, be attacked by viral switches (I made that up) and run into all kinds of other issues.

## Interfaces (and Bonds) are Part of Your Network Infrastructure

Hosts often have bonds that are configured to accept multiple VLANs. We have Link Layer Discovery Protocol (LLDP) to help with this, and to show that the interfaces and bonds are really part of the network infrastructure. Perhaps OVS sits on top of the bonds. Perhaps the bonds are only used for underlying IaaS infrastructure. Perhaps the bonds are Linux bonds or they are OVS bonds or both types exist. 
I have not tried as of yet, but apparently OVS supports [LLDP](http://www.dorm.org/blog/finally-a-workaround-for-lldp-with-open-vswitch/). 

Presumably, in a post-VLAN micro-segmentation enabled world, we would need something like OVS to control all interfaces in a physical host, even ones that aren't supporting virtual machines. Linux bridge isn't going to cut it. Don't get me started on [MLAG](http://blog.ipspace.net/2017/01/never-take-two-chronometers-to-sea.html)... :)

## Open vSwitch is Pretty Important

Does Open vSwitch support PVLANS? Not right now. Perhaps if OVS did then this whole thing would not even be a question. But to me, even if it did, how would it do it? It would be done with flows. Which you can do now. Further, as discussed in this blog post, in my opinion PVLAN is a heavy handed way to try to enable micro-segmentation, meaning that if we are to do micro-segmentation with OVS then it would likely be considerably more sophisticated than simply emulating PVLAN.

In the case of IaaS, virtual switches are in effect distributed and are controlled by a centralized system (for example OpenStack Neutron) or with an SDN controller (perhaps in conjunction with Neutron). If you have hundreds of OVS instances, then they need to be centrally managed. It's not enough to push a one-time configuration into each vswitch as obviously workloads are being constantly created and destroyed or (yuck) moved.

## Potential Micro-segmentation Solutions

### Security Groups for Tenant Workloads

One potential solution to micro-segmentation is firewall rules on the host. Or, if some form of segmentation is required in tenant workloads in an IaaS system, then usually they support security groups. 

Generally speaking, security groups in IaaS are used to specifically allow communication on a layer 2 network, just like any normal network would do, and are typically used to stop external layer 3 ingress connections. Usually the default is to allow all traffic on the local network, which makes sense in most cases. Depending on their implementation they may be able to implement a form of micro-segmentation but occasionally security group functionality is somewhat limited.

### Host-based Firewalls

For physical hosts, we could setup Iptables rules as a way to achieve micro-segmentation. Perhaps some 3rd party system would manage and deploy Iptables rules. It could be as simple as configuration management tooling, such as Ansible, but the question is what happens on changes, and what is the "single source of truth" for the config tooling to access? For example if a container is created, and it can come up with any IP on any hypervisor, then how will this 3rd party Iptables configurator know what to do and when to do it, unless that system is what is managing the life cycle of containers.

However, even without containers, things start to get rapidly ugly, even in a non-hypervisor environment, basically around heterogeneous systems, where there is a lot of variation in things like operating systems. Managing firewall rules on multiple platforms will not be easy.

Also, Iptables is global and you will take a performance hit.

>There is no concept of per-interface rules in iptables, iptables rules are global. This means that for every packet, incoming interface needs to be checked and execution of rules branched to the set of rules appropriate for the particular interface. This means linear search using interface name matches which is costly, especially with a high number of VMs. - [RedHat](http://redhatstackblog.redhat.com/2016/07/22/how-connection-tracking-in-open-vswitch-helps-openstack-performance/)

### Firewall Vendors, Service Function Chaining

Firewalls will probably never go away. I wish they would...all these middleboxes getting in the way, slowing things down, costing money, etc. Typically firewalls just have two sides: inside and outside. With zero-trust and micro-segregation we are trying to get rid of that false dichotomy.

Firewall vendors love the idea of micro-segmentation though. Palo Alto is a popular firewall provider; they have managed to step into the role that Checkpoint used to fill. They have a neat [demo](https://www.youtube.com/watch?v=68zhDttmnIc&t=1341s) using service function chaining and OpenStack to dynamically insert a firewall into a service chain. Very interesting and quite powerful, but also resource intensive. We can't insert an entire firewall into every flow on the network or drop one in front of every container...maybe one on every host? If the firewall is on every host, then it might as well be the vswitch. Perhaps companies like Palo Alto could either write their own OVS compatible vswitch or there is some kind of a plugin system.

We likely need something less resource intensive. While SFC is an important concept for providing tenant security and functionality, it is, however, not as useful for securing the underlying infrastructure, ie. it is an NFV technology as opposed to a datacenter security technology. That said in situations where attacks are detected inserting a powerful firewall or IDS may be valuable even in the DC.


### Firewalls in the NICs

We are likely to see a surge in specialized hardware, partially due to the slowdown in Moore's law. This could include having "smart NICs." VXLAN offloading is common, so why not flows as well. 

[Netronome](https://www.netronome.com/products/agilio-cx/) has some interesting products, though I have not used them. And of course now we'd be relying on firmware which is notoriously painful. However, this could be beneficial in that it is potentially separate(ish) hardware. But that said, I'm not sure how they are programmed or what their control plane looks like...perhaps from the host. :0

>Compounding the problem is heightened security, which is moving from a firewall on the perimeter of the network to policy-based security with network rules governing access to virtual machines and now software containers. Das says that it is common to require 1,000 security policy rules per VM in clouds these days, which works out to 48,000 rules for a two-socket server with a total of 48 VMs, and moreover that at some infrastructure running inside of the datacenters in China, a single server might have to juggle as many as 1 million – yes, more than a factor of 20X more – security policy rules. Shifting from VMs to containers, where a single server might have thousands of containers and yet a large number of security policy rules, will make the situation even worse. - [The Next Platform](https://www.nextplatform.com/2016/01/27/offloading-the-network-like-a-hyperscaler/)

1000 rules per VM sounds crazy. But maybe.

If we could get higher performance with specialized NICs, regain CPU cores, and still have the ability to easily update and install software on them, then this might be a good way to go about obtaining micro-segmentation.

### Software Defined Networking

My thesis is that if we want micro-segmentation, then they way to do this is in the network using some form of SDN which manages the local virtual switch as well as the underlying physical infrastructure, and hopefully it would utilize IPv6. (Surprisingly IPv6 is probably the hardest part, because few seem to support it.) 

The prior solutions, mostly utilizing firewall rules in some fashion, were really just ideas/brainstorming if a true SDN solution is not possible for whatever reason.

I don't want to get into what SDN systems might be able to meet these requirements in this post, so for now I will just leave it as a statement to come back to in future writing, especially once I have done some work in the lab.

## Various Paradoxes, Twiddling with VLANs, and Let's Do as We Say Too

Typically the underlying IaaS infrastructure will be deployed just like we have deployed "enterprise" systems for the last 20 years (minus virtualization). VLANS. Bonds. MLAG. Middleboxes. Manual configuration. Servers or server pairs that can never be down. Hand crafted and bug ridden. Expensive. Instance high availability. Time consuming. Waterfall. Complicated. Tribal knowledge. Security choke points. Hard-candy coating with soft milk chocolate inside. On and on.

We put IaaS on top of this old-school enterprise architecture and deployment and force the tenants to do "cloud native." It just doesn't seem good or fair to me. If anything this is why private clouds fail. Why do IaaS if it is just going to look like VMWare deployments of the last decade or two? Most of the time the answers are circular: we deploy it this way because nobody else is doing it the other way, so we deploy it this way. See IPv6.

PVLAN is a great example of VLAN twiddling. In an era where security is failing, we really need to inhibit malware  and lateral movement in DCs where east/west traffic is growing. Thus, we need to push SDN--not Iptables rules--all the way to host interfaces in order to enable micro-segmentation...somehow...

I'm not sure if anyone will actually read this post. I'm sorry it's kind of long (I left some things out, honestly, like Intent Based Networking for example) but I find this area is fascinating and there are many options. It's not a solved space. We really need to do something about at least two things 1) enabling zero-trust networks and 2) not continuing to deploy underlying IaaS infrastructure the way we have for the last 20 years: we need to deploy the systems that support the IaaS using cloud native designs as well.

Certainly I mention containers a lot in this post, and that makes it easy to dismiss much of what I'm saying because how many organizations are really going to run containers and be successful at it. Not many. Most organizations will still have a majority of workloads that will not fit into the container paradigm. However, they will certainly be using virtualization (really are they that different), and in some cases perhaps be looking at a technology like OpenStack, and in that situation it would be completely possible to work on enabling zero-trust networks and micro-segmentation, likely through and SDN solution. My thesis is that that SDN should not just live only in OpenStack, but should also be part of the underlying IaaS infrastructure. I think that is completely possible. Further to that, it is likely, if not inevitable, and if not already, that the OpenStack control plane will be containerized, meaning that if you run OpenStack you will be running containers, whether you want to or not.
