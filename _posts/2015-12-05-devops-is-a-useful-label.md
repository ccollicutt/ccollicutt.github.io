---
layout: post
title: DevOps is a Useful Label
categories:
header_image: /img/edmonton.jpg
header_permalink: https://flic.kr/p/A89DQq
---

# {{ page.title }}

## tl;dr

Define DevOps on your own. It's mostly about teamwork, not just automation. Ignore the haters--it's a useful label.

## DevOps and IaaS make their way up north

I live in Edmonton, Alberta, Canada. Because I've lived here so long I sometimes forget how isolated we can be. However, I don't mean isolated in terms of where Edmonton physically is. Surprising to many, we have a great university, a large international airport (I can fly direct to Amsterdam, and used to be able to fly direct to New York City), lots of good restaurants, a great theatre festival, and all kinds of wonderful things--especially in the summer when we celebrate being warm. :)

Instead what I mean is that it takes a long time for new technologies and related concepts to make their way up north. I'm talking about things like DevOps and using infrastructure as a service--be it AWS, other public clouds, and/or OpenStack. I might be in a bit of a filter bubble because based on my daily readings the technologies I just mentioned are so commonplace as to be standards. But not in Edmonton (yet). However, that is changing, and it seems to be changing right now.

In the last few weeks I've been talking to many people and organizations in Edmonton that are interested in modernizing, so to speak. Even though few companies in Edmonton are using public/private clouds or implementing concepts like DevOps, there are more every day.

## DevOps - What is it?

Many people I talk to don't like the term. They think it's a dumb fad. In fact I had to unfollow some people on twitter to get away from the constant negativity. I definitely think it's a real, and useful, thing. 

When people talk to me about DevOps I usually try to qualify what I think it is with the following statements:

1) It's a useful label
2) It has no real official definition, and in fact my be defined by the fact that it tries not to be defined
3) In my opinion, it's about creating high performing teams (which means empathy and communication, ie. teamwork) 

I should note that my perspective is one of deploying infrastructure, because that's what I do for a living. Most organizations use DevOps to deploy their applications(s) to a public cloud like AWS. A SaaS company is a canonical example: no servers of their own, many daily deploys, devs and ops working together, etc.

In my day to day work the application being deployed is OpenStack, and the code is Ansible playbooks. The infrastructure I maintain is defined in code or yaml as much as possible, and also goes through a continuous integration workflow. But I still have to deal with IPMI. :)

If you want to see a good presentation on the most recent thinking on DevOps, checkout [this talk](https://jaxenter.com/tools-culture-and-aesthetics-the-art-of-devops-122535.html) by J. Paul Reed. I wasn't sure that I was going to like the talk at the start, but in the end I agreed with much of what Reed said. It made me start to think about artifacts and aesthetics.

## It's not just automation

Most IT organizations want to increase their performance, but don't know how. To management, automation sounds great because then they might be able to reduce staff (which is, of course, not true). In fact I find that automation requires more staff, not less. Many companies try to get by with the minimal people required to complete a project and they hope to bring more on once the project is profitable. Unfortunately that is backwards. Automation projects should have a large front-end investment in people. But I can understand how finances can get in the way of doing that.

Automation projects also have high expectations. However, automation, like DevOps, is not a panacea. There's a great set of slides titled _[Infrastructure as Code Might Literally be Impossible](http://www.slideshare.net/ice799/infrastructure-as-code-might-be-literally-impossible_) that has many good points about how difficult things like programming (even bash) and operating system packages are. Basically a form of "how do computers even?" Once you abandon some abstractions it becomes clear that it is hard to determine what a complex system is doing. Repeatedly operating on that complex system in a predictable way is difficult.

At the very least automation systems like Puppet, Chef, and Ansible manage configuration files, services, and packages. If, at a low level, we can't trust how packages and package managers work, then how can we trust automation? Perhaps this is where abstraction fails us in computer systems engineering. I have been doing automation for quite some time and still find that it's easy to do an OK job, but difficult to do a _great_ job. Perhaps I'm not at the level I need to be yet, or I'm not choosing the right tools and workflows, or looking at it scientifically enough...I'm not sure. Regardless, writing a bunch of Puppet manifests isn't going to create magic.

## The switch to DevOps - change is hard

Not being able to define something important drives people crazy. Agile is nice because they have a convenient manifesto made up of a set of statements, or rules (which are often ignored). I'm sure that wasn't the entire point of Agile, but regardless the document exists and people link to it. 

With DevOps organizations need to define it on their own. I have seen more organizations fail at implementing DevOps related practices than succeed. The transition is extremely difficult because it requires a different mindset, and in the end relies on teamwork, which most organizations don't do at all, let alone well.

## A convenient label 

DevOps is an extremely useful label. Just because it's difficult to define doesn't mean an organization can't work on defining it internally (in fact that is what I think they *should* do). Just because it seems like a fad doesn't mean it can't help people to solve difficult problems and to create amazing teams that develop and operate complex systems and applications that are extremely profitable.

On one hand, there is no such thing as a "DevOps" but on the other hand, there are many people and organizations who do implement multiple facets of the DevOps paradigm, enough to be considered as "doing DevOps," certainly on their own terms. What's more, I often find that people who are already doing DevOps-like things are the most irritated by the label, perhaps because it's often misused and co-opted. I'm not clear on why people want to adopt DevOps strategies but reject the term.

Finally, if trying to work with the tools, processes and people involved in DevOps makes my life easier, and putting it on a resume helps me to find a great job, then I'm going to do that. People are both literally and figuratively searching for DevOps. Whether or not some people dislike the term, it's a useful label, and, I believe, a process for creating high performing teams.
