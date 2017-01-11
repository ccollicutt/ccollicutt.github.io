---
layout: post
title: OpenDaylight Boron, OpenStack, and Networking-ODL
categories:
header_image: /img/odl.jpg
header_permalink: https://flic.kr/p/cHbryS
---

# {{ page.title }}

Software Defined Networking (SDN), like many other technical terms (*cough* devops *cough*) is often considered a panacea, but is also fraught with peril, etc, etc...you can see where I'm going, feel free to complete this thought on your own. 

Anyways, the point of this post is that I have recently been able to spend some time getting the OpenDaylight Boron release to work as the SDN controller for an OpenStack cloud, and I thought I would relay some of the issues I encountered as well as some of the required software and steps to complete the integration.

## tl;dr - Official Documentation

As I write this there is a [page](http://docs.opendaylight.org/en/latest/opendaylight-with-openstack/openstack-with-netvirt.html) that details how to integrate ODL Boron with OpenStack. 

I only encountered two issues with this documentation, the first is that the DLUX web GUI feature/plugin mentioned is incorrect and you would want the DLUX-core plugin. But that wouldn't affect actually using ODL, just access to a simple GUI.

Next, due to a bug in OpenStack which was fixed in master and [back-ported](https://review.openstack.org/#/c/403672/) to Newton, I needed to deploy Newton Neutron from the stable branch. I would imagine that that back-port will now be in most OS packaging though, so you probably won't have to worry about it, unless you are deploying a pre-Newton OpenStack. Not sure if it was back ported to release before Newton.

Otherwise, besides showing first deploying OpenStack and then deleting all the neutron state (not sure why all the examples do that), the documentation is pretty good and I'm sure will improve.

## This is Boron

Most of the blog posts and documentation you will find on the Internet are related to previous releases of OpenDaylight, and most of these instructions will only be about 75% correct. 

Boron has a new ODL feature, [Netvirt](https://wiki.opendaylight.org/view/NetVirt) which has taken over some of the functionality for supporting an OpenStack cloud. Essentially, instead of using the *odl-ovsdb-openstack* feature/plugin, you will use the *odl-netvirt-openstack* plugin.

Other than Netvirt and requiring Java 8, Boron seems to be quite similar from a systems administration perspective.

## Networking-ODL

[Networking-ODL](http://docs.openstack.org/developer/networking-odl/readme.html) is required code, perhaps driver is the right term, to allow the Neutron API to work with and OpenDaylight controller.

>OpenStack networking-odl is a library of drivers and plugins that integrates OpenStack Neutron API with OpenDaylight Backend. For example it has ML2 driver and L3 plugin to enable communication of OpenStack Neutron L2 and L3 resources API to OpenDayLight Backend.

It does not come with Neutron packaging at this time, and needs to be installed separately, either via Pip as most examples show or via source. Typically I will install from source using *stable/\<openstack version\>* just to get the latest code.

## Configuration Management of Open vSwitch

There are a couple of settings that need to be made to each Open vSwitch before it can be used by ODL. Typically this will be done with your configuration management tooling, ie. what ever is managing OpenStack's configuration.

The first is the "manager". Most documentation will show how to set that.

~~~bash
$ sudo ovs-vsctl show | head
7f7084b9-f0d8-449a-97fa-06476303d3cf
    Manager "tcp:10.15.0.190:6640"
        is_connected: true
    Bridge br-int
        Controller "tcp:10.15.0.190:6653"
            is_connected: true
        fail_mode: secure
        Port "tap35617dae-c8"
            Interface "tap35617dae-c8"
        Port "tun18897aa9c4e"
~~~

Above the manager is connected to the ODL controller.

Next is the *local_ip*.

~~~bash
$ sudo ovs-vsctl list Open_vSwitch | grep other_config
other_config        : {local_ip="10.15.0.11", provider_mappings="provider:bond1"}
~~~

That IP will become the endpoint for VXLAN tunnels.

Finally, as can be seen above as well, there is a *provider_mappings* key/value pair. This will list out the various Neutron networks and the physical interfaces they are supposed to use. This will be what floating IP or provider networks (ie. "external networks") will use. ODL will use the value to setup OVS on each node.

All three of these settings would be put in place by your config mgmt tool, eg. Ansible. :)

![ODL Netvirt Pipeline](/img/odl_netvirt_pipeline.jpg)

(Example of [ODL Netvirt Pipeline](https://docs.google.com/presentation/d/15h4ZjPxblI5Pz9VWIYnzfyRcQrXYxA1uUoqJsgA53KM/edit#slide=id.p))

## Issues

* The NATP component, currently at least, does not support ICMP. So if you have an instance on VXLAN and there is a router with an external gateway on on external network, ping, for example, won't work, but UDP and TCP will. I recieved the errors below on the ODL controller when pinging from an instance. Apparently it is because OpenFlow doesn't support ICMP on flows. This can be confusing while testing...it certainly confused me for a while.

~~~bash
2017-01-10 19:13:03,772 | ERROR | pool-21-thread-1 |
NaptPacketInHandler              | 345 -
org.opendaylight.netvirt.natservice-impl - 0.3.2.Boron-SR2 | Incoming
Packet is neither TCP or UDP packet
2017-01-10 19:13:03,772 | ERROR | pool-21-thread-1 |
NaptPacketInHandler              | 345 -
org.opendaylight.netvirt.natservice-impl - 0.3.2.Boron-SR2 | Incoming
Packet is neither TCP or UDP packet
~~~

* I wonder about having a tunnel from each host to every other host. Not sure how well that will scale. Something to look into.
* Like many other SDN systems, sometimes the initial setup of an unknown flow can be slow, but once it is in place on the OVS switch obviously it's faster. I've experienced this with Midokura's Midonet as well.

Otherwise everything seemed to work fine.

## Why OpenDaylight?

What does one get from deploying ODL? 

My opinion is that it provides the potential to perform fairly advanced network configurations that without an SDN controller would be difficult or impossible. Frankly at this state ODL + OpenStack is still being worked on. Most of the basic features are there, but where it really starts to get interesting is once we are past that stage, and want to do things like connecting tenants across multiple clouds, connecting to hardware VTEPs, things like that. It is going to take a while for all of this to become ready for production use.

There were a fair number of features delivered in ODL Boron (3):

* Merge of NetVirt and VPNService projects
* L2, L3
* Auto-bridge creation
* Auto-tunnel creation
* Floating IP’s
* VLAN and Flat provider network support for multiple internal and external networks
* Security Groups: Stateful using conntrack, Stateless, Learn (for OVS-DPDK)
* NAPT, SNAT
* IPv6
* Layer 2 Gateway

My thoughts on the above, as well as additional capability not mentioned:

* No more br-int/iptables. Ok, br-int still exists, but security groups are NOT done by iptables. This was something that always bugged me, and a lot of people, but it was a good solution. However, now, with ODL, iptables is taken out of the stack and security groups are done with OVS rules. This is very powerful and important, as it will "remove the many layers of bridges/ports required in iptable implementation." (1)
* ODL takes care of layer 3 with a "distributed virtual router" that is on par with Neutron DVR (2), though no ICMP with NATP.
* Ability to do Service Function Chaining (SFC). This functionality may come to "vanilla" Neutron, but most existing software that supports deploying SF chains will support ODL.

## What's next? 

* Troubleshooting ODL - This is not easy, as the paths through tables are somewhat convoluted, but the flow map above certainly helps.
* High Availability - ODL can be clustered, but I have not ventured into that area as of yet. Distributed systems are tough.
* [L2 Gateway](https://wiki.openstack.org/wiki/Neutron/L2-GW) - Wanting to get your VXLAN based workloads out into a physical network is an oft requested feature. It seems ODL Boron has the ability to use L2 Gateway. This needs to be explored.
* Service Function Chaining - This might not be as useful for organizations who are not interested in Network Function Virtualization (NFV), but that said, this is still quite powerful and useful in many "cloudy" situations. :)
* OVS-DPDK

## Footnotes

1. [OVSDB Netvirt](http://docs.opendaylight.org/en/stable-boron/user-guide/ovsdb-netvirt.html)
2. [OpenDaylight PDF](https://media.readthedocs.org/pdf/opendaylight/1.0.0/opendaylight.pdf)
3. [NetVirt Basic Tutorial](http://schd.ws/hosted_files/opendaylightsummit2016/c9/ODL%20Summit%202016%20NetVirt%20Basic%20Tutorial%20(2).pdf)
