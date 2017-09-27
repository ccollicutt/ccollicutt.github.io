---
layout: post
title: Forms of Service Functioning Chaining and a BigSwitch Example
categories:
header_image: /img/chain-unsplash.jpg
header_permalink: https://unsplash.com/photos/gtVrejEGdmM
---

# {{ page.title }}

Service Function Chaining is an important concept in Network Function Virtualization (NFV). But it is also a powerful tool in more generic Software Defined Networking (SDN) as well.

I quite like the definition the [OpenStack Networking SFC](https://wiki.openstack.org/wiki/Neutron/ServiceInsertionAndChaining) project has/had:

>Service Function Chaining is a mechanism for overriding the basic destination based forwarding that is typical of IP networks. It is conceptually related to Policy Based Routing in physical networks but it is typically thought of as a Software Defined Networking technology. It is often used in conjunction with security functions although it may be used for a broader range of features. Fundamentally SFC is the ability to cause network packet flows to route through a network via a path other than the one that would be chosen by routing table lookups on the packet's destination IP address. It is most commonly used in conjunction with Network Function Virtualization when recreating in a virtual environment a series of network functions that would have traditionally been implemented as a collection of physical network devices connected in series by cables.

Essentially, at a high level, what we want to do is replicate several physical network devices connected together in a chain, except do it virtually. A simple example would be a network gateway, firewall, and intrusion detection system, all physical and all connected to one another through direct connections, potentially without even a switch involved, except, and this is important, it's not done at layer 3, it's done at layer 1 as a port to port setup.

I have been working with SFC for a while and have found that there are three major types:

1. Layer 3 SFC - Which isn't what I would actually consider SFC, but often it is done this way, because it is easier
2. Single Switch Layer 1 SFC - This is what this blog post will discuss in terms of BigSwitch
3. Multi-switch Layer 1 SFC - This is the Holy Grail of SFC, so to speak

## BigSwitch

BigSwitch is an SDN vendor with an interesting set of features. They provide an operating system that runs on baremetal whitebox switches (Switch Light OS) and a centralized controller. Their focus seems to be around security. I won't get into the details here, but they have a couple of different product options and what we are using is [Big Monitoring Fabric](http://www.bigswitch.com/sdn-products/sdn-products/big-monitoring-fabric/overview).

SDN is an interesting area to work in mostly because there are so many different ways one can implement networking when it is done (almost) completely in software. Certainly most of SDN systems rely on the now nearly standard ASICs in a physical switch, but the network logic is all in software. As I like to say, if we want to do SDN that is Apple Talk over USB we probably could. ;)

BigSwitch has an interesting layer 1 style SFC mode, which they call "big chains" or "inline security" or something to that effect. Using their technology one can dynamically insert devices into a chain of physical ports on a physical switch. This is not all that useful from an NFV perspective, as it is limited to a single physical switch and its physical ports, but in terms of a security chain, say one based on the connection of an insecure network (eg. Internet) to a secure one (a DMZ or other internal network), it is useful, certainly more useful than physically plugging and unplugging network cables. At least using BigSwitch we can do these things dynamically through a REST API.

Below we can see a "big chain" service chain. To start with there is no service inserted into the chain.

![initial chain](/img/chain1.jpg)

After inserting the service, we can see that some kind of device, in this case a firewall, has been dynamically inserted into the layer 1 port to port service chain.

![after insertion chain](/img/chain2.jpg)

It's important to note that this can be done manually from the web interface, or it can be done via the BigSwitch API. The ability to insert services directly into a layer 1 chain via a REST API is extremely powerful...it's hard to indicate with text and some pictures exactly how powerful that is. So take my word for it!

## Switchy

To accomplish the service insertion, I wrote a quick command line based tool that can access some of the BigSwitch features via the BigSwitch controller's REST API.

It's written in Python and uses the fun [Click](http://click.pocoo.org/5/) library to accomplish most of the structure for the actual commands. BigSwitch has some [Python examples](https://github.com/bigswitch/sample-scripts) which I used to create a little library for some of their Big Chain features. Click is also useful because it is smart enough to be able to use environment variables, so things like usernames and passwords don't have to be set on each run of the command, but can still be made required by the CLI.

```
$ switchy
Usage: switchy [OPTIONS] COMMAND [ARGS]...

  bigswitch command line interface

Options:
  --version          Show the version and exit.
  --controller TEXT  Controller IP or URL  [required]
  --switch TEXT      Switch ID  [required]
  --username TEXT    Controller username  [required]
  --password TEXT    Controller password  [required]
  --help             Show this message and exit.

Commands:
  insert-service  insert a service into a chain
  list-chains     list all bigswitch chains
  list-instances  list all bigswitch instances
  list-services   list all bigswitch services
  remove-service  remove a service from a chain
  show-chain      show a chain and services
```

For example we can list current chains, which show the chain name and which physical ports make up the ingress/egress points:

```
$ switchy list-chains
TEST-CHAIN-03, ethernet1, ethernet2
TEST-CHAIN-02, ethernet11, ethernet12
TEST-CHAIN-01, ethernet5, ethernet6
```

And list services:

```
$ switchy list-services
TEST-SERVICE-02
TEST-SERVICE-01
```

Or insert a service into a chain:

```
$ switchy insert-service --chain TEST-CHAIN-01 --service TEST-SERVICE-01 --instance 1
Inserted service into chain
```

Obviously anything one can do with the REST API can be done with Switchy, and of course we could add any logic that we would like into the command line application (which is the fun part of programming).

## Conclusion

Layer 1 port to port SFC is extremely powerful. It's important to note, however, that the BigSwtich example given here is not really one meant for NFV, as in NFV we have some kind of underlying infrastructure (termed the NFVi by ETSI), such as OpenStack, which manages many, many hypervisors, each with their own virtual switch. SFC that works in a multi-hypervisor/multi-switch environment looks much different than what is shown here, but this **IS** a stepping stone towards full blown SFC.

## PS.

In this post I've taken a very narrow view of what BigSwitch can do. The point of this post was not to review BigSwitch but rather to use it as an example of various ways people implement different kinds of SFC. BigSwitch is a great product and if it sounds interesting to you I highly suggest [trying it out](http://labs.bigswitch.com/users/login). Further to that I have not completely explored everything BigSwitch does nor how it does it.
