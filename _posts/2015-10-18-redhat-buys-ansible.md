---
layout: post
title: Thoughts on Redhat buying Ansible
categories:
header_image: /img/glitch3.jpg
header_permalink: https://www.flickr.com/photos/chrisdlugosz/4998710483/in/photolist-8BHGtv-7At8vF-bN2KqR-qLDTuE-8zfHw2-5r9xUT-bna5cv-7qGcRN-6S2Dve-6S5CJy-3URfoL-8uH7Qh-dRJzan-7mQL1U-aHDiNv-9kVfM8-4fkpfV-4rr1QZ-aHDiq8-aHDhCr-aHDj9D-7KSrvF-ancz34-aCesrE-8iUc9e-J8D5Q-aG4TY4-b6jHZz-ancz4z-6Ee2QN-aG4wne-52tnPy-bUkqxd-feQ3kg-a17CWM-8zBMtA-ctS8rQ-7zLFta-aGmrkD-aGmqbe-dVN3cF-kESkA-b6jHQc-gaEgaV-dMKrAQ-aHDoVP-dvTDGJ-CEe8E-aGmsTe-aGmquZ
---

# {{ page.title }}

Recently Redhat bought Ansible. Most people went "yay!", but I, as the curmundgeonly sysadmin, said "that sucks...for me."

## The "exit"

People seem happy that Redhat bought Ansible--there were many congratulatory tweets. It appeared that, at least on the surface, most thought it was a good exit for Ansible. $150 million. Pay back those VCs, and slip back into the comforting arms of old softy Redhat. 

However, I don't think it's good for me, and I think that's an Ok position to take--as a long time user I think I should get some kind of pass. The company I work for also purchased Ansible Tower so we weren't just using it for free either. Also I think it's the only open source project that I can actually say I contributed some code to (even if that code was only used by myself).

There are different views on how companies exit. I think most people would consider me contrarian. To that point, I don't know if companies should exit. What does that even mean? In startup terms, I think exit usually means either go IPO, or get bought out--both so that you can repay the venture capitalists. However, those are not the only options for a company. In fact, despite not being an economist by any stretch of the imagination, I feel like there is a lot of value in our economies that is made up of small or medium sized businesses--companies that aren't part of some conglomerate "ecosystem," ie. sales machine.

Puppet and Chef are still individual organizations. However, it seems like at some point they too will be bought up by a larger conglomerate like, I guess, EMC/Dell, uh, IBM?, Cisco--companies that many people think are [doomed](http://www.wired.com/2015/10/meet-walking-dead-hp-cisco-dell-emc-ibm-oracle/). Or I suppose their real goal is to IPO. Ugh.

> HP. CISCO. DELL. EMC. IBM. Oracle. Think of them as the walking dead. Oh, sure, they’ll shuffle along for some time. They’ll sell some stuff. They’ll make some money. They’ll command some headlines. They may even do some new things. But as tech giants, they’re dead. ([wired](http://www.wired.com/2015/10/meet-walking-dead-hp-cisco-dell-emc-ibm-oracle/))

Does Redhat fit into that list as well? If not, why not? Is Redhat a niche company or is it the open-source EMC? Or is it something completely different? (Unlikely.)

As of this moment, Redhat has a market cap of about 14 billion. Their PE is about 74. I think they are over-valued, otherwise I'd actually own their stock. This [article](http://techcrunch.com/2014/02/13/please-dont-tell-me-you-want-to-be-the-next-red-hat/) talks a bit about the problems with open source companies. Redhat is a one-off situation. I don't know if we'll see something like Redhat again for some time. For whatever reason, the business/finance world is much different than the innovation world. So we end up with companies like EMC and Redhat kind of fuddling along, extracting boring cash for newish systems from enterprise companies, who for the most part are moving to AWS, or other public clouds. (Great, now I've confused myself.)

## Redhat acquisitions

There's a nice list of acquisitions made by Redhat [here](https://en.wikipedia.org/wiki/Red_Hat#Mergers_and_acquisitions). Their model seems to be to pick up small companies that produce valuable open source systems, and to do so for about 100 million. In that respect I don't see them picking up anything else I use, other than perhaps haproxy. MariaDB would probably be too expensive for them. OpenStack is run by a foundation so you can't really buy it, though you can buy expertise, just like Redhat did with eNovance. Codership, who I believe makes the galera component of MySQL/MariaDB (I may be wrong) might be something good to look into.

Examples:

- JBoss - Which they bought for $420 million in 2006 (I've never used it)
- KVM
- Gluster
- Ceph/Inktank
- eNovance (OpenStack consultants)
- Ansible

KVM is probably the most important piece of software they've "picked up" and I have to admit it seems to be doing fine. It's what I use with Ubuntu. Gluster and Ceph also seem to be doing fine too. eNovance I only added because it is OpenStack related. So just based on that list perhaps I shouldn't be too concerned.

## Configuration management tools

I've used Chef and Puppet enough to know that I don't enjoy using them. I have preferred Ansible and run a large OpenStack system exclusively with it. However, it certainly has limitations, as does every other configuration management (or automation? or orchestration?) system. I'm not super happy with any of them, though I definitely feel like Day 1 (ie. the installation) is easy, but what about day 40? Or day 200? Ansible seems a better fit for actually operating a system over time. For example, I'm not clear on how one would migrate a bunch of databases from one cluster to another using Puppet, but then again I haven't looked too hard. Puppet seems like the Java of configuration management. But I digress...


## Inability to compete

As a company, Ansible is quite small. I believe about 50 people. I'm sure Puppet has more than 300. (Aside: I was in the Puppet office once. It's the nicest office I have ever been in. Full kitchen. Beer. Presentation space. It was way too nice. It was spooky.) Presumably Chef is similar in size and funding to Puppet. They are bigger than Ansible and better funded (to what end, I don't know). They have sales teams with white boards and quotas; perhaps dreams of an IPO.

Funding (pulled from [Crunchbase](http://crunchbase.com):)

- Chef - $137 million in 6 rounds
- Puppet - $85 million in 5 rounds
- Ansible - $6 million in 1 round, then bought for about $150 million
- SaltStack - $685K in a seed round (probably a good deal--someone buy them)

So Puppet and Chef are big. Puppet is probably working on another round of funding. They are both putting out features like nobody's business, with Puppet releasing some kind of orchestration engine and Chef with a continuous delivery system.

Basically, I'm sure Ansible felt much too small and underfunded to "take on" Puppet and Chef. Or perhaps they didn't feel like they had the staff to keep up with all the changes coming into Ansible from the public, ie. they couldn't merge all the pull requests in a timely fashion. Either way, too small. Not enough cash to burn on hiring for the short term as Ansible matures as a CM. Which is too bad, because I think they should have found a way to stay small, but that may not be possible with an open source license.

## The future of CM

My feeling is that CM products aren't doing a great job. That we need something different, and that it might be at the OS layer. But we're so entrenched with the OS that it will take a long, long time before anything changes there. Perhaps AWS' Lambda is closest to something really new, something really different. Even though I've been managing servers for over a decade, I don't think all these servers is a good thing--it seems like a waste of resources, not only to run them but also to manage them with a CM. Sure, we have all these new container scheduler systems, but I don't think that's the endgame. Containers are great but virtual machines are only 2% overhead anyways, so where's the gain? Boot up time? Who knows.

## In the end

I do think Redhat managed to obtain a great company for a reasonable amount of money (considerably less than what VCs will be trying to extract from Puppet/Chef). So it will be good for Redhat. However, I don't think it will be good for me, as a long time Ansible user. I'm sure Redhat will put more resources (especially sales, lol) towards Ansible, but I imagine a lot of those resources will be going towards integrating Ansible into the Redhat stable of products, and not necessarily towards making Ansible better for people like myself, who, right now anyways, don't use any Redhat products.

Further more, if Redhat is such an unusual company, and with AWS absorbing enterprise IT like a black hole, and huge sales organizations like EMC failing, soon Redhat itself will crumble. Which is scary.

Finally, one has to wonder how open source makes money. I need to do some research into smaller, private companies that provide open source software. I have a feeling there are quite a few software-as-a-service companies that are doing Ok, but otherwise...it's complicated. Just based on my current investigation, I don't think I would start a company that completely open sources its product, nor would I aim for exiting to pay back venture capitalists. Frankly I don't see the issue with small, non-public companies, and perhaps I'm projecting that onto Ansible. Hashicorp will be one to watch, though they may be an aberration.

Anyways, looking forward to Cloudforms (whatever that is) integrating with Ansible, and also looking forward to exploring other methods of system automation.
