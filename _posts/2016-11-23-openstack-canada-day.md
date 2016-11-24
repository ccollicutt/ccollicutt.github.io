---
layout: post
title: The First OpenStack Days Canada
categories:
header_image: /img/openstack_days_canada.jpg
---

# {{ page.title }}

On November 22nd, 2016 the first [OpenStack Days Canada](http://openstackca.org/) (OSDCA) was held in Montreal. [OpenStack Days](https://www.openstack.org/community/events/openstackdays) are regional events that bring together people using and operating OpenStack clouds. They are slightly bigger in size and geographic pull than your typical meetup, but not nearly as big as the OpenStack Summits. I believe the attendance at the first OpenStack Days Canada was around 190, which is very good. You can find a few tweets about the event with the [#OSDCA hashtag](https://twitter.com/search?q=osdca&src=typd).

Basically they are a small, one or two day conference. There are speakers, panels, and some of them do workshops and such, and perhaps some kind of social evening event.

## Montreal a Hotbed of OpenStack

There is a ton of OpenStack related activity in Montreal. Actually, [technology in general](http://www.cbc.ca/news/canada/montreal/montreal-institute-learning-algorithms-1.3859460). Montreal is an extremely wired city. There are many data centers, service providers, and service providers for service providers. There are great schools here, relatively low cost of living, and many interesting companies. It's probably the only place a boring anglophone like myself could live in Quebec, as English is pretty common, though I still feel like an doofus even ordering a coffee.

Some of the companies that were at OpenStack days and which are based in the Montreal area:

* Vexxhost
* Inocybe
* Internap
* Ormuco
* CloudOps
* and more I'm sure

## The Talks

There were two tracks of talks at OSDCA, a "user" track and a "fan" track. I jumped around a lot between the two tracks.

Below are the talks I attended. I can't remember what company everyone was from, but I remember a few.

**1 - Running Production Ready Kubernetes Cluster on Openstack** by Ayrat Khayretdinov

It's impossible to attend any kind of OpenStack technical event without hearing about Kubernetes from either the standpoint of 1) running the OpenStack control plane in k8s, or 2) running k8s in OpenStack on instances as part of a project. This talk was regarding option #2 (and I assure you the two models are completely different and almost not related at all).

One of his main points is that currently k8s does not do multi-tenancy well, so one way to do that would be to deploy multiple k8s in different OpenStack projects.

He also reminded me of the [Murano](https://wiki.openstack.org/wiki/Murano) application catalog system which I should look into again, as it does manage TOSCA templates (I believe), which is related to the work I do in Network Function Virtualization (NFV).

**2 - Ceph OSD hardware – a pragmatic guide** by Piotr Wachowicz of [Bright Computing](http://www.brightcomputing.com/)

This talk gave real examples of what kind of hardware and configuration is best for a Ceph cluster. There are real considerations to make in terms of the sizing of the Ceph nodes, especially in regards to the number and size of disks, the amount of SSD caching, and finally the size and utilization of the networking components. NUMA is also a concern, and frankly it seemed that one socket systems match up very well for Ceph storage nodes. It also sounded like PCIe SSDs would make a lot of sense in Ceph nodes as they can cache for a larger number of spinning disks.

**3 - Tales From OpenStack’s Gate: How Debugging the Gate Helps Your Enterprise** by Matthew Treinish

This was part of a talk that was done at the [Vancouver summit](https://www.openstack.org/summit/vancouver-2015/summit-videos/presentation/tales-from-the-gate-how-debugging-the-gate-helps-your-enterprise). I was quite interested in their use of Elastic Search, specifically their elastic-recheck setup. Hunting for bugs using Elastic Search is a pretty interesting subject.

**4 - InfraCloud, a Community Cloud Managed by the Project Infrastructure Team** - by Paul Belanger

The OpenStack foundation is now running its own OpenStack cloud, called [infracloud](http://docs.openstack.org/infra/system-config/infra-cloud.html). The OpenStack CI system uses many, mostly public, OpenStack-based clouds to run millions of tests. Now they can also rely on their own internal cloud as one of those regions. Infracloud does not need to be highly available because if it is down they can just use one of their other clouds. But they are learning what it is like to actually run an OpenStack cloud, other than DevStack, which I think is very valuable to operators such as myself, as they will likely find bugs that other testing may not.

Paul also reminded me of [bifrost](https://github.com/openstack/bifrost) which is a way to deploy a stand alone Ironic system to manage baremetal servers. Basically you could use this instead of something like Cobbler.

Finally, Paul went through their configuration management tooling which is a combination of Puppet and Ansible. They no longer use a true "puppet master" system, and instead run Puppet via Ansible, which is surprisingly quite common. 

**5 - OpenStack and Opendaylight integration** by Rashmi Pujar and Mohamed Elserngawy from [Inocybe](http://www.inocybe.com/)

I'm a big fan of OpenDaylight. I'm hoping to add OpenDaylight to my OpenStack lab environment, which is currently running "vanilla" OpenStack networking, and only using provider networks.

The talk was quite dense, and they had a demo as well, and frankly they just ran out of time. It's great to see a Canadian company, Inocybe, trying to lead the way in an important open source technology like OpenDaylight.

Many of the advanced networking features being worked in the OpenStack ecosystem are first developed using OpenDaylight, so if you want to be on the cutting edge of things like "service function chaining" then running OpenDaylight can be important.

**6 -Store All the Things: Storage options for your OpenStack Cloud** by Sean McGinnis

Sean is the Primary Technical Lead (PTL) of Cinder. He discussed Cinder (block storage), Manila (shared file systems) and Swift (object storage). I often forget about Manila, and it would be an important and useful system for applications that need shared storage.

His talk would be useful to people who aren't sure how to use storage in an OpenStack cloud.

**7 - OpenStack adoption stories: Linux Foundation & Opta Insurance Services** a panel with with Konstantin Ryabitsev (Linux Foundation) and Jin Lee (Opta Insurance, a division of SCM insurance) moderated by Mohammed Nasser ([Vexx Host](https://vexxhost.com/)).

This was probably the most interesting discussion for me as Jin Lee from Opta is a mathematician with the company and they do machine learning. He was very positive about their use of OpenStack and how easy it was for him to start writing basic code to spin up hundreds of instances to run their machine learning algorithms. The OpenStack foundation would have been very happy to hear his comments. If I remember correctly they use a combination of in-house OpenStack and Vexxhost to obtain computing resources.

He also mentioned speed as being important, in terms of their ability to run algorithms, discover information, and then create microservices that can provide that information. If they can provide services to their customers faster than their competitors, that is an advantage.

## Toronto is Next

The 2017 OpenStack Days Canada will be in Toronto. I look forward to attending and perhaps next year I will submit a talk.

## Thanks to the organizers

I'm sure a lot of work went into this event. Thanks to the sponsors and organizers of the event, and I look forward to the 2017 OpenStack Days Canada.
