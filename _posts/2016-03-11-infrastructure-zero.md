---
layout: post
title: Infrastructure Zero
categories:
header_image: /img/glitch-david-szauder-1.jpg
---

*[Image by David Szauder](http://pixelnoizz.wordpress.com)*

# {{ page.title }}

## tl;dr

In terms of Infrastructure as a service (IaaS) the point of open source software and hardware IMHO the point is to:

1. innovate faster than closed source, and,
2. drive down the cost of IaaS to near zero to focus on deploying applications.

Perhaps #2 is merely a by-product, but I think there is a considerable distinction between what enterprise organizations think they are supposed to do with Open Source Software (OSS), ie. not pay for it and fire staff (lol), and what they are really doing: using highly collaborative, world-wide software and hardware projects to implement  *and manage* new technology and reduce the costs of particular layers (such as, right now, the infrastructure layer) so that we can spend money on higher level systems, if not higher level abstractions, and thus do more with the same resources.

There's a difference between getting something for free, and using that something to commoditize systems and move up the value chain.

## Props to Randy Bias

In this [blog post](http://www.cloudscaling.com/blog/openstack/five-things-openstack-needs-to-do-now/) Randy Bias, now of EMC, formerly of Cloudscaling (which was purchased by EMC) mentions that infrastructure has no value.

> All value is from applications that consume [infrastructure].

In [this video](https://youtu.be/rI13aja9zm0?t=12m32s) he talks a bit more about *infrastructure zero*, likening infrastructure to utilities like power and water, and how it is not a differentiating part of your business in that we all need CPU, storage, and networking--that what *is* differentiating is the applications, and custom code, that utilizes those primitives. 

I've always liked what Randy says about OpenStack and infrastructure, because I feel that he's usually right, but the market tends to disagree with him and go on to screw everything up. Cloudscaling had some good ideas regarding infrastructure choices for OpenStack, but even though the company "exited," ultimately it failed, and as far as I know the Cloudscaling product no longer exists.

In an [Appformix blog post](http://blog.appformix.com/openstack-2016-what-you-need-to-know), Mr. Dunnigan rephrases Randy's theory:

>...the sole purpose of infrastructure is to serve the applications they run.  Infrastructure running zero workloads has zero value.

## Infrastructure Zero

![glitch](/img/glitch-5.png)

*Image: The facade 2010, Robert Overweg*

Currently IaaS is going through a period in which we are attempting to use OSS and hardware, as well as automation, to drive down the cost of operating IaaS as close to zero as possible. The reality is that IaaS has little value unless it's not working, in which case it has negative value. 

But in general its value is zero or close to zero, and the real goal is to get the **cost** of operating IaaS as close to zero as possible. But there is a long way to go, and doing so requires resources and talent given that important components of IaaS, such as OpenStack, are not as operationally mature as we would require to fully commoditize. Further, while we have whitebox network switches the operating systems available for them are relatively new.

This is not to say that OpenStack is not "production ready" (it is) rather that it doesn't necessarily cost less to operate than other traditional virtualization solutions because of the human resources required to, for example, upgrade it. That said it does come with what is now considered a standard API which most other IaaS-like systems do not. So if you consider the costs a wash then just having an API will improve things. But I digress...

## Public Cloud

I don't want to get into private cloud vs public cloud debate in this post. That said I feel comfortable saying a large majority of IT organizations will be using public clouds like Amazon, Azure, and Google. But it's unlikely to be 100%, and will take many years, if not decades. There will still be tens of billions of dollars available for private clouds, just not hundreds of billions.

Further, [some people](http://superuser.openstack.org/articles/why-openstack-is-the-kernel-of-the-data-center?awesm=awe.sm_eOjxU) believe that the bigger Amazon Web Services gets, the bigger OpenStack (ie. private clouds) will be. However, based on who is saying that there may be bias in that thinking.

Suffice it to say that I am a believer in public clouds, but also a realist in terms of the fact that businesses have to sell what people want, and many organizations--at least right now--want private clouds.

It's from this standpoint that I make most of my generalizations on *infrastructure zero*, in that work that public clouds, and other large IaaS users (eg. Facebook) are are doing is trickling down into private clouds (see Open Compute or the Xeon-D processor), and in conjunction with software like OpenStack we hope to drive the cost to zero so that we can work on deploying applications and systems that provide differentiation and ultimately *make money*. 

## Nobody wants IaaS, but everyone needs it

![glitch city](/img/glitch-city.jpg)

I have been working with OpenStack for several years, including deploying a public cloud in Canada based on OpenStack. I can say without reservation that no one wants *just OpenStack*. 

Deploying an OpenStack cloud and handing over credentials is a surefire way to fail, because, most often, end users will have no idea what to do with it. Users want to deploy applications. Simply having access to OpenStack APIs is (usually) not enough. There needs to be another layer of abstraction in between IaaS and most users--another layer of utility.

Recently Werner Vogels, of AWS fame, posted an [entry](http://www.allthingsdistributed.com/2016/03/10-lessons-from-10-years-of-aws.html) on his blog that detailed ten things he has learned building a public cloud (perhaps *the* public cloud). One of his points regards "primitives":

> One of the most important mechanisms we provided was to offer customers a collection of primitives and tools, where they could pick and choose their preferred way to engage with the AWS cloud...

What I take that to mean is that building a cloud requires ensuring certain primitives exist so that layers of abstraction and workflows can be defined and overlaid on those blocks. The primitives are obvious (compute, network, storage, etc) and have to exist, but they do no directly provide value as we still need abstraction and automation above them. 

In many ways I am surprised that AWS has grown so quickly given that it is a complex system of primitives sans frameworks. In this post I am suggesting that primitives are not enough, and yet AWS is thriving, and is, I believe, one of the fasted technology companies to a 10 billion run rate. 

However, as AWS matures, and the ([somewhat creepy?](http://www.nytimes.com/2015/08/16/technology/inside-amazon-wrestling-big-ideas-in-a-bruising-workplace.html)) socio-techo organization behinds it learns what frameworks and workflows its customers want, I expect it to begin identifying layers of abstraction that will continue to drive growth and enable simpler, faster, more powerful deployments of technologies, making it easier to develop, deploy, scale, and operate new applications. (Such as [games](https://aws.amazon.com/lumberyard/).) They'll develop frameworks that will quickly become standards.

Another point of note: I recently listened to a podcast interview with someone from Pivotal (a platform as a service product based on Cloud Foundry). Unfortunately I can't find it so this is anecdotal. Occasionally when Pivotal talks to prospective customers, these customers have technical staff who ask what operating system their code is running on, and Pivotal pushes back on that question, because why do they need to know? Is the application running properly? If yes, then it's irrelevant what the underlying virtual machine operating system is, just like it's irrelevant as to what the IaaS is. The abstraction layer will (hopefully) deal with it. The user doesn't need to know. The PaaS can ask the IaaS for CPU, storage, and network. Push your code up, it's running properly...that's all that matters.

## controller.controller
![controller.controller](/img/controller.jpg)


By "controller.controller" I mean a "controller of controllers" or "manager of managers." In the Vogel post I mentioned about he also uses the term "framework" which is probably similar. Right now this area is rather messy. Everyone is working on building these systems and it is a difficult thing to do. It reminds me of how many Javascript frameworks there are. I don't know if we want as many IaaS abstraction frameworks as there are Javascript frameworks. Then again most attempts at a "single pane of glass" fail. So somewhere between two and several.

From my perspective a controller of controllers could be as simple as using OpenStack Heat or Hashicorp's [Terraform](https://www.terraform.io/), or as complex as the [ETSI defined MANO layer](http://www.etsi.org/deliver/etsi_gs/NFV/001_099/002/01.01.01_60/gs_NFV002v010101p.pdf). The point is that when an OpenStack system cloud is deployed, so to should systems that enable higher level abstractions, and likely more than one.

These systems will be incredibly important in terms of not only managing infrastructure but also in terms of driving business profitability, on both sides of the equation.

## The Infrastructure|Software Paradox

![price of software](/img/price-of-software.jpg)

*(Image from [Bloomberg](http://www.bloomberg.com/news/articles/2015-02-05/six-things-technology-has-made-insanely-cheap))*

I recently ordered the book [The Software Paradox](http://thesoftwareparadox.com/) but it hasn't arrived yet so I haven't read it. But I can't wait to dig in. The premise:

> The Software Paradox explores the counterintuitive idea that software’s realizable commercial value is headed in the opposite direction of its market importance.

Essentially ["software is eating the world"](http://genius.com/3115790) but it's getting harder to make money off of it. How can that be?

In many ways this post regards "The Infrastructure Paradox" in which infrastructure is more important than ever but it's become less profitable. Or is it simply that it has become commoditized and has lower margins which requires a different business model? To be honest I don't know, but the reality is that the change is unstoppable.

## What do we do now?

From my standpoint as a long-time systems administrator, I feel like my best path is to work to aggressively reduce the cost of IaaS (ie. automation, monitoring, ect), make deploying applications as easy and automatable as possible, and understand the dynamics of the public and private cloud marketplace. Further--as a systems administrator of all things--understanding the business model of a product that is simultaneously incredibly important but has limited positive financial value seems paramount.

---

<a name="band">1</a>: controller.controller was a [Toronto indie rock band](https://en.wikipedia.org/wiki/Controller.controller). I don't know if I've ever even heard one of their songs, but I remember their name because it always seemed like a tech company rather than a band. 
<br/ >
<a name="utility">2</a>: Not sure where to put this but perhaps this is all just a rehash of "utility computing" that I believe originated in 1961!
