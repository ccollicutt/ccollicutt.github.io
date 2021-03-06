---
layout: post
title: OpenStack 2012 Summit Day &#35;4
categories:
---

# {{ page.title }}

## Security Track!

Having been a security administrator at a medium sized University; having taking fairly extensive education in Information Security; having attended security conferences like CanSec West and H.O.P.E I am happy that all of Thursday has a dedicated security track. Unfortunately I can only attend a couple of sessions, but at least I can do that much. :)

## Creating an OpenStack Security Group

Bryan D. Payne from Neubula and Robert Clark from HP talk about the need for a OpenStack Security Group. The [OSSG](https://launchpad.net/~openstack-ossg) is "hiring", ie. need volunteers especially security engineers, technical writers, and security experts that operate OpenStack clouds.

Computer Security
- Design for security from the start
- Understand your threats
- Understand your goals
- Pervasive security culture (not just "that paranoid guy has it under control")

Security Challenges for OpenStack
- Security as an afterthought
- Security as silos
- Security by non-experts

There was some mention of the [Cloud Security Alliance](https://cloudsecurityalliance.org) which Bryan is a co-lead on a workgroup for and that the CSA is a more oriented to high-level theory whereas the OSSG will need to be directed at applied security.

## Delivering Secure OpenStack IaaS for SaaS Products

_"Everyone has a great plan until they get punched in the mouth"_--Mike Tyson

Andrew Hay, is the Chief Evangelist at CloudPassage, Inc. where he serves as the public face of the company and its cloud server security product portfolio. 

Three places to "put security" in OpenStack
- Quantum
** Coolest thing about Quantum is the ability to inject 3rd party products
- Keystone
- Nova
- Security groups/firewalling with iptables
- VLANS
- Initial configuration of nova

Issues in Cloud Security
- Host security
- Security of images
** Network-based security is only so good in multitenant cloud
- The ultimate target is the endpoint--so secure it
- Cloud servers are more exposed
- De-provisioning of servers releases public IPs that might have the wrong firewall rules

He then went over some basic host security concepts, quite a few of which I disagree with, or at least he wasn't able to really go into depth with what he actually meant by them. 

Generally speaking I think host security has been reduced mostly because in OpenStack most people use images that have already been provided to them and don't run any specific configuration management (ie. chef/puppet/[ansible](http://ansible.cc/)) "hardening" processes. 

Nor do people usually build their own OpenStack images, which I think is an important skill to have. Though if more OS vendors provided base OpenStack images things would be a little easier. As far as I know, at this time, only Ubuntu publishes OpenStack compatible [cloud](http://docs.openstack.org/trunk/openstack-compute/admin/content/starting-images.html) [images](http://cloud-images.ubuntu.com/).

## SDSC

Unfortunately, or fortunately, depending on your perspective, that was all the OpenStack security talks I was able to attend on Thursday because I had to go to a meeting at the San Digeo Super Computing Center, or the [SDSC](http://www.sdsc.edu/), and wasn't able to come back to the summit.

I was excited to go to find out more about what they are doing, especially because they have one of the largest [OpenStack Swift installations](https://cloud.sdsc.edu/hp/index.php) in the world. I think about five petabytes worth.

They were also very excieted about their new supercomputer [Gordon](http://www.sdsc.edu/News%20Items/PR030512_gordon.html), which has a ton of SSD storage (ie. flash--get it...gordon + flash).

Also they had some old tape systems they were retiring. Tape is not dead yet though!

![](https://raw.github.com/ccollicutt/ccollicutt.github.com/master/img/sdsc_tape_silo.jpg)
_(old tape silo being retired; sad to see it go)_

