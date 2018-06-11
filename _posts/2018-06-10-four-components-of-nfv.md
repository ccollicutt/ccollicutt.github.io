---
layout: post
title: The Four Major Components of NFV
categories:
header_image: /img/four.jpg
header_permalink: https://unsplash.com/photos/dacj4UrnT3k
---

# {{ page.title }}

Network Function Virtualization (NFV) is a broad and complex topic. However, I think there are four major components to NFV and they are as follows:

1. NFV Infrastructure
2. Softwarization of the network and SDN
3. Management and Orchestration
4. Deploying Network Services

Certainly the above is a simplification, but I think that these four pieces are the underlying structure of NFV.

## NFV Infrastructure

This is probably the best defined at this time, because, generally speaking, OpenStack is the defacto NFVi and Virtualization Infrastructure Manager (VIM). It's not the only option, but it's the most common.

Regardless of whether it's OpenStack or something else, there has to be a system that can act as Infrastructure as a Service (IaaS) with APIs and schedulers and the like--something to manage compute, network, and storage, as well as other more advanced NFV resources (GPUs, Numa, etc).

ETSI is doing an admirable job of working to standardize the NFVi and VIM (as well as other areas, of course).

## Softwareization of the Network and SDN

While some people believe that SDN has failed, I'm not one of them. If we only consider SDN as a centralized controller that uses OpenFlow, then perhaps failed is correct (though I usually suggest that OpenFlow was an academic effort). But that is not the only way to define SDN.

Regardless of the definition of SDN, to be successful with NFV we need the ability to automate the network, and the only way to do that is to have access to APIs that talk to schedulers and resource managers to control network components. When we talk about the NFVi, or IaaS, we often say that it will control compute, storage, *and networking*. And it does. However, we need to step outside the boundaries of the NFVi into the rest of the network--including the core network. We need to manage as much of the network as we can as though it is software...APIs, libraries, dependencies (ugh) , and code.

Thus we end up with the somewhat clumsy phrase "softwareization of the network." Clumsy but appropriate and workable. To be successful with NFV many traditionally separate network domains need to be combined together into a global interface.

Certainly implementation of this concept will be challenging. Traditional, legacy networks will push back on any kind of softwareization, but, in my opinion, it has to happen to some degree in order to be able to deploy network services successfully.

Unfortunately it does not seem like this is an area that can be standardized, at least not at this time. While networks tend to do the same thing, they all seem to have been designed differently, and may be unique.

## Management and Orchestration

This is another area that ETSI is working to [standardize](http://www.etsi.org/deliver/etsi_gs/NFV-MAN/001_099/001/01.01.01_60/gs_NFV-MAN001v010101p.pdf). What we need is some kind of higher level system that can talk to all the other APIs that we have made available (VIM and the "softwareized network") to actually be able to deploy network services that customers can use (and pay for).

Currently it is exemplified by the rather complex system called [ONAP](https://www.onap.org/)--the Open Network Automation Platform. There are other similar systems, such as ETSI's own [OSM](https://osm.etsi.org/) and many commercial products as well.

By using a management and orchestration (MANO) system, we can abstract away the underlying complexity of the NFVi and network and move towards the real work of actually (for real) deploying network services.

## Deploying Network Services

Once we have the NFVi/VIM, a softwareized network, and a management and orchestration system (or systems), we can finally get to the work of onboarding VNFs, building network services, and selling them to customers.

However, it's important to note that this is not a layer that one can simply purchase. The other three layers are infrastructure. They are systems to be deployed, and theoretically could be completely outsourced. This final layer is where all the actual work should occur--all the investment in the other layers is to service this one.

This layer is effectively a form of application development. I don't want to go into too much detail in this particular post, but typically onboarding a VNF or Network Service requires writing some code (eg. TOSCA). Even if the code is provided by vendors or from open source, each Service Provider (SP) will have to do some customization of this code, as well as do their own testing and validation of the VNFs and Network Services it defines. Further, SPs will need to develop workflows for deployment of services that match their historical corporate organizational structure. While overall every SP is the same, the way they approach that sameness is different and often (unfortunately) unique, much like their network and security policies.

## Conclusion

The first three layers are infrastructure. They need to be commoditized and automated as much as possible in order to abstract away their complexity. Once those layers are available and trusted, the hard work to deploy customer facing network services can begin.
