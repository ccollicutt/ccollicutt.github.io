---
layout: post
title: Moore's Law and the Datacenter
categories:
header_image: /img/paradox.jpg
---

# {{ page.title }}

I'm a big fan of [Packet Pushers](http://packetpushers.net) and their various podcasts. Recently I listened to a [podcast](http://packetpushers.net/podcast/podcasts/show-315-future-networking-pradeep-sindhu/) with Pradeep Sindhu who is the CTO and co-founder of Juniper Networks. I quite enjoyed the podcast, and while I think perhaps the hosts found Pradeep's desire to explain and define a baseline for everything a bit long winded, I enjoyed his obviously science-driven attitude towards discussing technology. We should all strive to be more precise.

Many people are aware that Moore's Law, which Pradeep redefined as an observation, not a law, is ending. For decades we haven't had to do much to get continual, massive (exponential) increases in computing power. But this is no longer. Instead, currently, we get access to more cores instead of faster CPUs. Clock speeds have maxed out. Now in order to achieve increases in computing power we have to look to parallelism, specialized hardware, and ways to deal with latency.

## Networking

What I thought was interesting in this particular podcast is that this also applies to networking technology. A good example that was given is the fact that a 100 GB network connection is actually 4 25GB connections which are tied together (1). So even in networking we have hit the end of Moore's Law and have to deal with parallelism, which is more complex. As Pradeep mentions in the podcast, most, if not all, applications will have to be (re)written to use parallelism, and those that do not will be left behind.

That said there does seem to be some debate if Moore's Law applies to networking at all, given it's really bounded by the speed of light, and that managing latency is going to be what is important. 

> ...the future of telecoms in general is firmly centred on managing latency due to contention between flows created by competing applications and users. This means scheduling resources appropriately to match supply and demand. - [Five reasons why there is no Moore’s Law for networks](http://www.martingeddes.com/think-tank/five-reasons-moores-law-networks/)

But then again, if we are going to use software to manage networks, then indeed that software will hit Moore's law.

I suppose the point is that in order to keep moving forward and gaining performance, networks will have to become smarter, and, likely, less predictable.

>The path forward requires that the bandwidth our networks provide becomes smarter. The networks themselves need to become programmable platforms. The infrastructure needs to be as real-time, flexible and dynamic as our smartphones have become. The answer to the problem of increased demand on the network is to flip that phrase around and evolve to what can be called network-on-demand. Network topology, connectivity, service class and quality of service all need to be on-demand services that can be customized to suit the needs of the end user. - [When Moore Is Not Enough – Why Our Growing Networks Require More Software](http://techonomy.com/2016/01/when-moore-is-not-enough-why-our-growing-networks-require-more-software/)

This reminds me of the requirement of 5G to have very low latency. In fact I believe the target is 1ms (though I don't see how that is possible, frankly).

>Physics stands a bit in the way of this. The speed of light and electricity is limited and in 1 ms, light can only travel around 200 km through an optical fibre. So even if network equipment does not add any latency whatsoever, the maximum round trip distance is 100 km. - [5G Latency - 1ms - Is it Possible?](https://www.linkedin.com/pulse/5g-latency-1ms-possible-akshay-mahajan)

Some of Pradeep's suggestions are discussed in a Network World article:

>The next step will be to use the network to interconnect multiple data centers with low latency connections. In this case, think of the cloud as a collection of pooled resources, like processing and storage, over geographic distance and being inter-connected with a high-performance network. In this scenario, its unlikely that network managers can continuously update the network fast and efficient enough to meet the needs of an environment that must scale over distance. This brings up step two of Pradeep’s Principle: network automation. This doesn’t mean simple automation like the provisioning of QoS or VLANs. Sindhu made it clear he was talking about a network that was almost autonomic and self learning in nature and could quickly adapt to any kind of environmental changes. - [Pradeep’s Principle: Give up on Moore’s Law and embrace automation](http://www.networkworld.com/article/3130229/software-defined-networking/pradeeps-principle-give-up-on-moores-law-and-embrace-automation.html)

Many network engineers are unsure if software defined networking (SDN) is really all that useful. Pradeep suggests that in order to increase performance we are not going to have much choice but to look to SDN, network automation and even machine learning. Things are going to get a lot more complex in the network.

## Hardware is more important than ever

Another interesting point made in the podcast is that hardware is more important than ever. Because we are nearing the end of Moore's law, and thus general computing is no longer receiving the easy gains of the last few decades, we will likely see more specialized, single-purpose hardware being developed. This specialized hardware can realize the large performance gains we require to move ahead in computing as a species. Thus, while in many ways we have moved to commodity hardware in the datacenter, it's quite likely we will also see a movement towards specialized hardware, especially in the realm of artificial intelligence, machine learning, and networking, among others. Certainly we see that already in terms of TCP offloading in network interface cards and chips like [Barefoot Networks](https://www.wired.com/2016/06/barefoot-networks-new-chips-will-transform-tech-industry/) P4 processor.

Further, given the recent advances in machine learning, there will likely be specialized hardware developed to increase performance in this area:

>Strong demand for silicon tuned for algebra that’s crucial to a powerful machine-learning technique called deep learning seems inevitable, for example. Graphics chip company Nvidia and several startups are already moving in that direction... - [Moore's Law is Dead: Now What](https://www.technologyreview.com/s/601441/moores-law-is-dead-now-what/)

## Co-processors 

As a simple systems administrator, I can see the value in "co-processors" and the easiest example I have is of a network card that is doing TCP off-loading or other intelligent network processing. One company working in this area is [Netronome](https://www.netronome.com/products/intelligent-server-adapters/overview/). 

These co-processors concern me a bit because that will mean a lot of the functionality will be embedded into the firmware of the network card, and there is not much worse for a systems administrator than firmware and it's related issues, but there is probably no way around that if we are to continue improving intelligence and speed in the network. To enable performance we give up the ability to easily change software. That will be more costly, however.

![By Altera Corporation - Altera Corporation, CC BY 3.0, https://commons.wikimedia.org/w/index.php?curid=6642224](/img/altera.jpg)

## FPGA

This is not an area I understand well yet. However, recently AWS released a developer preview where users can rent large FPGAs. It does seem that they are currently difficult to program.

>Amazon’s AWS cloud computing service today announced that it is launching a new instance type (F1) with field-programmable gate arrays (FPGAs). The promise of FPGAs is that they are essentially chips that you can reprogram on the fly and tune them for your specific applications, making them faster than traditional CPU/GPU combinations. - [TechCrunch](https://techcrunch.com/2016/11/30/aws-announces-fpga-instances-for-its-ec2-cloud-computing-service/)

## Paradoxes

These thoughts lead me to a couple of paradoxes:

**1) Hardware is more important than ever**

Most startups are based on software. Software is awesome. It's (relatively) easy and cost effective. But as discussed so far in this post, given Moore's law and our lost of easy performance gains with general computing, we will need specialized hardware.

Futher, there has been a rapid movement to commodity hardware which essentially means many x86 servers. Thanks to virtualization, most workloads are heading to x86. I don't expect to see AWS offer Power CPU based virtual machines, though Google always threatens with Power processors, most likely to be able to have some kind of bargaining...er chip...with Intel.

That said, perhaps we just need some new inventions to get us past the limits of silicon CPUs. Certainly novel CPU technology is inevitable, but will it be enough, who will do it, and when?

Also, co-processors have/will become much more important.

**2) Lower latency means datacenters and/or caching closer to end-users**

It seems to me that telecoms are [selling off datacenters](http://fortune.com/2016/11/04/centurylink-data-center/). Further, the public cloud is concentrating computing in fewer, much larger datacenters. If we are really going to be limited by latency, this seems to be a paradox. Perhaps there will simply be a point where we either can't get faster networking, or we don't need it. Then again people are still moving into cities, perhaps we'll just have an AWS region per large city and they will be large data caches. Isn't caching one of those hard computer science problems? :) That said, I believe Akamai is doing well.

## Conclusion

There is a lot happening in the IT world, and some of it is paradoxical. On one hand we have a serious movement towards the concentration of computing power in large, regional data centers (ie. AWS), but on the other the speed of light is a serious boundary and will require (some? most?) data and computing to be closer to end users. Further, software development gets most of the attention, but we are going to need considerable material and engineering breakthroughs in hardware to continue to increase computing performance. What's more, despite the movement towards commodity hardware (ie. x86 + virtualization), we will need specialized physical systems in networking and machine learning.

## Links

Here are a few random links that I've read while writing this blog post.

* [After Moore's Law](http://www.economist.com/technology-quarterly/2016-03-12/after-moores-law)
* [Barefoot Network's Chips Will Transform the Tech Industry](https://www.wired.com/2016/06/barefoot-networks-new-chips-will-transform-tech-industry/)
* [Wikipedia - 100 Gigabit Ethernet](https://en.wikipedia.org/wiki/100_Gigabit_Ethernet)
* [Google's Tensor Processing Unit could advance Moore's Law 7 years into the future](http://www.pcworld.com/article/3072256/google-io/googles-tensor-processing-unit-said-to-advance-moores-law-seven-years-into-the-future.html)
* [Google supercharges machine learning tasks with TPU custom chip](https://cloudplatform.googleblog.com/2016/05/Google-supercharges-machine-learning-tasks-with-custom-chip.html)
* [CenturyLink Reaches $2 Billion Deal to Sell Data Centers](http://fortune.com/2016/11/04/centurylink-data-center/)
* [Even if Moore's Law is "running out," there's still plenty of room at the bottom](http://boingboing.net/2016/07/01/even-if-moores-law-is-runn.html)
* [Five reasons why there is no Moore’s Law for networks](http://www.martingeddes.com/think-tank/five-reasons-moores-law-networks/)
* [The Death of Moore’s Law Will Spur Innovation](http://spectrum.ieee.org/semiconductors/design/the-death-of-moores-law-will-spur-innovation)
* [AWS Developer Preview of FPGA](https://aws.amazon.com/blogs/aws/developer-preview-ec2-instances-f1-with-programmable-hardware/)

## Footnotes

1) [4 25 gigabaud lanes](https://en.wikipedia.org/wiki/100_Gigabit_Ethernet)
