---
layout: post
title: What is StarlingX?
categories:
header_image: "/img/starlingx.jpg"
header_permalink: "https://unsplash.com/photos/b7MZ6iGIoSI"

---

# {{ page.title }}

I recently joined the Technical Steering Committee (TSC) of the open source [StarlingX](https://www.starlingx.io/) project. As one of the members of the TSC, I work to enable the governance model of the project, which is part of the OpenStack Foundation. Good open source governance is important, but it's not the most interesting part. :)

## What is StarlingX?

StarlingX, which I usually abbreviate to STX, is an open source project that provides a customized Linux distribution which delivers a fully managed Kubernetes and OpenStack deployment on one, two, or more physical servers. It's designed to run mission critical workloads on the edge.

If you are working in the world of Network Function Virtualization (NFV), "Industry 4.0", Internet of Things (IoT), and other similar areas, STX would likely be interesting as it presents a way to provide de-facto standard infrastructure APIs in edge locations--locations where it may only be feasible to have one or two physical servers. STX can manage hundreds of compute nodes if needed, but for the edge single and dual modes are the most compelling.

## OpenStack Foundation - More than OpenStack

StarlingX is open source project that is supported by the OpenStack Foundation. The seed code came from Wind River's Titanium product. Going forward, Wind River Titanium will be the downstream commercial based on the upstream open source code. Of course, as open source software, anyone is free to take the code and artifacts generated and deploy it or build it into their own product.

For those of you that don't know, the OpenStack Foundation provides structure for more open source projects than OpenStack. At this time there are four other major projects within the OpenStack Foundation:

* [StarlingX](https://starlingx.io) - Edge OpenStack and Kubernetes
* [Zuul](https://zuul-ci.org/) - CI/CD and project gating
* [Kata Containers](https://katacontainers.io/) - Virtual machines instead of containers
* [Airship](https://www.airshipit.org/) - Declarative OpenStack and Kubernetes automation

## Features of STX - Designed for the Edge

STX does a lot of things, but I think the most important features that help to differentiate it are:

* It can run on a single physical server and still provide Kubernetes and OpenStack APIs. This is called "simplex" mode.
* It can run on two physical servers, providing a highly available version of Kubernetes and OpenStack. This is called "duplex" mode.
* It runs OpenStack on top of a fully managed Kubernetes deployment--OpenStack is just an app in k8s.
* StarlingX has made design choices and created specialized software services and features designed specifically to run mission critical workloads. High availability. Integrity. Safety. These are words the project's values.

(Note that STX has many other features and integrated systems that I haven't mentioned here in an attempt to keep this post short.)

## Open Source Software - Help Wanted!

Much of the work going on in the project now is to upstream the seed code into various open source projects, from the Linux kernel to OpenStack Neutron. However, StarlingX is also advancing and adding new features and functionality. A good example is the Kubernetes integration work occurring right now.

If OpenStack, Kubernetes, and new technology paradigms such as the edge, NFV, IoT, and others are of interest to you, I heavily suggest getting involved in the project. Have a look at the community page on the website and get in touch if you'd like to participate. Or, if your organization would like to run a proof of concept or has questions about the project, reach out and let us know.

![Starlingx Logo](/img/starlingx-logo-1.png)
