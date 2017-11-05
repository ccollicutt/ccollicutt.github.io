---
layout: post
title: Three Pillars of Modern Networking
categories:
header_image: /img/pillar.jpg
header_permalink: https://unsplash.com/photos/pVq6YhmDPtk
---

# {{ page.title }}

Computer networking is at an interesting state. We have many new interesting technologies and methodologies for building and managing networks, but ultimately those new technologies have not made much of a dent in how the vast majority of computer networking is architectured and operated.

I don’t have a classical networking background, no CCIE or anything like that, but I am very interested in new networking technologies and I believe in some cases my lack of traditional training (dare I say “baggage”) allows me a bit more room to operate in terms of adopting new technologies.

Ultimately we still do networks with things like ARP, STP, VLANs, LACP and BGP. We connect relatively large broadcast domains together with layer 3 routes managed by BGP. That’s really about it. It does work, but we aren’t really moving ahead, even something like Ipv6 is just not seeing adoption. (There may be valid reasons for that, but I mention it anyways.)

In this post I want to talk about three major technologies that can help to improve modern networking:

* Network Automation
* Software Defined Networking
* Intent Based Networking

## Network Automation

At its base, Network Automation is the application of the same configuration management tooling and methodologies as have been applied to server infrastructure, but instead of servers we simply apply them to network switches and routers. That’s about it. Useful? Yes. Revolutionary? No.

Ansible, the configuration management tool I usually reach for (which has as many shortcomings as it does valuable features) has probably taken the lead in having the most easily accessible network automation tooling as well as scope of devices it is able to manage. Ansible’s module based approach makes it pretty simple for even novice developers (like myself) to create new modules and thus there are many core modules related to networking in Ansible.

Python is a go to language for many people who need to write code but being a developer isn’t necessarily in their job title, people such as network engineers. Python also has a substantial network library called Napalm that can help to abstract many network devices into code.

Ultimately, configuration management is probably a stop-gap measure between current methodologies and something better, something that has not necessarily arrived in networking yet. In the area of “server administration” overall configuration management is still valuable, but containers have moved the cutting edge to immutable style deployments based on one-time images as opposed to continuous usually agent based configuration management.

Will this happen to networking...unknown. Technically many switches and routers are already configured to utilize an image model of updates and monolithic configuration files, but convergence time could be an issue in devices that are expected never to slow a packet let alone stop accepting a flow, nevermind the absolute fear in the minds of network admins around changes. Some switches, like Arista, can run containers and I expect to see more of that. Further, many large organizations are realizing the millions and millions of lines of unused code on switches and routers are a liability and are requesting modularity from vendors or rolling their own simpler networking operating systems (though companies doing that are the likes of LinkedIn and Google). But I digress.

## Software Defined Networking

Software Defined Networking (SDN), like many new technologies, was, for a while, considered a potential panacea. Unsurprisingly that did not turn out the way many expected or hoped. However, what is surprising is how little overall traction SDN was able to achieve, and how few SDN startups, if any, succeeded. The joke that SDN means “still don’t know” managed to become reality. Sure we have some successful deployments, but their functionality is limited in terms of the massive scope that SDN originally had.

I have been working with SDN for quite some time at about the time when everyone was just starting to realize OpenFlow was not going to win out. Well over 3 years ago now, I designed and deployed one of the first production SDN deployments in Canada when I built a public cloud based on OpenStack which used Midokura’s Midonet product. Midonet is a good example of where the SDN ecosystem has gone as they never quite achieved the success they were looking for and have pivoted into another business model and, I’m afraid, ultimately might not exist.

What I like about SDN is that is presented the possibility to create networks that can do anything. Unfortunately, what we ended up with SDN is a bunch of overly complex control planes that are essentially wrappers around VXLAN, BGP, and some kind of distributed state stored in a system like ZooKeeper et al (see OpenContrail, Nuage, Midonet, etc, etc). Because they rely on an underlying physical network (the “underlay”) and universally speak BGP at the edge, all the legacy networking deployments we are used to are still required and instead we just pile on SDN as another layer. (“So now you have two problems/networks.”) I do think overlays are valuable, but the complexity required to get them in this exact context are considerable.

Further,  AFAIK, most SDNs are deployed in concert with either OpenStack or VMWare, though VMWare is essentially (practically speaking) enforcing only using their SDN product NSX. I am quite biased in terms of my experience mostly being related to OpenStack, but I have not met anyone doing SDN outside of the context of a virtualization system and usually SDNs are “boxed inside” of the virtualization platform, though they may interact with larger networks at the BGP level for North/South traffic.

Frankly I think that we didn’t push hard enough on really rethinking networking as a whole. As well the traditional, dare I say “heritage”, networking is too embedded and difficult to remove, and thus we ended up just piling more layers on top of if. On one hand if it ain’t broke don’t fix it, but on the other we can’t stagnate at the network level, because the network really is a major component of  “the computer.” Are we losing value as a species with our inability to move past IPv4?

I look at technologies like Service Function Chaining, which, while still based on the reliability of standard networks, make packets appear on ports as though transported through a wormhole, as a direction that SDN should have gone in. Networks don’t have to look like they do.

I still have hope for SDN. :)

## Intent Based Networking

Intent Based Networking (IBN) is where SDN was three to five years ago.  IBN is a lot of things, but I believe that first and foremost the idea is that some kind of consumer asks for something, such as transport from A to B, and IBN determines how that is done. This is pretty well described in the paper [Intent NBI – Definition and Principles](https://www.opennetworking.org/images/stories/downloads/sdn-resources/technical-reports/TR-523_Intent_Definition_Principles.pdf) presented by the Open Networking Foundation.

Practically speaking it’s an admission that organizations are unwilling to alter or replace their existing legacy infrastructure, or legacy vendor relationships, with SDN. IBN is a higher layer of abstraction working above network automation and seeks to utilize a methodology where the intent of the network (say, to run applications) is described in some kind of language or configuration, and the actual deployment and operations of the network is created from that “intent” but that the actual underlying physical and virtual devices are not relevant.

In some situations this simply looks like a piece of software that can abstract the configuration of multiple brownfield systems (all your Juniper and Cisco and Arista routers and switches, different versions and variations thereof) and then (re-?)configure them in such a way as that the network will fulfil the obligations “suggested” by the intent or consumer requests. (See Apstra Networks.) Instead of human beings interpreting business and technical requirements and writing, often by hand, over long periods of time, thousands of configuration files for network devices, and what's more/worse keeping the connections in their head, the IBN system automates that, in effect forcing us to forget about the how and just focus on the what and why. At least in theory anyways.

Machine learning may also be applicable to IBN, so expect to see considerable fanfare around that. Maybe IBNs can the “hallucinate” the network design (ala [letsenhance.io](http://letsenhance.io) product that upscales any JPG to 4K by hallucinating detalis).  Or maybe we can have AI invent new network protocols like AlphaGo created new Go moves.

## Conclusion

So there you have it. Three technologies that I believe are the current pillars of modern networking. They all have issues, but promise as well. Maybe IBN and Machine Learning will take us to some kind of network nirvana, or maybe it will stagnate like SDN seems to have. Only time will tell.
