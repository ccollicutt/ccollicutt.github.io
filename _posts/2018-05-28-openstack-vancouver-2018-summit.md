---
layout: post
title: OpenStack Vancouver Summit 2018
categories:
header_image: /img/vancouver-2018.jpg
header_permalink: https://unsplash.com/photos/dacj4UrnT3k
---

# {{ page.title }}

Last week I was at the OpenStack Summit in Vancouver. This is the second time I've been to a OpenStack summit in this amazing location. It is probably the best place in the world to hold a conference, as the conference center is beautiful and the view is absolutely unbeatable. I am sure that this will not be the last OpenStack conference held in BC.

## OpenStack Slows Down and Matures

Having been at the previous Vancouver summit, I believe in 2015, I know that it was much larger. I'm sure that the attendance at the 2015 version was six or seven thousand. This summit I believe was about 2500 people. There is a good reason for the reduction in attendance in that there are not as many developers that go as they now, instead, attend another event called the "PTG" (Project Team Gathering) which usually happens in a less expensive city (such as Denver...but strangely also Dublin). So that drops attendance by about half.

The other factor is that OpenStack is maturing. It's coming through they hype cycle and settling down. Certainly there is a lot of work that needs to be done to keep OpenStack going, especially with regards to NFV, but overall it's a bit less "exciting" at this point. For example Telecoms are putting it into production. Think about that. The most conservative organizations in the world and deploying thousands of OpenStack instances.

For another perspective, Redmonk has a [balanced blog post](http://redmonk.com/sogrady/2018/05/25/openstack-at-a-crossroads/).

>Many event organizers, OpenStack included, set themselves up for this criticism by using attendance themselves as a sign of health and progress, but that doesn’t change the fact that moving towards smaller events can be a boon for communities as it makes the events more navigable and weeds out less committed third parties. Attendance, in other words, is not a particularly useful metric if you’re trying to assess the OpenStack project’s health, and not just because it forked its developer and user events.

Should there be a Linux Foundation, an OpenStack (or Open Infra) foundation, and a CNCF foundation? I think so. OpenStack's community is really built on collaboration. I give them a hard time occasionally, but compared to other open source projects they really try to keep things civil. I think the triumvirate of these three foundations is a powerful substrate for open source software development. Two might not be enough, and four too many.

## Open Infrastructure

As OpenStack focuses on other open source projects under their umbrella (Zuul and Kata Containers) it's becoming fairly obvious that OpenStack will rebrand in some fashion around "Open Infrastructure." They are putting out signals that this might happen, and I think it makes sense. I help to run the Toronto OpenStack meetup, and as well I recently started an Open Source Networking meetup. Just from my own view it would make sense to rebrand our meetups as Open Infrastructure and start focusing on a broader topic base.

Vendors like to try to pin OpenStack to just being a set of APIs into which any backend can be plugged into. However I don't really view it like that as OpenStack is a Linux ecosystem integration platform. It really is open infrastructure. I prefer to consider what gets tested by the OpenStack CI system as being what "OpenStack" is, and add in some key components like Ceph and open networking, as well as automation systems such as ONAP.

## Network Slicing

We presented a couple of talks at the summit. The one that probably resonated most with attendees was our presentation on [5G Network Slcing and OpenStack](https://www.youtube.com/watch?v=x_b4-w4Mrcg). Like a lot of technologies, it's not that easy to define what Network Slicing is, but I personally find it fascinating, if at the very least academically. Certainly Network Slicing will occur in some fashion, but it is possible we won't call it that...could just be some form of end to end virtualization.

## Code Drops - Airship, StarlingX, Akraino...

There were several code drops from larger companies:

* [Akraino](https://www.akraino.org/)
* [StarlingX](http://www.starlingx.io/)
* [Airship](http://www.airshipit.org/)

### Akraino

AFAIK Akraino is essentially an integration project to create an edge stack. I'm not sure how much code actually exists at this time, , but I think it's a good idea to do work in this area whether it's successful or not. Certainly there is a good opportunity to get involved in the project right now as it's still quite fluid.

I personally still think that innovation occurs in open source, so this kind of project is needed.

### StarlingX

Wind River has an OpenStack distribution (Titanium) which is aimed at Network Function Virtualizion (NFV) infrastructure and thus telecoms (and to be fair, industrial use cases as well). They had many proprietary changes to OpenStack in order to improve it and for various reasons have decided to try to upstream that code.

Wind River is a pretty interesting company. I've got to give props to any company that writes code that has to work or otherwise lives are in danger.

>For over 20 years, Wind River® has provided NASA with the most proven software platform to bring dozens of unmanned systems to space, resulting in some of the most significant space missions in history. - [Wind River: 20 Years in Space](https://www.windriver.com/inspace/)

### Airship

It's unfortunate that it seems like we need another way to deploy OpenStack, but apparently we do. From my limited introduction to Airship I like the overall tone of the project in terms of how it's abstracting the layers, and I believe it is also taking care of Kubernetes deployments as well. But don't quote me on that as I haven't looked at it closely enough. It also relies heavily on [OpenStack-Helm](https://docs.openstack.org/openstack-helm/latest/readme.html).

## Conclusion

Overall, it was a nice summit. Times are a changin'. I fully expect the OpenStack Foundation to rebrand in some fashion, that is assuming once doesn't already consider them as having rebranded with "Open Infra" and emphasis on Zuul and Kata Containers as new, separate projects. I also expect a push back on complexity in NFV, but we need infrastructure that has APIs and schedulers, so the complexity is unavoidable, we just have to get good at it.

The foundation has changed the PTG so that now operators, such as myself, are invited. We used to have a separate operators meetup, but now that will be co-located with the five day PTG. So the next time I go to an OpenStack event will be the upcoming PTG event in Denver.
