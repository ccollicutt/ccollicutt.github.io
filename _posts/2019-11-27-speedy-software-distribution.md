---
layout: post
title: Speedy Software Distribution
categories:
header_image: "/img/speedy-software-distribution.jpg"

---

# {{ page.title }}

I would wager most, if not all, developers have edited a file on a remote server to update an application. Those were the days. Now the spectrum of software distribution ranges from manually copying files, to container images, to bit-torrent and more.

## The Current Standard: Container Images

[Container images](https://www.opencontainers.org/) have vastly affected how we distribute software. Given the desire for many organizations to use [Kubernetes](https://pivotal.io/platform/pivotal-container-service), which relies on containers, which in turn rely on container images, the model of distributing singular binaries of an OS file system has become the default modern day standard.

While in the past we may have used simple things like copying and rsync-ing files, operating system packages, even bit-torrent, we now strive to distribute OCI images.*
We're All Distributed Programmers Now

Cornelia Davis, CTO Pivotal, has [this to say](https://devclass.com/2019/08/16/pivotal-cto-kubernetes-means-were-all-distributed-systems-programmers-now/) about modern day software development:

>"...we’re all distributed systems programmers now. When I started my degree 30 years ago, that was niche. Now, in the cloud, everything is distributed..."

The prevalence of distributed systems means that we need to get software running on >1 application instance. Maybe thousands. How can we quickly, efficiently, and accurately distribute software to thousands of servers running around the globe?

## The Need for Speed

I believe speed is key to the future of software distribution. How container images, or some other package, get from host is as important as how those packages are defined. I would like to have my software running globally in hundreds of milliseconds.

One doesn't have to look much further than [Uber's Kraaken](https://eng.uber.com/introducing-kraken/) project to see what might be involved in speeding up software distribution.

Kraaken is a peer to peer Docker registry:

> Docker containers are a foundational building block of Uber’s infrastructure...but as the number and size of our compute clusters grew, a simple Docker registry setup with sharding and caches couldn’t keep up with the throughput required to distribute Docker images efficiently.

Another interesting and speedy software distribution component is [Chrome's Courgette](https://blog.chromium.org/2009/07/smaller-is-faster-and-safer-too.html) (from 2009 no less). Courgette can be 10x more efficient than something like [bsdiff](http://www.daemonology.net/bsdiff/) (which itself is extremely bit thrifty). Courgette does not directly help container images, but it represents an efficient way to create a binary delta. Smaller deltas mean faster distribution. Faster distribution can mean more frequent updates.

> If the update is a tenth of the size, we can push ten times as many per unit of bandwidth.

In that post they show the improvement in data transfer using Courgette.

```
Full update: 10,385,920 bytes
bsdiff update: 704,512 bytes
Courgette update: 78,848 bytes
```

700k to 78k...that's a an impressive improvement resulting in many bits that don't have to traverse the network (and again, [bsdiff](https://pdfs.semanticscholar.org/4697/3fb8c3b038648e1fe848a72275a75ff49fd2.pdf?_ga=2.252430473.1684130984.1567251025-566867337.1566753693) is extremely efficient). For those of you that would like more information on the Courgette model, there is considerable research in the area, going back at least to [exediff](https://robert.muth.org/Papers/1999-exediff.pdf).

## What's Next

At this time I believe there are at least three major things to think about when thinking about distributing code:

* Peer to peer deployment (eg. Kraaken)
* Advanced delta encoding (ala Courgette and exediff)
* Improvements in code injection / application restarts

This post has only, just barely, grazed the surface of software distribution. As we continue down the distributed programming path and focus on getting obstacles out of the way of developers, we will find that distributing applications at insanely fast speeds will be a key enabler.

---

\* *Of course these binary images are made up of things like OS packages, but lets not think about that too much right now. Also, what we are really distributing is the layers of these images.*