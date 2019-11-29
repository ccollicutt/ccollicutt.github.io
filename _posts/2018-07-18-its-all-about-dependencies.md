---
layout: post
title: Infinite Regress - Dependencies All the Way Down
categories:
header_image: "/img/turtle.jpg"

---

# {{ page.title }}

I have a pet theory that it's all about dependencies.

I used to work in an area, Network Function Virtualization, NFV for short, that is made up of many complex systems. The most obvious example is OpenStack, which is notoriously considered complicated. To be fair, there are many more high level systems like it that are also part of the ecosystem. Also see Kubernetes or almost any Software Defined Network.

## Complexity Reigns

It's ["turtle's all the way down"](https://en.wikipedia.org/wiki/Turtles_all_the_way_down) as OpenStack is built on the Linux ecosystem, which is itself extremely complex, never mind the underlying hardware Linux typically runs on: CPUs, proprietary firmware and drivers, etc. Code is everywhere. And code has dependencies.

> "Turtles all the way down" is an expression of the problem of [infinite regress](https://en.wikipedia.org/wiki/Infinite_regress).

I've spent some of my 20 years in IT as a Linux systems administrator which has given me a particularly strong view of operating system dependencies. Having worked so much with OS packages (RPMs, deb files) and similar mechanisms (Pip, CPAN) I'm keenly aware of how important dependencies are, and how challenging they can be to manage.

## Hidden Dependencies

Unless you write code or manage systems that have packages you might not be as aware just how many dependencies there are. Often organizations pay companies to abstract away dependency issues.

As far as I'm concerned, a large part of what you are buying when you purchase commercial software, especially distributions of complex systems like OpenStack, is the ability to mitigate the risk of dependencies.

However, ignoring dependencies through abstraction is a double edged sword. The thesis of this article is that "it's all about dependencies." In my opinion, if an organization is going to be good at managing complex technology they also must be (very) good at managing dependencies. If dependencies exist throughout the technology stack ("turtles all the way down"), there comes a point when every organization must deal with them.
Breaking Changes

I recently read this fascinating paper, ["Why and How Java Developers Break APIs"](https://arxiv.org/pdf/1801.05198.pdf), which studied a set of Java libraries for breaking changes.

>...we monitored 400 real world Java libraries and frameworks hosted on GitHub during 116 days. During this period, we detected 282 possible breaking changes

As part of the study, the authors attempted to contact the developers of the libraries to ascertain if the detected changes were actual breaking changes or not. Of those 282 potential breaking changes, 59 (39%) were confirmed by the library developers. (The rest remain unconfirmed and are still potential breaking changes.)

Thus, over a relatively short period of about 4 months, 59 confirmed breaking changes occurred in only 400 libraries. That is a considerable number.

## You Own Your Uptime

If I can conclude with anything it's that in order to manage complex systems (and apparently they are all complex) I believe organizations must be great at understanding and managing dependencies. To deal with complexity requires expertise in dependencies. Certainly we can try to push the risk onto vendors, but at some point we must take responsibility. You own your uptime.