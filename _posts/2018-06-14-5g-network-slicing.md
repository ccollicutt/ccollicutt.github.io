---
layout: post
title: An Introduction to 5G Network Slicing
categories:
header_image: /img/slice.jpg
header_permalink: https://unsplash.com/photos/SpP3InZud7E
---

# {{ page.title }}

A colleague and I gave a presentation entitled "[5G Network Slicing and OpenStack](https://www.youtube.com/watch?v=x_b4-w4Mrcg)" at the 2018 Vancouver OpenStack summit. We did a lot of research, reading 70 to 100 papers (whitepapers, standards body papers, academic papers, you name it--so many we [published a bibliography](https://github.com/idx-labs/openstack-network-slicing/blob/master/BIBLIOGRAPHY.md_)) to try to determine what network slicing is and how OpenStack can participate.

## What is Network Slicing?

After reading all of those documents, I ended up with a set of quotes that I particularly liked. Here are a couple of them.

>“We define network slices as end-to-end(E2E) logical networks running on a common underlying (physical or virtual) network, mutually isolated, with independent control and management, and which can be created on demand.”- Network Slicing for 5G with SDN/NFV

>“Network slicing is a key feature for the next generation network. It is about transforming the network/system from a static "one size fits all" paradigm, to a new paradigm where logical networks/partitions are created, with appropriate isolation, resources and optimized topology to serve a particular purpose or service category...” - 3GPP 28.801

>“Slicing allows to concurrently support a large variety of use cases with diverging requirements on a common infrastructure platform...Examples of resources to be partitioned or shared, understanding they can be physical or virtual, would be: bandwidth on a network link, forwarding tables in a network element (switch, router), processing capacity of servers, processing capacity of network elements.” - Applying SDN Architecture to 5G Slicing

Whether or not network slicing truly comes into existence, it's fascinating to see how much change people think *could happen*. NS is nothing more than a total transformation of the network.

## What is Network Slicing?

The main reason Network Slicing (NS) is required is to meet the need to support diverse 5G use cases. Typically the following use cases are provided as examples:

* Enhanced mobile broadband
* Massive machine type communications
* Ultra reliable and low latency connections

The requirements of these networks are quite different. Some focusing on (much higher) bandwidth, and others latency. There is also a need to provide network access to 3rd parties and support MVNOs-like (Mobile Virtual Network Operator) business models. Safe, secure, and reliable critical communications also plays an important role. Slices could even exist across multiple operators.

Simply stated, Network Slicing is the ability to provide APIs that, through the use of virtualization and quality of service, partition a network into diversified sections (AKA slices) across a set of heterogeneous domains. Effectively, the same concepts used in Infrastructure as a Service are applied to the network. By creating network slices we can make physical networks capable of carrying diverse traffic in a secure, multi-tenant fashion.

## Requirements

![network slicing](/img/ns.jpg)

*(One of my favorite diagrams, care of [Netmanias](https://www.netmanias.com/en/post/blog/8325/5g-iot-network-slicing-sdn-nfv/e2e-network-slicing-key-5g-technology-what-is-it-why-do-we-need-it-how-do-we-implement-it))*

Once all of these papers have been read, a set of common requirements starts to develop.

* End to End - RAN, CN, Transport, edge clouds, core clouds--all of it :)
* Orchestration - Manage heterogeneous network resources
* On Demand - CRUD actions at any time
* Elastic -  Grow/shrink network services resources based on need
* Extensible - Expand network slice with additional functionality and characteristics
* Safety - Failures in one slice not causing failures in others
* Protection - Events in one slice do not have a negative impact on other slices
* Recursion - Ability to build a new network slice out of other slices
* Isolation - Guaranteed isolation and non-interference between network slices in data and control plane
* Flexible -  Simultaneously accommodate diverse use cases
* Exposure - Provide secure access to 3rd parties
* Network Functions - Not just switches and routers

## NS with SDN and NFV

![slice](/img/slice2.jpg)

*(image care of [metaswitch](https://www.metaswitch.com/blog/5g-network-slicing-separating-the-internet-of-things-from-the-internet-of-talk))*

SDN and NFV aren't strictly required to perform NS, but it would be surprising if they were not involved.

An important part of the definition of a slice is that it can include network functions. A slice isn't just a partition, but also a series or chain of network functions. Sometimes these functions are part of multiple slices.

SDN is also key because it provides the ability to program the network, and with functionality like Service Function Chaining (SFC) packets don't necessarily have to follow a standard layer 3 path. (Note that I don't define SDN as a centralized OpenFlow controller--it's much more than that.)

## How does OpenStack fit in?

Simply: it's just a domain. By end-to-end I mean multiple domains are crossed, such as the Radio Access Network (RAN), edge clouds, MPLS core, data center networks (and intra-DC), WAN, etc. That said, as a domain OpenStack will have to support the required features to enable network slicing. The good news is that by supporting virtualized networks and having an API OpenStack already meets a large portion of the aforementioned requirements. However, some work, a discussion of which would not fit into this post, still needs to be done.

## Conclusion

Network Slicing is a fascinating topic. Like any new, complex, paradigm shifting technology it's easy to push back on (see NFV), but from my perspective the same value in IaaS would be gained from NS, and IaaS has shown to have considerable return on investment, especially in the area of automation. To what extent NS requirements are met remains to be seen, but overall NS in some fashion makes so much sense its likely unavoidable.
