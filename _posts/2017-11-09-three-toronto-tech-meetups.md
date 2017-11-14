---
layout: post
title: Three Toronto Tech Meetups
categories:
header_image: /img/to.jpg
header_permalink: https://unsplash.com/photos/3OdGZzTt1fM
---

# {{ page.title }}

Recently I moved from Edmonton to Toronto. There were a lot of reasons to make the move, and I won't get into them here. Suffice it to say I'm sure I'll be living in Toronto for the next five to ten years, then maybe, if I'm really lucky, retire to Drumheller, AB or Nelson, BC, a nice small town somewhere, something like that. It's good to have goals. (Yes the housing market is insane in Toronto, and Ontario is much more expensive overall compared to Alberta...financially speaking moving here was probably a mistake. But so is having dogs.)

One of the things I felt would be valuable about living in Toronto is the large information technology community. I was part of a lot of meetups and the like in Edmonton, such as the Python meetup and the Golang meetup. There were several good meetups that I didn't attend, so it's not like there was a lack of technology, it just didn't have the breadth of Toronto. Often it was hard to get enough people to come out to the events...there's a certain number that is a critical mass of attendees and we often couldn't hit that number. We had a good location to have those meetups in Startup Edmonton, but attendance could be challenging. Not so in Toronto. (Though getting space here might be next to impossible.)

## Meetup #1 - Toronto Artificial Intelligence and Deep Learning Meetup

The title of this talk was "How Machine Learning Saved $25 million!"

I enjoyed the presentation, as it was more of a practical overview of how to do machine learning from a process standpoint, based on the example of an e-commerce site trying to predict the shipping weight of items, errors in which was costing millions of dollars per year. A real life issue to be sure.

Ultimately, I believe the point of the presentation was not a technical one, rather that Machine Learning is really 75% data science, 10% deployment, and 15% actual ML. The ratios aren't from the presentation, nor accurate, but my point is it's mostly data science, and then you still have to deploy the ML code somehow (maybe as a H2O.io plain old java object?).

I'd say about 120 people attended, and the audience was very diverse in terms of demographics, which was nice to see. It took place at the WeWork space in downtown Toronto, and was organized and sponsored by H2O.io which is some kind of ML and AI toolkit. They were very nice about being sponsors and tried not to tout their product too much. I'm sure it cost $1000 or more to rent the WeWork space, so thanks H2O.io.

## Meetup #2 - Toronto Pivotal User Meetup

I don't really know anything about the Pivotal product, other than it's some kind of PaaS based on Cloud Foundry, and they have some kind of configuration management tool, BOSH, that is apparently magic, and that they are a very successful business unit of DELL/EMC or whatever that conglomerate is called. That sentence sounds a little flip, but I really do like that Pivotal exists and I'm happy that they are doing so well in the Global Fortune 500ish space. It says a lot about automation, containers, and just delivering applications. I would imagine that OpenShift is a competitor of sorts, and OpenShift is also doing very well in the enterprise.

The talk was by [SnappyData](http://snappydata.io). First off, please take what I say here with a grain of salt because I don't have any experience with Apache Spark. My impression of what Snappy does is allow data to be stored in Spark so that you don't have to go out to an external cluster, say a Cassandra system, to get data, which reduces complexity and latency. I might be wrong about that but that is certainly my impression from the talk.

Snappy is actually based on an existing product called [GemFire](https://pivotal.io/pivotal-gemfire) which apparently has been around for more than a decade and is in production in 1400 customers. I get the impression that this is one of those pieces of software that is quite valuable to those who know they need it. It's always amazing to me how valuable software can be. There are all these small, hidden companies making tens of millions of dollars of a bunch of C code that a small, long term team has been writing and refactoring for years. But that's just me speculating based on previous experience.

Anyways, a lot of features Snappy has come from Gemfire. But, one of the more interesting things that Snappy can do is statistically sample large data sets and query that instead of the whole data set. This makes the queries much faster. Less accurate, but faster. But often there are situations where we don't need great accuracy to make decisions or do things, such as visualization. It's called their "synopsis engine" and seems very promising for quick decision making.

Also, I learned about [Apache Zeppelin](https://zeppelin.apache.org/).

## Meetup #3 - Toronto Enterprise DevOps

Next, I went to the Toronto Enterprise DevOps meeting held at ObjectSharp, who also bought the pizza. (So far every single one has served pizza. This particular pizza was not great lol. But three days of pizza is two days too many.)

I really wanted to go to this meetup because it was called "Hashistacking the Enterprise" and I thought this meant that there were enterprises in Toronto that were really into the Hashicorp stack, ie. Vault, Nomad, etc. But the talk, although well done, was really an introduction to Hashicorp projects.

The presenter had recently worked at Chef and was a big fan of Inspec so that was also discussed. I like Inspec too but it is difficult to use across a large fleet because you have to replicate the inventory somehow, though that wouldn't be too hard if you have a centralized CMDB of some kind. But still a pain.

I have worked a little bit with Nomad, mostly using it as a distributed cron system which is not really what it is meant for. I have also used Vault a bit, but am having some issues understanding the security model around it handing out certificates. I used to be a big user of Vagrant, but I don't run VMs on my local workstation any more.

One thing I did learn is that Terraform does everything in order, much like Ansible, and opposed to say Puppet which has a compilation step. I should say I have not used Terraform yet. Another interesting point is that it can be used for multiple clouds, as opposed to Cloud Formation or Azure ARM. That point/feature is quite important and I think Terraform is a successful product simply based on that.

Their recently released Sentinel product was discussed as well, and I think it is absolutely fascinating as a policy engine, but it is closed source so I doubt I will ever have a chance to use it. Maybe it will spur an open source project that provides similar functionality. If only I was a 10x coder... ;)

## OpenStack Toronto

I am also helping out with the OpenStack Toronto meetup. Space is a challenge.

## Conclusion

Toronto has a diverse tech community, but I'm just basing that on three meetups. There are all kinds of people doing all kinds of technology, some advanced, some not, some with new technology, some not. This is great to see.

Obviously space is a big issue for having meetups in TO. I am trying to help the Toronto OpenStack meetup, but there are challenges for getting a low cost space that meets all requirements. The meetup space market, just like the housing market, is very difficult to get into.
