---
layout: post
title: Vent - Kubernetes the Hard Way with Ansible and Packet.net
categories:
header_image: /img/vent.jpg
header_permalink: https://unsplash.com/photos/ZOUSOJFzQHg
---

# {{ page.title }}

In preparation to write the Certificate Kubernetes Administrator exam I’m going through the ever popular Kubernetes the Hard way (kthw), as developed by Mr. Kelsey Hightower. I actually did [the same thing a couple of years ago]( https://github.com/ccollicutt/kubernetes-the-hard-way-with-aws-and-ansible), in which I translated kthw from a step by step command line depoyment into some Ansible that deployed kthw to Amazon Web Services.

In this blog post I discuss a repo I created that does something similar--it still converts the shell commands into Ansible, but instead of AWS it deploys Kubernetes to bare metal nodes on Packet.net.

## Packet.net

I’m a big fan of packet.net. I like four major things about packet.net:

1. Bare Metal resources
1. Layer 3 networking
1. Simplicity
1. Spot pricing

### Baremetal

Obviously the main point of packet.net is that instead of getting virtual machines, you get bare metal nodes. I really like getting bare metal nodes. You don’t have to worry about sharing resources with other tenants. You get the full speed of bare metal. They can be pricey over time, but if I was an organization seeking speed, for example in CI/CD, it would be well worth it to utilize packet to push things as fast as they can go.

Specifically for Vent I focussed on the AMD EPYC nodes which are under plan c2.medium.x86. With 64GB of RAM, 20GB of networking, 24 cores, and about 1TB of SSD, it’s perfect for Kubernetes.

Other cloud providers also have bare metal nodes, but I haven’t used them. I think AWS provides bare metal, and they have the Nitro hypervisor, which is effectively bare metal. But that is a discussion for another day. :)

### Networking

I like how Packet.net has done their networking--it’s all layer 3 based. As a public cloud, this is the right way to go. People often think networking means a shared layer 2 with a /24 netmask, like what you get with most clouds virtual private networking.

Packet.net’s layer 3 model means that each node gets three IPs: a public IPv4, an Ipv6, and an internal, private RFC 1918 address (all on the same bond0 interface). The private address is in the 10.0.0.0/8 range, but it’s usually a /31, which means it just has a couple usable addresses--the hosts IP and the gateway. A route is setup to send all 10.0.0.0/8 addresses through that gateway, and packet must be doing filtering to allow “private networking” in the sense that other nodes in the same project are put into a filtering group (ie. network ACL), so that they can only talk to each other on the private network. So if you have three nodes, they each have their own /31 private IP space, but are part of an ACL group (presumably).

This is a very scalable model for packet.net, and with an underlying layer 3 private network as a tenant you can set up an overlay network if you like, vxlan, what have you, and in the example of Kubernetes you could do something simple like use flannel, which is an overlay.

When deploying things like Kubernetes and OpenStack I pay a lot of attention to networking models--it takes some thought about how to fit k8s into an environment. Overlays almost always work because they just ride on top of the underlying physical network and just need L3 connectivity between the virtual switches or whatever is taking care of the overlay. However, overlays are not alway the. Good architecture decisions must be made.

One issue I have with this model is that I don’t necessarily want nodes listening on public IPv{4,6} address if they don’t need to be, but would still like them to have egress access to the Internet. It would be worthwhile for packet.net to have a nat gateway setup like AWS can do. They would have to perform some network magic in the background, but I think it’d be doable. More likely they will create a VPC-like model with a hidden overlay, perhaps using EVPN. Who knows. :)

(Aside: I did do a lot of work and setup dnsmasq, squid, and apt-cacher-ng in packet.net. I removed the public IPv{4,6} addresses from the interface and setup nodes to go through a cache. This worked for 99% of what I was doing--but in some cases applications *cough* docker images *cough* are really annoying and require access to the Internet without a proxy. Again, another story for another day.)

I had to do some tricks with macvlan to get flannel working with packet.net’s model, but it’s flannel’s issue. See the [vent repo](https://github.com/ccollicutt/vent) for more information on that. At least I finally learned about macvlan to work around the issue.

*NOTE: Packet also support setting up your own standard VLANs in some capacity, but I haven't explored that yet.*

### Spot pricing

Spot pricing is essentially a way to bid on computing time. You put in your bid, and if it’s high enough you get the compute resources you asked for. Later on, if someone puts in a higher bid your resources get diverted to them. What this really means is that your bare metal server will be deleted.

I’m a big fan of spot pricing, as I think that economics is really important in computing, or it should be at least. With spot markets you can really test out some economic theories. Or, at least, get less expensive short term resources, such as CI/CD or for testing out deployments, which I do a lot of. While working on Vent I usually was able to get the AMD EPYC nodes for about $.30 in spot pricing versus $1 they normally are per hour. That’s quite a discount.

## Simplicity

There is a core set of features that a public cloud requires, IMHO. Public clouds like AWS have hundreds of custom services. It can be very confusing to understand what is available and how they work, let alone their billing mode. I'm not saying one way is better than another, but it's often refreshing to have a simpler view of public cloud resources. Digital Ocean is similar in their desire to keep things clean. Time will tell if this is a good product design, but I appreciate it. A good networking model, access to compute resources, and persistent volumes, elastic IPs...the basic requirements, it's all I need. I'm not even a big fan of "security groups" which packet does not have. Load balancers are probably a good idea though.

## kthw

Mr. Hightower has changed kthw considerably over the years. As I write this I’m sure he’s creating a new version that will come out soon. WHen I last worked with Kubernetes and kthw it was version 1.3.0. Now we are on 1.10.7 for Vent, 1.10.2 for kthw, and 1.11 for Kubernetes proper.

kthw is not at this time using docker in a way that you think it would. Docker is not actually installed, instead it’s using [containerd](https://containerd.io/). It also uses [gvisor](https://github.com/google/gvisor), another Google project.

> Gvisor [is] a new kind of sandbox that helps provide secure isolation for containers, while being more lightweight than a virtual machine (VM). gVisor integrates with Docker and Kubernetes, making it simple and easy to run sandboxed containers in production environments.

I really appreciate the work that Mr. Hightower has put into kthw as I have used it quite a bit to learn about Kubernetes.

## Vent

Finally, lets chat about Vent a bit. First, I have an [overview document](https://github.com/ccollicutt/vent/blob/master/OVERVIEW.md) in the vent repo that discusses the choices I made and issues I ran into if you’d like something more in depth.

With Vent I took kthw and turned it into a set of Ansible playbooks. Specifically I didn’t use roles, each step in kthw is a separate playbook. For convenience, they are all brought together in a monolithic file, all.yml. I chose this model to make it look like kthw, with a step by step process. In the future I may change this to be role based.

Provisioning packet.net instances is not done with Ansible, and instead I’m using the packet.net golang based CLI. it has support for spot instances. You can provision Packet.net instances with Ansible or Terraform, and probably other systems, but Ansible and Terraform don’t seem to support spot instance pricing (at least at this time). I should probably contribute that to the Ansible module, but haven't had time.

One thing I don’t like about how I’m deploying k8s is that I'm relying pretty heavily on the flannel k8s deployment yaml to manage the networking. I’d like to have Ansible set that up instead of using the magic deployment from CoreOS. Also, I’d like to not use flannel at all and move to an Open vSwitch based networking model, but that will take some time to research. For now it’s magic flannel. :)

Like kthw, I’m not putting the k8s control plane into containers or self-hosted in k8s itself. Self-hosting is what kubeadm and some other deployment tools do. Something to look into in the future.

Going forward I hope to write a “vent” CLI that does a lot of the setup, including provisioning, setting up a few basic variables, etc.

## Conclusion

Thanks to packet.net for a great service. Thanks to Mr. Hightower for kthw. And wish me luck on the CKA exam!

