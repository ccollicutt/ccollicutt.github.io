---
layout: post
title: Kubernetes and Mimesis - The YAML is not the (M)App
categories:
header_image: "/img/not-the-map-1.jpg"

---

# {{ page.title }}

100% of organizations I have worked with and talk to either want to automate or already doing so. Everyone "automates." Everyone "DevOps."

> "Yes, yes, we have DevOps, we have automation. Been doing that forever. But it's not enough." - Customer

Most want to do more than what automation can provide on its own. In my opinion, doing more demands abstraction.

## Fear of Abstraction

Technology leaders often equate abstraction with "lock-in," be it technical lock-in, contractual lock-in, or other. Abstraction is, to them, a concerning concept and something to be avoided.

To engineers the mention of abstraction conjures a fear of the inscrutable. To engineers, abstraction is a black box. Key, even sacred, information is hidden within the confines of the dreaded cube. Of course, some information hiding is precisely the goal of abstraction. However, as an engineer might ask you, if information is hidden, then how can one truly understand every component of their vast and complex platform? Full stack or full stop.

Thus we end up, as an industry, in a situation where abstraction is alarming to both technical leadership and engineers. So it is typically avoided, especially in terms of infrastructure.

Instead of innovating and building powerful business outcomes on abstraction, we recreate--we imitate--the existing terrain by drawing an extremely detailed but novel map (just like the old one). With a new and detailed map in hand engineers feel safe and energized. With a new and detailed map, technology leaders have an artefact that can be displayed under glass with a label of progress. But I'm getting ahead of myself.

![](/img/not-the-map-2.jpg)

## Release the Shackles!

Prior to, oh say around about 1910, give or take, artists painted real things and those paintings looked like real things (as much as a two dimensional image can, anyway). In fact, if you didn't paint real things that looked like real things, say something "abstract," it was cause for serious concern. Take, for example, the harsh criticism of James Whistler's "Nocturne in Black and Gold – The Falling Rocket" via which he eventually entered bankruptcy.

When Malevich created his most well known work, "Black Square," it was arguably "the first time someone made a painting that wasn’t of something." Malevich's work was designed, in part, to free the art world from its current limitations related to the explicit desire to continually imitate something real.

Most importantly, I believe, in Malevich's work "the white backgrounds against which they were set mapped the boundless space of the ideal." When we avoid abstraction in technology, we also miss out on the benefits of the open playing field it creates. Without abstraction we don't have room to run.

![](/img/not-the-map-3.jpg)

## Kubernetes and Mimesis

First off, no, mimesis is not some new Kubernetes related project or product (at least...I don't think it is). The typical definition of mimesis revolves around the idea of imitating or reproducing reality. To reiterate the quote I have from Richard Spalding above, "Malevich's intent [was] to liberate painting from the shackles of mimesis." I believe mimesis is an easy trap to fall into with k8s.

Make no mistake, I am a big fan of Kubernetes. I even spent three hours wrangling YAML for the Certified Kubernetes Administrator exam. That said, I do feel like we, as an industry, have to be careful not to end up creating yet another detailed map of the territory. We should endeavour not to simply imitate existing constructs. We have to be willing to not only accept the wide open playing field the box/square/abstraction provides but to have an appetite for it. And, most importantly, use k8s to create it.

This point has been well made by some of the originators of Kubernetes, people such as Joe Beda and Craig McLuckie of Heptio, and now VMWare, fame. Others too.

![](/img/not-the-map-4.png)

Kubernetes role is not to mimic the existing platforms and systems, ie. a map where the scale is a mile to a mile. Rather its mission is to provide the advanced machinery necessary to develop new abstractions on which we can then create novel, and financially beneficial, work. Abstraction doesn't create black boxes, it provides the freedom to invent.

In conclusion, suffice it to say, the YAML is not the (m)app.