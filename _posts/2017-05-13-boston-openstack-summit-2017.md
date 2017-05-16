---
layout: post
title: OpenStack Boston Summit 2017
categories:
header_image: /img/infinite.jpg
---

_(A picture I took at the Boston Museum of Fine Arts)_

# {{ page.title }}

I'm not sure how many OpenStack summits I've been to, the first one I attended was the San Diego summit. I felt like this was one of the better summits I've been to, mostly because I had some colleagues to hang out with. Unfortunately, while other people from my current employer were supposed to attend, due to business reasons they had to cancel. But I was able to have some good discussions with some fellow Canadians (we have [slack channel](https://openstack-canada-slack-invite.herokuapp.com/) now) as well as my co-presenters for the multi-cloud/mult-site/what-have-you session.

## NFV

I mostly work in the area surrounding "Network Function Virualization" (NFV). If you haven't been to a summit in a while, I think I can sum up telecommunications companies participation in OpenStack by saying Verizon sent 100 people to the summit. As another example, AT&T is leading the [OpenStack Helm](https://git.openstack.org/cgit/openstack/openstack-helm/) project. Yes, AT&T, and according to their presentation on the helm project, they will be replacing all of their current OpenStack control planes with it (presumably moving away from Mirantis Fuel, which itself has been replaced by Mirantis MCP, but I digress).

## OpenStack Helm - Deploying the OpenStack Control Plane on Kubernetes

Previously I wasn't sure if the OpenStack control plane really needed a full _Container Orchestration Engine_ (COE) like Kubernetes, because it's hard enough to run OpenStack let alone an underlying (and complex) COE. Personally I usually use LXC based containers. But after seeing some of the work AT&T on the helm project, I think my mind has changed. That said, I am not sure how they are provisioning the actual underlying Kubernetes infrastructure, which I presume is outside of the scope of the Helm project (ie. probably something like kubadm.

I wanted to attend the hands on session, but it was at the same time as the telecom/NFV working group session. From this [Calico blog post](https://www.projectcalico.org/3-takeaways-on-the-future-of-openstack-networking-from-the-boston-summit/) it sounds like they were using Calico in the workshop. I wonder if that is what they are using in production...that would be surprising:

> Even more impressively, the AT&T Integrated Cloud team hosted a hands-on workshop with over 100 participants in a packed room also performing the same task of deploying, and upgrading OpenStack on Kubernetes leveraging Helm, and Calico for networking, and completing this lab in mere minutes.

Can/should Telecoms act like Facebook? Not everything hyperscalers like Facebook is doing is applicable, but if you are using containers then I don't think everything should be dealt with on layer 2. We should be using IPv6 with containers. But Calico with IPv4 and treating everything as layer 3 is a good start.

Treating infrastructure as software is difficult. It's not the same as running a web application. There is a lot of hidden state in the underlying infrastructure. However, we cannot continue to treat infrastructure, such as private cloud, as hand-rolled bespoke, manually installed one-off systems. We have to find the right balance between running highly available infrastructure, and treating it like software, with continuous integration, and multiple deployments per day, as well as reconciling the fact that infrastructure will fail and applications have to be "cloud native." Too far one way, and infrastructure is unstable, too far the other and we end up with set-and-forget.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">@ Most pop workshop <a href="https://twitter.com/OpenStack">@OpenStack</a> Summit on OpenStack-helm with <a href="https://twitter.com/v1k0d3n">@v1k0d3n</a> <a href="https://twitter.com/portdirect">@portdirect</a>. thanks to <a href="https://twitter.com/kubernetesio">@kubernetesio</a>  to move OpenStack to next level! <a href="https://t.co/cRhoQFZYNO">pic.twitter.com/cRhoQFZYNO</a></p>&mdash; Archy (@archyufa) <a href="https://twitter.com/archyufa/status/862802663779577857">May 11, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### AT&T: Zones and Regions

First, this is something I need to do more research into, because I only heard a tiny snippet of a conversation around this, but, it sounded to me like AT&T was deploying clouds with a region and zone concept, where every region had two (availability?) zones. This would be very similar to what AWS, and any intelligent cloud provider, would do. Presumably the concept is that if one zone fails (which is probably an entire cloud) then the other (geographically close) zone is still running, and whatever application you deploy would exist over both zones.

My point is not that this is a new concept, but the fact that a large telecom grasps, and has implemented, zones is impressive because it shows some "cloud building" (sorry) maturity.

## NFV: Nirvana Stack

Naming is hard. I don't know if I like the name of this project, but I can understand where they are coming from. :)

Telecoms and groups like OPNFV are coming together to establish some kind of reference stack with OpenStack and other open source systems, mostly around "Sofware Defined Networking" (SDN) plus OpenStack. I could not watch all the presentations, as they had an entire day dedicated to this in the OpenDayLight track, but my impression is that while using OpenStack has become fairly standard to do NFV, implementing an SDN solution is not part of the OpenStack project itself, and typically telecoms want the extra features and capabilities that an SDN solution can provide, but they are often difficult to integrate. So they want to perform some integration work on OpenStack + "Open Source SDN", like Open Daylight, and this work is under the umbrella of the _Nirvana Stack_.

It makes sense to me as while I have done a fair amount of work with ODL, installing it with OpenStack is not that straight forward. Also, part of the integration would be to identify and help to fix any issues upstream.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">ATT Lisa Fung gave introduction to Nirvana stack and objectives <a href="https://twitter.com/OpenDaylightSDN">@OpenDaylightSDN</a> day <a href="https://twitter.com/hashtag/OpenStackSummit?src=hash">#OpenStackSummit</a> <a href="https://t.co/dW6AmumjzU">pic.twitter.com/dW6AmumjzU</a></p>&mdash; George Zhao (@georgeyzhao) <a href="https://twitter.com/georgeyzhao/status/862658180681265153">May 11, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## NFV: ONAP

If you work in the NFV area, you have probably heard about AT&Ts [ECOMP](http://about.att.com/content/dam/snrdocs/ecomp.pdf) project. Basically it's a massive layer of software (MANO+) that sits above OpenStack performing all kinds of automation. However, ECOMP has now transitioned into being a project named ONAP, and it's in the process of [merging](https://www.sdxcentral.com/articles/news/logical-happens-open-o-merges-ecomp/2017/02/) another open source MANO system, Open-O. This transition is ongoing and will take some time, but at this point there is [actual working ONAP code](https://git.onap.org/)!

AT&T demoed ONAP (ECOMP) at the summit, and also provide Heat templates which will deploy the ONAP application itself into an OpenStack cloud. (However, I believe at this time the templates are only tested with Rackspace, though any OpenStack cloud (with Heat) would presumably work, perhaps with a little tweaking.)

I'm really impressed with AT&Ts willingness to release this code. Combined with their work on projects like OpenStack Helm, they are doing some really great things. Very impressive for such a huge telecom.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Telco grade Orchestration ONAP for Openstack Orchestration <a href="https://t.co/sBLsB7TeU7">pic.twitter.com/sBLsB7TeU7</a></p>&mdash; Sudeep Batra (@sudeepbatra) <a href="https://twitter.com/sudeepbatra/status/862324663510335488">May 10, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Open Contrail

Contrail is a SDN system from Juniper. They have an open source version called [Open Contrail](http://www.opencontrail.org/). On Wednesday night they had a Contrail user group meeting at Fenway park.

[Randy Bias](https://twitter.com/randybias) recently joined Juniper to spearhead the Contrail product. For those of you not familiar with Randy, he is usually considered the progenitor of the "Pets vs Cattle" meme (which I don't like, but sometimes it's the easiest way to get the point across). So, you know, he like gets cloud and stuff and has for quite some time.

He started off the Open Contrail night by saying they (Juniper) were going to totally reboot the community. Everything he said sounded great, and are things that need to happen with Open Contrail, with one exception...he mentioned the term "open core." I personally don't think open core really works...but then again I am not much of a business person. That said, I'm not actually clear on what he meant by open core, so we will just have to wait and see. Overall, the changes he mentioned during the talk were all things that need to happen. I'm not sure Juniper corp. will like it much, but he was certainly saying the right things.

One of the major items Bias mentioned was that he wants to grow the number of Contrail deployments by an order of magnitude or two. I think that this is something that needs to happen if Contrail is going to be successful. They just don't have enough deployments right now to get good community contributions and feedback...it's a single vendor open source project and those rarely work out. It's not so much that they need to make more money (ie. get more commercial deployments), which of course they do, it's that they aren't getting enough feedback and external contributions to improve the product, not enough critical mass.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Another great <a href="https://twitter.com/hashtag/opencontrail?src=hash">#opencontrail</a> meetup at the <a href="https://twitter.com/hashtag/OpenStackSummit?src=hash">#OpenStackSummit</a> by <a href="https://twitter.com/emaganap">@emaganap</a> good lineup of speakers and their stories at <a href="https://twitter.com/hashtag/fenwaypark?src=hash">#fenwaypark</a> <a href="https://t.co/2xg8yowWUe">pic.twitter.com/2xg8yowWUe</a></p>&mdash; Amit Tank (@amitTank) <a href="https://twitter.com/amitTank/status/862769221398351872">May 11, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Massively Distributed Clouds

Part of the presentation I worked on for the summit regarded "massively distributed clouds" (MD). As usual there are many issues around this in terms of naming and use-cases definition. I was in several sessions during the week regarding the phrase "massively distributed" and most of the sessions became quickly confusing to all in attendance. Often telecoms (somehow) know they need massively distributed clouds but they don't actually know what it means, what is possible with OpenStack now, what isn't, and what would require new architecture and designs...and most importantly new code.

I heard three or four things in these meetings over and over again which are not related to what I would consider _massively distributed clouds_:

1. Some people just want massive single data center systems. This is possible now and is not massively distributed. It's just a large cloud; a buncha compute nodes. If you have 5000 servers in a single data center I don't think it matters if you have to dedicate 0.5% of them to the control plane. It's massive, but not massively distributed.
2. Some organizations want a massive number of distinct, _separate clouds_. This is possible now. There are not many good options for managing a large number of clouds, and would likely involve some kind of Management and Orchestration (MANO) system and other software to manage and sync them, but I don't really consider it a massively distributed use case. Large number yes, distributed no.
3. Another desire is for "zero touch provisioning" in which you have many small clouds or devices and you want a "zero touch" method install and update them. To me this is an important requirement, but it can be solved many ways outside of OpenStack, and would probably be a project of its own (or maybe using Ironic...I don't know). Hopefully something is happening around this in the IoT area or we are in big trouble security-wise. The point is that we can provide this functionality outside of OpenStack if required.
4. I also encountered some people, more than one person, OK, two, that was quite sure that if you use Kubernetes or containerize the OpenStack control plane then it somehow gets smaller. I suppose it's possible, but in general that is not the case. No matter if you run all the of the required control plane services in containers or not they are still taking up resources, need to be managed, etc.

Finally we have what some people call "the retail use-case" in which an organization, say, for example, Walmart, has 10,000 stores with a few hypervisors in each. They don't want to put a full, highly available OpenStack control plane in each and every store, ie. the cloud is made up of remote (or distributed) hypervisors. I believe that if we solve this use case we would solve most related problems, ie. reducing the size of the OpenStack control plane, dealing with latency, etc. To accomplish this OpenStack would have to change considerably. In my mind, this is what _massively distributed_ really means. Sure, you have some other problems to solve (like "zero touch") but overall if we could solve this problem then many other issues and use-cases fall into place.

There's an [etherpad](https://etherpad.openstack.org/p/BOS-Fog-Edge-MassivelyDistributed-BoF) page (essentially online collaborative notes) from one of the sessions, the Birds of a Feather meeting, if you are interested in perusing some of the thoughts around MD.

I should also mention that several people from the OpenStack foundation were in the room, including Sparky Collier, Jonathan Bryce, and Lauren Sell. It's pretty clear to me that the foundation believes MD (or edge, fog, what have you) is important to the future of OpenStack.

## The Forum

A major change to this summit was the creation of _The Forum_. Previously summits were also well attended by developers and there was an entire section of the summit dedicated to getting developers together discussing their projects. However that work has now been moved to a separate meeting, called the "Project Team Gathering" (PTG), which is not held at the same time and location as the summit. The first one was held in Atlanta in February, and I believe the next one is in Denver. What this means is that there are fewer developers at this summit. Three people mentioned to me that the summit felt more business like, and they didn't know about the forum component how it had changed the makeup of the attendees.

I attended many forum sessions, and they mostly turned out well. Certainly all positive discussions.

## OpenStack Operators Telecom/NFV Working Group Session

I have been involved with, and currently chair, and OpenStack working group called the **OpenStack Operators Telecom/NFV Working Group**. I won't go into too much detail on what this group is about--it's for people who build and operate NFV related clouds on a day to day basis, and the OpenStack foundation was kind enough to provide us some time and space to meet face to face.

There is an [etherpad](https://etherpad.openstack.org/p/BOS-ops-telecom-nfv) page that was created for this session and it has some notes about what we discussed during the session. Have a look. :)

Overall, I made a request to everyone in the room to go back to their respective organizations and seek out people who might fit well into this group and to let them know about it. Hopfully at the next [meeting](http://eavesdrop.openstack.org/#OpenStack_Operators_Telco_and_NFV_Working_Group) we have some new members!

I should also mention another working group in the OpenStack community that has ties to Telecom and NFV: the [Large Contributing OpenStack Operators](https://wiki.openstack.org/wiki/LCOO) group (otherwise known as the LCOO). I think they missed an opportunity to name themselves the _Contributing OpenStack Operators who are Large_, ie. COOL, but no one asked me. :)

I've been working pretty closely with this group for a while, and they have been welcoming and accommodating, as they try to work with various large, usually telecom, companies who want to participate in the OpenStack community but maybe don't quite know how...yet.

## Conclusion

For me, the summit was good. I missed hearing about what the developers are doing and working on by being able to sit in on their sessions, but I can understand why the OpenStack Foundation moved to creating the PTG concept. Only time will tell if it can improve the development of OpenStack, but I have a feeling it will. The worst part was that there wasn't as many free drinks and candy around...only devs get that perk. ;) Devs, devs, devs.

Because I work in the NFV area, I was extremely attuned to what is happening around it at the summit, and there is a lot. AT&T, and soon Verizon (among others?) are really trying to participate in the open source community and putting resources into writing open source code. In contrast, other, usually smaller, telecoms are in over their head and will continue to rely on beating up vendors as their main technical strategy. My recommendation to all telecoms would be to invest in people and everything needed to make them successful.

It's going to take telecoms time to figure it all out, but if they do, then they will make a considerable impact on OpenStack over the next few years, and in fact it would not surprise me if NFV became the dominant use case for OpenStack...as telecoms around the world will be deploying thousands of clouds, which honestly I believe is thousands more than what has been deployed in the enterprise.

Now I'm looking forward to the OpenStack Operators meetup in August in Mexico City, and the next summit which is in Sydney Australia. It's a long way to go, but it should be fun.
