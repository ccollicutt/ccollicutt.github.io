---
layout: post
title: OpenStack Operators Midcycle, OpenStack Ansible Midcycle, and Ansible Fest
categories:
header_image: /img/london_bridge.jpg
header_permalink: https://flic.kr/p/8dtdiS 
---

# {{ page.title }}

This is a brief post on my recent trip to the UK to attend:

* The OpenStack Operators Midcycle meetup in Manchester
* The OpenStack Ansible project's midcycle meetup at Rackspace's UK HQ in London Hayes
* Ansible Fest London

## OpenStack Operators Midcycle Meetup (Manchester, UK)

For the first leg of the trip I spent two days in Manchester's Media City attending the OpenStack Operators midcycle meetup. Originally this meetup was for European operators. At the time I registered for the meetup, a few months ago, I was working at a Canadian OpenStack-based regional public cloud. Europe has several similar clouds and I was interested in meeting up with the operators of those clouds to exchange ideas. So that was my original intention in terms of attending the meetup. After registering, the meetup turned into the official midcycle operators event, and I also moved on to a new job that didn't involve running a public cloud. So both the event and my purposes for attending the event changed, but in such a way that it made sense for me to participate.

The most important thing I learned from the midcyle event is that OpenStack is the perfect cloud system for Europeans. The European Union recently put out a RFP (I have been looking but cannot find the actual document) requesting vendors to propose how they would run a series of cloud systems (IaaS, PaaS, etc) for the EU. In some parts of that document OpenStack was called out as a standard. 

OpenStack fits much better into the mentality of the EU than it does with North America. In my opinion, the EU wants OpenStack because OpenStack is an open project, because it's open source, it's an open community...all the values that the project stands for and stands on. In fact OpenStack is based on ["the four opens."](https://wiki.openstack.org/wiki/Open) This is opposed to what I typically see in North America which is reducing costs and trying to avoid the dreaded "vendor lock-in." (Neither of which are likely to be achieved when adopting OpenStack.) Further, European operators are interested in federation, and the private clouds of NA are not.

Other notes from the midcycle:

* I met a couple of people who are using whitebox switches and network OSes like Cumulus Linux. This is good because when I returned to my desk in Edmonton there were a couple of Edgecore 5712s on my desk that are due to go into our lab.
* Ceph continues to be a popular storage system to go along with OpenStack installations. I'm finally setting up a ceph cluster, for the first time, in the near future as we have a small cluster of ceph nodes on order as well, more lab fodder. There are large companies heavily relying on ceph storage for production systems.
* People are finding that Chef and Puppet are not completely working out. It's becoming common to utilize Ansible in combination, especially around orchestration (depending on how you define the word).
* OpenStack Ansible was used to deploy a large cluster (500+ compute nodes). Anecdotally people have said that once you hit a couple hundred compute nodes that it's time to start looking at cells. That does not seem to be the case with more recent versions of OpenStack. 500+ compute nodes is a lot of virtual machines.
* Speaking of cells...in Mitaka everyone will use cells V2, but to start you can only have one cell.
* Apparently a lot of "ceph enablement" has been done in Mitaka

Manchester's Media City reminded me a lot of the business area of Santa Clara. It's a vast business desert. Very few places to eat, no drug store, not much to do. In short not a great place to visit. I'm sure Manchester itself has a lot to offer, but if I was staying longer to enjoy the city I'd definitely be moving to different hotel downtown. I don't usually like to have breakfast in the hotel, but I had little choice in Media City. I find it amazing that hotels can charge $25-30 for a buffet lunch made up of oily eggs and weird tasting hashbrowns. At least the coffee had caffeine in it.


## OpenStack Ansible Midcyle

The midcyle ocurred at Rackspace's UK HQ, and there were about 12 people there, with a few more coming in and out online via video conferencing. Notably four members of the HPE Helion lifecycle management team were also in attendance.

I have been following the OpenStack-Ansible project since it started. At my previous job I was able to make use of some of the components of the project, such as the roles to deploy and manage MySQL Galera and RabbitMQ, but that was over a year ago. Much has changed. I would imagine the code has completely rolled over as the project has seen impressive growth. At my current position I'm hoping to make use of the OpenStack-Ansible project to deploy our OpenStack lab, and in doing so be able to commit some code back to the project. For the purposes of our lab I'll need OpenVSwitch (OVS) support, which does not quite exist in the project yet, but is on the way. Fortunately OpenStack-Ansible already supports using a ceph cluster (though does not deploy one, rather [ceph-ansible does that](https://github.com/ceph/ceph-ansible)).

Topics for discussion ranged from patterns and workflows that should be in use in OpenStack-Ansible to how to manage the life cycle of an OpenStack cloud. HPE (now with an E) recently released a snapshot of their [Ansible-based](https://github.com/hpe-helion-os) deployment and management system for their Helion project. Several HPE employees were in attendance as it seemed like there was the possibility of some collaboration, or at least shared learning (misery?) in terms of what each group has done with OpenStack and Ansible. The HP team had moved from Chef to TripleO and are now using Ansible, specifically because they found the previous two difficult to use in terms of managing (mangling?) an OpenStack system over time. Certainly Ansible is not the perfect tool, but, at least in HPE's experience, it is a more appropriate configuration management system for their purposes. Unfortunately the reality is that it is hard to maintain a life cycle for any system, especially complex ones like OpenStack. As I am often saying, even today in fact: it's easy to deploy OpenStack, but managing it over time (eg. upgrades) is much, much more difficult.

One thing I learned from the midcycle is that code talks. In many cases it's easier to discuss the results of a proof of a concept, based on code, than it is to discuss that concept without any code. It's easy to lapse into a two hour discussion about what _could_ happen if a pattern or workflow was followed, as opposed to a five minute discussion when you have actual code to look at. Code can talk much faster.

It was certainly nice to meet a good portion of the core OpenStack-Ansible team and to put IRC nicks to faces. I hope to be able to contribute back to the project in the near future as the OpenStack lab I'm working on is deployed. I'm appreciative of the OpenStack-Ansible team for allowing me to attend the midcycle, especially given my limited ability to contribute to the conversation.

## Ansible Fest

![ansiblefest](/img/ansiblefest.jpg)

_(Ansible Fest from the middle row, before the talks started)_

First off it's important to note that I was at a previous Ansible Fest, the 2014 event in NYC (I think that's when it was). It was a small event and one that I felt was technical. It's possible that I'm remembering incorrectly. However, I felt it was technical, and that the Ansible Fest London 2016 event was not. London was much larger, which is to be expected given Ansible's popularity and growth. According to the event staff there were about 530 people in attendance. Growth is good. RedHat buying Ansible is probably good. Certainly more developers have been added to the project and if that is the only thing that RedHat adds then that is positive just by itself.

However, I don't think I would go to AnsibleFest again, mostly because the event seemed to be focused on telling people to do automation, devops, immutable infrastructure and the like, as opposed to really showing what Ansible can do, ie. telling vs showing. Certainly there were some attempts to provide examples of powerful uses of Ansible, especially around networking and winops (a term I first heard at this event, which I like), as well as a presentation by jimi-c on how (and why) to write Ansible modules, but it just wasn't quite enough. If I went to an Ansible event again it would have to be more workshop-like or perhaps have multiple tracks with less showy presentations. In the end I think they did their best and catered to the crowd they knew they would have.

Another nit: for a day long conference it was quite expensive. Certainly cost is a relative thing, but compared to other events I've been to AnsibleFest was costly

A few things I learned from the conference:

* I was speaking with a company that works in the gambling industry, and they are using AWS as their main site host, but also have a secondary physical site ("luke warm" was the term used) in a co-location center so the government can have something physical to seize in case of some sort of legal or regulatory issue.
* Ansible is often used in continuous integration or deployment (CI/CD) pipelines. These pipelines are important to organizations because if the pipeline stops working, perhaps due to Ansible, then they can't deploy their code, can't get fixes or features out, and that costs money. So they want support from Ansible so they can get their pipelines back up and running as soon as possible. Basically everything in that pipeline requires support of some kind.
* Speaking of support, most of the questions in the question/answer periods were about support. Many organizations want Ansible support.
* Through a "raise your hand if" question from a presenter, it was noted that the large majority of people in the audience work at companies with more than 1000 employees.
* Ansible Galaxy runs on donated resources from Microsoft Azure.
* One of the main announcements from the conference is that Ansible is dedicated to supporting network automation, and released several modules for various networking companies such as Cisco and Cumulus Linux. I've been using the Cumulus modules for a couple weeks, so in effect they just moved the modules from one github repository into another, but it's great to see a focus on networking for Ansible. It's dearly needed.
* Ansible is in some ways a better fit for network automation because it's agentless. You can run Ansible over SSH, or you can write Ansible modules that use APIs that the networking system provides.
* CLOS networks are becoming the norm.
* Some organizations have to manage millions of lines of network configuration that are cut and pasted into network devices. I can't see how that works.

All of the presentation videos are [available online](https://www.ansible.com/videos-ansiblefest-london-2016).

## Europe in general

![Shakespeare's Globe Theatre](/img/globe.jpg)

_(Shakespeare's Globe Theatre, under construction for a new show)_

Other than getting the flu, which seems to happen on every trip, I had a great time traveling from Edmonton to Amsterdam to Manchester to London and home again. Edmonton has a direct flight to Amsterdam via KLM, so that makes travel to Europe easier for me. Unfortunately it's currently the only direct flight to Europe. We used to have a direct flight to NYC as well, but this isn't a "Edmonton Direct Flight Blog." :)

I was in five hotel rooms in nine days but everything worked out 0K, and I made it home in almost one piece. However, I did get lucky in a couple instances. At one point I was quite lost after getting off the train from Manchester (forgot about a transfer) and had to take an expensive cab from the train station to my hotel. However, the cab driver could not find the hotel and we drove around for an extra 30,40 minutes with the meter off as he tried to locate it. I did eventually get to my hotel and he was apologetic. In general I found Londoners to be a pleasant group of people no matter how many of them we crammed onto the underground. Once I was looking at a map of Canary Wharf and a policeman actually gave me directions. This has never happened to me anywhere else, not even in Edmonton.

Probably the biggest thing that I learned is that Europe (OK, just Amsterdam and London) is cold and rainy at this time of year. Edmonton is quite a dry place, and that makes it feel warmer than it is. Zero degrees in Amsterdam felt like -10 in Edmonton, and one of the first things I had to do was buy a scarf. I was freezing. Also it rains a lot in Amsterdam and London. It rarely rains in Edmonton, and my home town is one of the sunniest places in the world. We get over 2300 hours of sunshine per year versus 1662 for Amsterdam and 1481 for London [1]. Even notoriously rain-soaked-everyone-has-an-umbrella Vancouver has more sunshine than London. Poor London. (Apparently Yuma, Arizona is the sunniest place in the world with 4000 hours of sunshine a year...11 hours a day.)

My next trip should be to Austin for the OpenStack summit. I love Austin, but am a bit concerned about the recent Texas open carry laws. However, Austin is such an amazing place that it might be worth the risk.

***

1: [Sunshine Hours](https://www.currentresults.com/Weather/United-Kingdom/annual-sunshine.php)

