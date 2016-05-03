---
layout: post
title: Austin 2016 OpenStack Summit 
categories:
header_image: /img/cart.jpg
---
_Random shopping cart in Austin_

# {{ page.title }}

##Or, You Know What’s Cool? A Trillion Dollars

The OpenStack summit is a large, multi-faceted event. It brings vendors, operators, developers and users (among others) all together in one massive conference. At this summit there were ~7500 people in attendance [1]. The scale wasn’t just based on the number of people though--the number of conference tracks and talks was almost overwhelming.

##Niche Clouds

First I have to say that I felt that, even before leaving for the event, I would be more interested in what was going on at this conference than the Tokyo summit. During that summit I worked on operating a small public cloud in Canada. There are not that many, in my opinion, OpenStack-based public clouds. Thus it was hard to find some kind of operations solidarity in that particular niche. While I believe that there are in total somewhere around 18 public OpenStack-based public clouds [0], some much larger than others, their influence, especially the smaller ones, pales in comparison to the number of private OpenStack cloud deployments. 

At any rate, my point is that I no longer work in this particular OpenStack niche, and have moved into a new niche, one that is getting a considerable amount of attention.

##Network Function Virtualization (NFV)

I knew coming into this summit that NFV would be pretty high on the hype cycle. The sleeping telco giants have all been startled awake and their millions of employees and billions of dollars are yawning and stretching. Despite the hype of NFV and influence of telco (TELCO SMASH!) it is an exciting time in technology, especially around networking, automation, containers, and I would argue, operations. Change is good, but it’s hard. 

In one of the sessions it was suggested that telecommunications is a trillion dollar market. That is a massive number, and most of these organizations are only getting started with OpenStack. However, I have no doubt in saying **all** of them are looking to deploy OpenStack clouds. (Certainly, some will be implementing OpenStack clouds in hopes they fail, and they probably will, but others will be putting in an honest effort to succeed.) This is a sea change in IT if there ever was one.

Almost every networking related design session I went to dealt with, in some capacity, the need to improve support for telecommunications requirements--especially around technologies and workflows like service function chaining (SFC) and monitoring/fault management. I went to several network component design sessions and SFC was discussed in nearly all of them. Networks are about to get a lot more complex, but also easier to automate and program.

##Ops Telco/NFV Working Group

I moderated a session in the Ops summit on Telco/NFV. Quite a few people showed up and we had a good discussion on Telco/NFV. [2] I think there is so much interest in NFV that people are looking anywhere for help and information. That’s why we ended up with so much interest in the working group. However, it’s an ops group, so if there is an operators need for the Telco/NFV group then we need to make sure we’re meeting that need, and not overlapping with other working groups and entities. So it will take a bit of time to figure out what we can do. 

One thing I can say is that I quite like the approach the Ops large deployment team has taken, which is to select one or two changes they would like to see in OpenStack and really shepherd those changes through the...uh stack over a long period of time to make sure they get into the project. More on this later.

##Open Platform for NFV - OPNFV

OPNFV is a really interesting project. I realized at this summit that OPNFV is essentially a group of OpenStack operators. They are doing the same work a large operators group would do, specifically around getting OpenStack and NFV up and running and nearly ready for production. They are modelling and improving, as best they can, OpenStack + NFV in production-like environments. I watched a couple great presentations by people working within OPNFV, and I'm hoping to go to their summit in June in Berlin.

##Ops Informal Meetup

At the Tokyo summit there was an informal ops meetup on the Friday, basically hanging out in the dev summit lunch area. I didn’t attend that meetup because there wasn’t quite room at the time for me even to sit down. I didn’t mind because I just went out into Tokyo for the day. But it apparently was a great meetup because the foundation scheduled a large room for the entire Friday for operators to meetup and have an informal discussion about being OpenStack operators. The result was a wide-ranging discussion on almost all facets of OpenStack. We ended up with a 700 line etherpad. [3]

Some items I took out of the informal meetup:

* Maybe there is a need in Horizon to have some sort of "simplistic" interface option, where simple means less options. After having worked in a public cloud I have no problem saying that most customers are using it to create long-lived virtual machines, and are not using much of the other functionality, at least not yet. Having some kind of operator option to...er reduce options would be a good idea. See Digital Ocean.
* Redfish - I'm still not sure what this is, but if it's about making IPMI/ILOM/BMC/out-of-band management easier then I'm all in. [4]
* ZFS is in Ubuntu Xenial, but there is no OpenStack Cinder driver that I'm aware of. That'd be cool. Someone should do that. I would if I was good enough to do it quickly, but I'm not.
* OCP hardware - There were a couple of people deploying OCP hardware. OpenStack Ops discussions never seem to talk much about hardware, perhaps because it's so vendor centric. One would think OCP would be a different story. That said, it's not always easy to buy small amounts of OCP hardware.

Those are just a few things that I took from the informal session. I had hoped to split out the Telco/NFS working group in the afternoon, but I had the feeling most of the Telco/NFVers left after the morning session and the facilities just weren't quite setup for splitting off. Perhaps the next ops meetup or summit will be better suited.

##OpenStack Mission Statement

I went to one BoF session on the mission statement. It seemed like the goal of the session was to try to ascertain how well OpenStack was doing in particular areas mentioned in the relatively new mission statement:

> To produce a ubiquitous Open Source Cloud Computing platform that is
easy to use, simple to implement, interoperable between deployments,
works well at all scales, and meets the needs of users and operators of
both public and private clouds.

They broke out pieces of the statement:

* Ubiquitous
* Open Source Cloud Computing Platform
* Easy to use
* Simple to implement
* Interoperable
* Scales
* (Can't remember how they broke out the rest)

and we talked about how well we were meeting these objectives.

We got stuck on easy to use and simple to implement for a while. I personally don't think those are actually OpenStack goals, especially around "simple to implement" because OpenStack is not a singular product, rather it's a framework to build "Open Source Cloud Computing Platforms." OpenStack doesn't really have any artifacts of its build process--it's just a collection of code and some documentation, and of course, a community around that code. (Actually I think they publish tgzs somewhere...)

The tone in the room felt unusual, in that the people writing and thinking about the mission statement are pretty far away from actually using it or operating it. I understand that these phrases are long term goals for OpenStack, but at least one is not possible to meet.

I would probably write something more like:

>To produce an ubiquitous Open Source **framework** for cloud computing that is easy to use, interoperable between deployments, works well at all scales, and meets the needs of users and operators of both public and private clouds.

and just get rid of the whole "simple to implement" component.

##Austin

I've been to Austin twice now. Austin is a great city, very eccentric. My home town of Edmonton always wants to be Austin. I took the bus from the Alamo Drafthouse downtown to Book People, and it was the happiest bus ride I've ever been on. An entire bus load of strangers was just laughing and having a great time. I thought I had stepped onto some kind of roaming festival, but nope, just a normal Austin bus ride. Oh, and I had a quick discussion with an Austinite who had recently watched Strange Brew. If it wasn't so damn humid in Austin I could probably live there. Humidity is only 40% in Edmonton today.

Also--Cheer Up Charlies was a great bar. People were bringing their dogs in! Crazy. Also, Argo's foodtruck at Cheer Up Charlies had the best vegetarian bacon burger I've ever had. It was perfect. I think my biggest personal expense for the trip was eating there.

##Conclusion

OpenStack, public clouds like AWS, automation, containers, software defined networking--I was wondering where we’d go after innovation in these areas. At this point it seems like Telecos are now entering these relatively new spaces and bringing their considerable clout to bear. Perhaps we flounder around here a bit and then move on to whatever comes next. There is some danger of the massive resources of telcos putting pressure in the wrong places for OpenStack, and that things might break (though I didn’t see any obvious signals at the summit). 

I would imagine the OpenStack foundation is working hard to determine how to best direct those resources and ensure that the OpenStack project stays healthy, but it will be a difficult job. Telcos are strange beasts, but I think it’s important to remember that they, as organizations, have more change to implement to use OpenStack than vice-versa, mostly around their culture. Relatively speaking it's much easier to deploy OpenStack than it is to adapt your culture to use it. OpenStack does offer a tremendous opportunity for telcos, and I suppose I should really be worried if telcos can handle the change, not the other way around.

---

[0] [https://twitter.com/e_monty/status/725482310879997953](https://twitter.com/e_monty/status/725482310879997953)

[1] Note that the summit will be changing over the next year or so. I believe the design portion of the summit is going to be split out into its own event.

[2] [https://etherpad.openstack.org/p/AUS-ops-NFV-Telco ](https://etherpad.openstack.org/p/AUS-ops-NFV-Telco)

[3] [https://etherpad.openstack.org/p/AUS-ops-informal-meetup ](https://etherpad.openstack.org/p/AUS-ops-informal-meetup)

[4] [http://redfish.dmtf.org/redfish/v1](http://redfish.dmtf.org/redfish/v1)
