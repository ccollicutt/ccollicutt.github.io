---
layout: post
title: Service Providers and Telecoms Must Accept Complexity
categories:
header_image: /img/bridge.jpg
header_permalink: https://unsplash.com/photos/F9o7u-CnDJk
---

# {{ page.title }}

## tl;dr

NFV is complex. But in order to automate we need APIs. Something has to schedule compute, network, and storage. Something has to provide the APIs to orchestrate it all. That something is NFV. Telecoms can't give up so easily on NFV, because to do so would be at worst a rejection of automation and at best a renaming of NFV.

## Accepting NFV Complexity

I've been working with OpenStack and SDN for a long time. As well, for the last 2+ years, I have also been working in the area of Network Function Virutalization (NFV). At this point NFV is a bit of a catch all term for all the new things that have been happening in and around the service provider or telecom industry. Automation. SDN. 5G. All of these things are complex and perhaps complicated.

OpenStack has found a place in telecoms. I think it's fair to say that every telecom in the world is thinking about OpenStack in some capacity. We are at the point now, I believe, where many telecoms are past the proof of concept phase and are really starting to try to put OpenStack, and more specifically NFV infrastucture (NFVi), into production. However, this is the point where we will also start to see telecoms give up on NFV (and SDN) because it's too complex.

Telecoms are good at deploying physical boxes and manually deploying required applications on top of them because they heavily rely on the vendors to do it. If a telecom buys a piece of software from a vendor, they then put the vendor through the wringer to get them to deploy it and ensure it works. Unfortunately, this is "set and forget."

In the case of NFV, however, it's not just a single vendor that is involved, and what's more, telecom staff are being asked to understand how NFV works and to be able to deploy network functions into NFV infrastructure via automation (eg. ETSI MANO). Instead of relying on vendor professional services internal telecom staff are being asked to deploy network functions. But doing that is not as easy as beating up on a vendor. It's at this point we will start to see a rejection of NFV as operationalizing NFVi, SDN, and automation is challenging. Not impossible, just challenging. (Aside: the real issue often in telecoms is the complexity of the organization, not the technology.)

## Rejection of NFV?

I read Tom Nolle's CIMI Corp blog quite a bit. I enjoy his take on telecoms and NFV. He has a recent post entitled [Have Operators Uncovered a New Transformation Path?](http://blog.cimicorp.com/?p=3375) from which I take the following quote.

>The leading technologies in transformation, according to popular wisdom, are SDN and NFV.  Operators have been griping about both in both conversations with me and surveys I’ve done.  SDN, they say, is too transformational in terms of the network technical model.  Central control of forwarding, the classic ONF OpenFlow SDN model, may or may not scale far enough and may or may not handle large-scale faults.  NFV, they say, is simply too complicated to be able to control operations costs, and its capital savings are problematic even without opex complications.

I have some issues with the above statement. (Again, I like Tom's work, I just have an issue with this particular post.)

First, I don't think anyone really wants a "OpenFlow" model of SDN. I prefer to think of OpenFlow as an academic project that furthered our thinking about SDN, not as something we would actually want to put into production. We need SDN, absolutely, but SDN != OpenFlow.

The second statement I disagree with is that "NFV is too complicated." We need automation to drive down costs. In order to automate we need APIs. And what APIs? OpenStack has the defacto NFVi API. In order to automatically deploy network functions we need something to schedule compute, storage, and network resources. That something is NFV. The vast majority of NFVi will probably be based on OpenStack.

If we were to reject NFV all that would happen is that the abstraction of compute, storage, and networking would be hidden by another API (example Mesosphere perhaps?) but that does not avoid the complexity, it just gives it another name or is another product. We can create as many abstractions as are necessary to provide the appearance of reduced complexity, but somewhere there is an NFVi, and somewhere there is SDN, and these things must be well understood and operationally perfect, and that is challenging. But not so challenging that we have to give up on NFV, because giving up on NFV means giving up on automation and staying with the status quo of *manually* deploying applications onto physical boxes and *manually* wiring them up.

My suggestions to telecoms would be to understand that some complexity is unavoidable and to allow their staff to get up to speed on operationalizing management of NFVi and deployment of NFs with the understanding that it is definitely harder to do than a one-time, static, unrepeatable deployment handled by a vendor. If you want to see complexity try altering one of those deployments a year down the road. ;)
