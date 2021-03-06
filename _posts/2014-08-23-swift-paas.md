---
layout: post
title: Why OpenStack Swift is Great for Platform as a Service
categories:
---

# {{ page.title }}

I'm a big fan of object storage. What is object storage? To me:

> Object storage is a system that allows storing and retrieving files via a HTTP restful interface.

Object storage is not a file system and doesn't look anything like one.

In terms of this blog post, I think object storage system should also be highly available, scalable, redundant, and durable. (Maybe some of those terms mean the same thing.)

So we have these minimal requirements:

- Store and retrieve files via restful HTTP interface
- Highly available / Redundant
- Scalable
- Durable

Guess what open source solution meets those requirements? [OpenStack Swift](http://docs.openstack.org/developer/swift/).

## PaaS: Platform as a service

I really like the concept of platform as a service (PaaS). However, like almost every difficult to define term in information technology, it's become overloaded. PaaS could mean anything from [Heroku](http://heroku.com) to a git hook.

Recently Ander Shafer wrote a [blog post](http://blog.pivotal.io/cloud-foundry-pivotal/p-o-v/the-silent-aas) for Pivotal that suggests the "as a service" should be silent.

> [Is a] platform is just how one deploys code?

I would tend to agree with that sentiment, in that PaaS is really just a platform to which code is deployed and turned into a managed application (of some kind). Why make it more complicated than that? There are a lot of different PaaS implementations and I doubt there will ever be a canonical definition. What is PaaS for me could just be [Mesos](http://mesos.apache.org/) and [Marathon](https://github.com/mesosphere/marathon), and what is PaaS for you could be an exact Heroku clone, or Cloud Foundry, or OpenShift, or a bash script kicked off by a git hook.

Typically, however, there are certain things that people want out of a PaaS system:

- Don't want to run _the servers_
- Want it to be _scalable_ in some fashion
- Highly available
- Just push code (binary, git, zip, war...whatever)
- _Some_ state: database, file, nosql...thus backups!

I don't want to go much farther into what PaaS is...or isn't (partly because I don't know). I think it suffices enough to say that most users want the PaaS system they use to at least be scalable and highly available, and that is where OpenStack Swift can help.

## 1. Replace shared and distributed file systems with Swift

Many applications require the ability to upload or generally create files and then be able to access them later. However, if you run multiple application servers each application server probably needs to access those files. Now you need some kind of shared or distributed file system, eg. NFS or Gluster (among others) respectfully. While those systems can scale up or out fairly far, at some point they might not be enough for a large system, or to be a data store for a PaaS.

Enter Swift. If your application can be (re?)written to support object storage such as OpenStack Swift, and the PaaS being used supports it as a backend, then you don't need a complicated distributed or shared filesystem--you can just store and retrieve files from object storage.

## 2. Store Docker images in Swift

Many PaaS systems use, or will use, Docker as an important component for isolation. Docker is heavily dependendant on its registry server to manage images. Thus, any HA PaaS will also needs its Docker registry to be up as much as possible. Certainly Docker images could be cached on the servers that run Docker, but at some point a new image might need to be downloaded, and that has to come from a Docker registry.

Not surprisingly, the [Docker Registry server](https://github.com/docker/docker-registry) already supports using OpenStack Swift as a backend.

## 3. Store code in Swift

The code to be deployed has to come from somewhere. One example I have is with Apache Mesos and the Marathon framework. When you create an application in Marathon you can specify a uniform resource identifier (URI) where the application/code can be downloaded from. When a new instance of that application is created, for example when an application is scaled up, Marathon downloads the code from the URI(s).

![](/img/marathon-uri.png)
_(screenshot from creating an application in the marathon webgui)_

Thus, in order to scale an application, at least in this example, the URI, typically a web server URL, needs to be up and running so that the code can be downloaded.

If the application files are stored in an HA OpenStack Swift system, then that code should be availble to each application node to install. Certainly there are a lot of other ways this could be done, but I like the idea of using OpenStack Swift for this. Many of the examples given for deploying an application with Marathon show using Amazon's S3 object storage system.

I even see at least [one try](http://techs.enovance.com/6642/openstack-swift-as-backend-for-git-part-1) to back git with Swift.

## 4. Store backups in Swift

Depending on your definition, perhaps magic backups are part of your PaaS requirements. I say magic only slightly facetiously. I do believe that if a PaaS supplies data stores that they are backed up properly.

If you're building your own PaaS, having backups automatically replicated *and* replicated off-site would be a nice thing to have. Thus, if you have an OpenStack Swift cluster that spans multiple data centers, then all you have to do is stick whatever backup files you have into Swift and they will be replicated across zones. Swift even has the ability to create a [replication only network](http://docs.openstack.org/developer/swift/replication_network.html) that can run over your data center interconnect so you can do quality of service (QoS) if desired. Because of Swift's eventual consistency model it doesn't even break the [CAP theorem](http://en.wikipedia.org/wiki/CAP_theorem). Upload a file once and have it replicated across timezones? Yes please!

## Conclusion

I am just learning about PaaS, but already I can see how valuable object storage can be to platform as a service, and not just in terms of replacing file systems. OpenStack Swift is a mature and well thought out system that can have storage servers and proxy servers down for maintenance--unplanned or otherwise--and objects will still be available. Further, it's possible to upgrade Swift in-place without downtime. Also, as mentioned, it can scale across data centers.

If you are deploying a PaaS system then I heavily suggest taking some time to consider deploying OpenStack Swift along side it as a datastore.

I'm sure there are more good uses of object storage in a PaaS system, so if you think of anything let me know and I'll add it.

## Notes

- Like OpenStack in general, Swift can be a bit difficult to deploy. It depends on your skillset (ie. Linux). Start with [Swift All-in-one](http://docs.openstack.org/developer/swift/development_saio.html) to get an idea of the complexity. I don't think deploying Swift, or OpenStack completely, is that bad, but maybe some do. Also I have a project called [Swiftacular](https://github.com/ccollicutt/swiftacular) that can setup a multi-host test OpenStack Swift system using Vagrant and Ansible.

- PaaS is not a silver bullet, but I do think it helps to consider a platform as just a platform, with all its parts, good, bad, and missing. This is a great [blog post](http://blog.lusis.org/blog/2014/06/22/feedback-on-paas-realism/) (and the previous one, [_Paas for Realists_](http://blog.lusis.org/blog/2014/06/14/paas-for-realists/)) which discuss what may or may not be missing from popular PaaS systems, and what people building PaaS systems should consider as requirements.

- There are many people who believe that scaling applications is not as easy as some PaaS sytems make it out to be. Certainly it would be fairly straight forward to scale a stateless app up and down, but almost all apps have state of some kind, be it files or database or nosql entries, and at some point those data stores might fall over if there are too many application servers making requests. Also, same for the network.
