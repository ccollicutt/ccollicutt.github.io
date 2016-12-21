---
layout: post
title: What OpenStack Distros?
categories:
header_image: /img/destro.jpg
---

*Oops that's Destro not Distro. Note the OpenStack logo though :)*

# {{ page.title }}

On the surface it seems like there are many ways to install an OpenStack cloud; many [OpenStack distributions](https://www.openstack.org/marketplace/distros/). 

OpenStack requires services and infrastructure and the only way to get those all co-oridinated is to use some kind of automated tooling, which is typically what these "distros" provide.

However, my definition of "distro" may not line up with most. Essentially I would consider a "distro" a OpenStack installer, and, more importantly something that helps manage OpenStack *after* the initial installation. Often the installation process is called "Day 1" and managing OpenStack after the installation "Day 2." It's not enough just to install OpenStack, distros need to provide Day 2 help as well.


## tl;dr

There are way fewer OpenStack distros than one would think. Also, even if you have an OpenStack distro you will *still* need internal staff and/or professional services from the distro vendor. I'm aware that is not that helpful a conclusion. :)

To quote [Boris Renski of Mirantis](https://www.mirantis.com/blog/infrastructure-software-is-dead/):

>Everybody’s OpenStack software is equally bad

But let's go through this process anyways... :)

## Official Distro List

OpenStack provides a [list of most of the distros](https://www.openstack.org/marketplace/distros/). Here's what is listed on there as of today:

* Stratosphere
* Appformix
* Dell EMC RedHat Solution
* StackBuffet
* Cisco Metacloud
* HPE Helion OpenStack
* T2 Cloud OS
* Hyper-C
* Enovance Service Provider Cloud
* Suse OpenStack Cloud
* H3 Cloud OS
* Dell EMC VXRack with Neutrino
* Aqorn Thunder
* Redhat OpenStack Platform
* Platform9
* Ubuntu OpenStack
* Bright Computing
* Mirantis OpenStack
* Oracle OpenStack
* TransCirrus Cloud Appliance
* Mirantis
* IBM Spectrum Scale for Object Storage
* AWCloud OpenStack Distribution
* Oracle OpenStack for Oracle Linux
* RackSpace Private Cloud
* Vmware VIO
* EasyStack ESCloud
* Animbus CloudOS
* Huawei Fusionsphere OpenStack
* Ultimum Cloud Platform
* IBM Cloud Manager with OpenStack
* ZTE TECS OpenStack
* Debian 
* SwiftStack
* Breqwatr Cloud Appliance

That seems like a long list with 35 entries; let's go through it, examine the entries and see what can, and should, be removed. 

**Not Distros**

A few are not OpenStack distros: Appformix (which was just bought by Juniper and is really a metrics system), IBM Spectrum (just object storage), SwiftStack (also just object storage), as well as Debian (which provides invaluable packaging, but not actually an installation and/or management system). I'm a big fan of Swift and thus SwiftStack, but it is not a full fledged OpenStack distro (ie. does not provide compute, networking).

**Defunct**

Then there are a few that are essentially defunct: eNovance was bought by RedHat some time ago, and Suse just bought HPE Helion. Oracle just cancelled Solaris. I think IBM Cloud Manager is dead too.

**Biased Removals**

A few I am going to remove just because I don't see them as real entries, such as Oracle and VMware VIO, also the VXRack entry. Add them back in if you prefer, best of luck. :)

**APAC**

Some of these companies I just can't find much information about, mostly because I am not familiar with APAC companies or languages. I wish I knew more. T2 Cloud. H3Cloud OS. AWCloud. Animbus Cloud. EasyStack. If your company is an APAC or works with APAC customers, then perhaps one of these entries would be a great choice. I would certainly be interested to know how these companies OpenStack distro works.

**Small Companies**

Next up are small companies. I think it would be very difficult to recommend a small company for an OpenStack distro as who knows if they will be around in a couple years. I suppose the same can be said for larger companies, given HPEs sad retreat. Bright Computing (Amsterdam). Aqorn (US). Ultimum (Prague). 

Further, many companies, large and small, actually use someone elses distro, eg. RedHat or Mirantis.

**Appliances**

Personally, I would toss any appliance vendor. What is an appliance in 2016? It's an x86 box with Linux on it. Which is what you would deploy the OpenStack control plane on anyways. So out go Breqwatr and Transcirrus (both of which I'm guessing are quite small companies). Mirantis Unlocked Appliances isn't even a distro, it's just certified hardware for the Mirantis plaform, not sure why it's on the list at all.

**Solutions**

Some options are "solutions" as opposed to distros. Example Dell + EMC + RedHat.  Presumably I could put my own solution up, say Mirantis + SolidFire + OpenDaylight or any combination of OpenStack, Storage, and SDN. Not a distro.

**Unclear**

I'm not sure what Aptira StackBuffet is, but based on a quick look it seems to be a system to generate OpenStack packages for the operating system layer. I would not consider that a distro. I'm sure Aptira provides OpenStack clouds, but I'm not sure how.

**Overspecialized**

I'd put Hyper-C into this category as it seems to be directed at Windows only shops. Presumably you can run any workload on their distro, but it does seem to have a specific marketing orientation. I would put Stratoscale here as well as they are a hyperconverged entry, apparently with their own hypervisor. 

## So what are we left with?

* SUSE/HPE Helion
* Cisco Metacloud
* RedHat OpenStack Platform
* Platform9
* Ubuntu OpenStack (does this mean BootStack?)
* Mirantis
* RackSpace Private Cloud
* Huawei FusionSphere
* ZTE TECS OpenStack

Before going further with that list, lets checkout some open source "distros."

## OpenSource Distros

The OpenStack project hosts several distros, or libraries:

* [Puppet OpenStack](http://docs.openstack.org/developer/puppet-openstack-guide/)
* [OpenStack Ansible](http://docs.openstack.org/developer/openstack-ansible/)
* [OpenStack Kolla](http://docs.openstack.org/developer/kolla/)
* [Tripleo](http://docs.openstack.org/developer/tripleo-docs/)

Puppet OpenStack is probably the most commonly used of the three I mention above, but it is more like a libary of Puppet manifests that can be used to deploy OpenStack. However, you need to create your own "composition layer" that will actually utilize those manifests. Thus I would not consider it a distro, though again, it is commonly used and well tested, it just requires some additional work.

OpenStack Ansible and Kolla are distros though. OpenStack Ansible is quite mature and stable, and is also, I believe, the basis for the RackSpace Private Cloud. Koll, on the other hand, is relatively new. It is getting a lot of press, so to speak, because it uses Docker images as a deployment mechanism, and, of course, it is also embroiled in the k8s hype. 

I am quite familiar with Tripleo. It is the basis for RedHat's Director installer. I am not it's biggest fan, but it does work. It uses the aforementioned OpenStack Puppet as well as Ironic and Heat to deploy OpenStack. I personally have a concern with Day 2 and Tripleo, however.

Also, and this might be lesser known, the Open Platform for Network Function Virutalization (OPNFV) project has a [few installers](https://wiki.opnfv.org/display/bgs/Installers). Some are based on existing systems like Tripleo. I don't know if you could use any of these in production, as they are mostly meant to be part of OPNFV's continuous integration system, but they do exist.

## Conclusion

Whether you agree or disagree with my inclusions and removals, I think it's safe to say that there is not a massive number of OpenStack distributions, and the ones that do exist are quite different in what they support and how they work after Day 1.

But...what distros do we have left?

* SUSE/HPE Helion
* Cisco Metacloud
* RedHat OpenStack Platform
* Platform9
* Ubuntu OpenStack (does this mean [BootStack](https://www.ubuntu.com/cloud/openstack/managed-cloud)?)
* Mirantis
* RackSpace Private Cloud
* Huawei FusionSphere
* ZTE TECS OpenStack
* OpenStack Ansible
* OpenStack Kolla
* OpenStack Tripleo

I don't know anything about Huawei Fusionsphere or ZTE TECS OpenStack. They may very well be based on an existing distro. Can't comment.

Platform9 is a unique entry due to its SaaS design, which may not be suitable for all deployments, but I do think it is quite innovative.

RackSpace Private Cloud is based on OpenStack Ansible.

RedHat OpenStack Platform is based on Tripleo. 

Mirantis is/was a "pure-play" distro which is now [moving quickly](https://www.mirantis.com/company/press-center/company-news/openstack-kubernetes-mirantis-collaborates-intel-google/) to use Kubernetes to manage the OpenStack control plane. (1)

As far as Cisco Metacloud, I don't know much about it, so can't really comment, though it does seem t o be a valid option, assuming you like dealing with mega-sales-companies. 

Next up, the Ubuntu/Canonical entry. I have not used it, but it seems to be a valid choice. I am hesitant regarding the Juju configuration management tool, but I have not used it so that is unfair. The visualization of components is interesting.

The sale of the HPE Helion intellectual property to SUSE is weird. There would have to be some period of instability here. I can't see it being a viable option right now. Basically HPE has completely removed its involvment in OpenStack.

So, in my extremely biased opinion, what are we left with? Not much.

## What's left? Recommendations...

The Ubuntu/Juju distro seems like it would be Ok, but I have not used it.

RedHat OpenStack and Tripleo are good too. I have concerns about Tripleo on Day 2.

Rackspace Private Could is probably a great option given it is based on OpenStack-Ansible. Or OpenStack-Ansible by itself is great, if you want to use the open source version and support it yourself with help from the community.

Mirantis is still viable, but their k8s direction is concerning. Also, they are themselves questioning the value of a distro:

>...very few companies can manage a complex, distributed and fast moving piece of software such as OpenStack...Therefore, most customers end up utilizing extensive professional services from the distribution vendor. - [Mirantis Blog](https://www.mirantis.com/blog/how-does-the-world-consume-private-clouds/)

So even *with a distro* you are still using professional services. That said, their recent focus on Day 2 is powerful, though to be honest I have not used Fuel.

If you are a company that wants to use OpenStack and is hoping to make it simpler by using a distro, then I think you are in for a surprise. OpenStack is not cheap as it usually requires a team of fairly sophisticated infrastructure engineers to deploy and maintain, whether you are using a distro or not. Now, those engineers may be employees of your company, or they may be employees of a professional services firm. Either way, OpenStack requires people to manage it. Frankly distros may get in the way.

## Footnotes

1. Kubernetes (k8s) has really thrown a wrench into distros because everyone is scrambling to keep up with the hype. Kubernetes is great, but I think using k8s simply to manage the OpenStack control plane is overkill, and the additional complexity is not needed.
