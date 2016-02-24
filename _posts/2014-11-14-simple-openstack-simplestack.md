---
layout: post
title: SimpleStack
header_image: /img/simplestack.png
categories:
---

<small>_(simple diagram of...simplestack network setup)_</small>

# {{ page.title }}

For some reason I'm fascinated by the thought of simplifying an OpenStack installation. As a Linux/Unix systems administrator, I actually think OpenStack is great--it's like this huge ball of every Linuxy system and service packed together to provide useful infrastructure via an API. It's got everything: storage backends, database, queues, web servers, hypervisors, firewalls, image files, file systems, load balancers, plus all the networking components . You name it you can probably use it in OpenStack. Thus, as a systems administrator, you have to be able to use pretty much everything, and then automate it and operate it. From PXE boot to haproxy. It's a challenge that's for sure.

Because OpenStack can be so varied and pluggable it means a large number of choices. However, generally speaking, people don't want choices.

## Choices

I read about this paper once a few years ago: [Decisions and Revisions: The Affective Forecasting of Changeable Outcomes](http://www.wjh.harvard.edu/~dtg/Gilber%20t&%20Ebert%20%28DECISIONS%20&%20REVISIONS%29.pdf). In the study, students were asked to pick a photograph. One group had to pick it right away, and the other group could wait for up to four days to decide which print they wanted.

<blockquote>
...students believed that having the opportunity to change their minds about which prints to keep would not influence their liking of the prints. However, those who had the opportunity to change their minds liked their prints less than those who did not.
</blockquote>

I'm not a scientist of any sort, but that paper has always stuck in my mind as being an example of how having too many choices makes things hard for people, and in fact can lead to regret. This is part of why IT workers are always searching for the mythical "best practice," as in:

<blockquote>
How do I reduce the possibility of regretting what I did by narrowing my choices, and reducing the time it takes to make those choices?
</blockquote>

## Linux distributions

I'm typing this post on Ubuntu 14.10. I didn't build my own Linux Kernel or the entire system using [Linux From Scratch](http://www.linuxfromscratch.org/) (though maybe everyone should try that at least once). Most systems administrators and developers I've worked with haven't even heard of the [Linux File System Hierarchy](http://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard).

Linux distos are a way of reducing complexity; of making things simple. Linux distributions make all kinds of choices for us, especially with regards to packaging. When we use the OpenStack packages from Ubuntu 14.04 Canonical has made many decisions regarding OpenStack already. For most of us that is a good thing. (Though certainly many OpenStack operators build their own packages of OpenStack.) Choices are reduced, and for most, that's good.

## Too many features?

As I write this Amazon re:invent is going on. The size of AWS is difficult to grasp. It's so huge. It has hundreds of features and services. Even billing is extremely complex. Some people and organizations need these features. Some are intimidated by them.

With AWS complexity in mind, I can see how an infrastructure as a service (IaaS) provider such as [Digital Ocean](http://digitalocean.com) can be successful in a niche. They provide a vastly simplified system with few extra features:

- Simple virtual machine sizes and pricing
- Multiple regions across the world
- One shared internal network per region
- Snapshotting, copying snapshots to all regions
- DNS
- No volumes

That's it. I think over time they will add additional features in order to try to grow, but I hope they don't. I hope they stay in their niche and keep it as simple as possible. Nothing wrong with a successful company that doesn't grow by 20% year after year (except in the stock market).

## SimpleStack

I put together [SimpleStack](https://github.com/ccollicutt/simplestack) to use as an example of a "simple" OpenStack installation. I've got more work to do on it, so it's just an example, but a fun place to start, if you're into that kind of thing. :)

SimpleStack will setup and install one controller and two compute nodes using [Vagrant](http://vagrantup.com) and [Ansible](http://ansible.com).

I did recently use it to provide a company with a test installation of OpenStack on two hardware servers, a sort of Proof-of-Concept, so I know it can work in a Virtualbox/Vagrant as well as a hardware environment, though the playbooks would need a couple of changes to work in a non-vagrant environment (something I'll fix soon).

Notable features, or lack thereof:

- Automated, idempotent install using Ansible
- One controller, multiple compute nodes
- Front facing APIs using self-signed SSL
- Multihost FlatDHCP networking
- No private network (or private networks)
- No Horizon (so no web gui)
- No Cinder (so no volumes)
- Not using Neutron, instead nova-network

And that's it.

## Try it out

If you want to give it a test drive, just checkout the latest README file in the [Github repo](https://github.com/ccollicutt/simplestack). As I mentioned, there is more work to do so it's not perfect.

I like that I can see exactly how many actions it takes to setup a simple OpenStack system, 55 for the controller and 18 for the compute nodes. Though I'm using templates so that reduces a lot of the tasks.

<pre>
<code>curtis$ ansible-playbook site.yml
SNIP!
PLAY RECAP ********************************************************************
compute01                  : ok=18   changed=0    unreachable=0    failed=0
compute02                  : ok=18   changed=0    unreachable=0    failed=0
controller01               : ok=55   changed=0    unreachable=0    failed=0
</code>
</pre>

## Conclusion

This is a fun little experiment to see how much OpenStack can be simplified, and, in effect try to make it look something like Digital Ocean, though even SimpleStack is missing some features comparitively.

A few things I want to try out in the next "version":

- Separate the networks in the Vagrant config (all vms are on all networks at this point)
- Add a shared private network ala DO, [automatic floating IPs](http://www.sebastien-han.fr/blog/2012/08/03/openstack-auto-assign-floating-ip/) on the public interface for all instances
- Find a way to figure out the interface's purpose without using names like em2, eth0, ect, so that SimpleStack can work in any environment, not just Vagrant (that is, without changing the playbooks--there are a couple spots where "ansible_eth2" and the like is used, this is an Ansible variable issue I need to figure out)
- IPv6?
- A second region? Share glance images?
- A simplified API with a command line tool?
- Firewalling and routing with OpenBSD?

Let me know if you have any ideas or see a mistake. :)
