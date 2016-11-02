---
layout: post
title: OpenStack and ETSI MANO
categories:
header_image: /img/telephones.jpg
header_permalink: https://flic.kr/p/dtMQd 
---

# {{ page.title }}

The summit for the Newton release of OpenStack was held in Barcelona. I attended and found it to be a very interesting summit, one that, as usual, marks change for OpenStack.

OpenStack has always had more existential crisis than other open source projects. Previous crisis have regarded things like CloudStack (remember that?), the "public cloud" (which is still partially ongoing), the "big tent,” the continued--but improving--separation of developers and users, whether or not it is simply a vendor dumping ground, etc, etc. 

The current crisis would be around its growing adoption by telecommunications companies. As we enter a phase of OpenStack's lifecycle in which it sees serious adoption by extremely large, conservative, (*cough* inefficient) enterprises I would imagine that we could very well see OpenStack slowly change into something that might be unrecognizable to early adopters.

## Telecommunications

One of the biggest changes that has occurred in the past year for OpenStack is that telecommunication companies have (finally) begun adoption. And they are going all in, so to speak.

Telecommunications is potentially a 1 trillion dollar industry. Even a small percentage of that is a huge sum of money, and given the relatively minimal size of the OpenStack market share, is considerable in terms of how it could sway OpenStack's development and progress. (1) 

Jay Pipes, someone who has been around the community for some time, has thoughts, or at least a tweet, on where he thinks OpenStack is going:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Prediction: in less than 2 years, Telcos will have fully taken over <a href="https://twitter.com/hashtag/OpenStack?src=hash">#OpenStack</a> and converted it into a purpose-built NFVi+MANO solution</p>&mdash; Jay Pipes (@jaypipes) <a href="https://twitter.com/jaypipes/status/790943000931233792">October 25, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## MANO - Management and Orchestration

For those of you unfamiliar with the acronym MANO, it stands for "Management and Orchestration." The standards group ETSI has defined a large part of the way that telecommunications companies are going to perform Network Function Virtualization (NFV).

At this point OpenStack mostly makes up the NFV infrastructure (NFVi) and Virtualization Infrastructure Manager (VIM) components, and, in my opinion, currently stops there. The ETSI MANO layer, however, sits on top of all of this infrastructure, which would likely include many OpenStack clouds in many datacenters as well as other components such as Software Defined Networking (SDN) controllers, and magically manages and orchestrates it. (4) 

To suggest that OpenStack will also own the MANO component is surprising. So far, other than relatively unused multi-region support, OpenStack has not had many projects dealing with managing multiple clouds, or external systems outside of its own particular realm. Adding a MANO system would be a big change for the OpenStack project, one I'm not sure will happen. There are several open source MANO projects (2) and I have difficulty seeing them being moved within the OpenStack namespace. But, it could happen.

An example of an OpenStack project within reach of the MANO layer is the Tacker (3) project. Tacker is a generic Virtual Network Function Manager (VNFM) and a Network Function Orchestrator (NFO) but (I guess) not a full fledged MANO. But in effect it can act as one, as one of Tacker's features is the "[a]bility to orchestrate VNFs across Multiple VIMs and Multiple Sites (POPs)." So in some respects there is already evidence of OpenStack working (somewhat?) at the MANO level. Once could see a MANO project coming into existence (if it's not there already) and use Tacker as the VNFM component, and the rest of OpenStack as the NFVi and VIM blocks.

However, these MANO layers are going to be extremely important to the success of telecoms as they migrate/evolve to NFV. Further, these MANO layers are going to be extremely complicated and feature rich. My impression is that they will attempt to be all-encompassing.(5,6) They will have to talk to all kinds of complex systems. The MANO layer might simply be too big to be part of the OpenStack project and may more likely end up beneath the Linux Foundation (which is where Jay Pipes says OpenStack will end up eventually anyways).

Common sense suggests that OpenStack remain a set of primitives that higher level abstractions can use. So this MANO layer can use Neutron, Nova, Heat, Telemetry, etc, etc, to perform its job across multiple, complex systems, and not just OpenStack clouds. But, perhaps instead of common sense this is really just opinion, and other people and organizations will have a differing one.

## Footnotes

1. “Revenue from OpenStack, the open source software platform for cloud computing, will likely top $5 billion by 2020 and grow at a 35 percent compounded annual growth rate, according to 451 Research. While that growth rate is strong, the platform’s overall revenue is still fairly small compared to that of market leaders like VMware and Amazon Web Services (AWS), the firm said.” - via [SDXCentral](https://www.sdxcentral.com/articles/news/openstack-revenue-will-top-5b-by-2020-says-451-research/2016/10/) and [451 Research](https://451research.com/images/Marketing/press_releases/10.24.16_OpenStack_Pulse_PR_Final.pdf)
2. Eg. [OSM](https://osm.etsi.org/) and [Open-O](https://www.open-o.org/)
3. [Tacker](https://wiki.openstack.org/wiki/Tacker)
4. Should have called it ETSI MAGI. ;)
5. Video: [OpenStack and the Orchestration Options for Telecom NFV](https://www.youtube.com/watch?v=pxrDmqCMBLQ)
6. I would expect all-encompassing MANO systems, ones that try to take over everything (*hint* vendor lock-in *hint*) to fail.



