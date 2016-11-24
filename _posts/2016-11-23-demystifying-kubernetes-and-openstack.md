---
layout: post
title: Demystifying Kubernetes and OpenStack
categories:
header_image: /img/ying-yang.jpg
---

*OpenStack on k8s and/or k8s on OpenStack*

# {{ page.title }}

There seems to be some confusion around using Kubernetes (k8s) and OpenStack together. As an OpenStack Operator, it bugs me a little bit that two very different models of using OpenStack and Kubernetes together are often conflated, occasionally for marketing purposes. In this post I want to identify the two major ways they work together and which is which.

##tl;dr

The two main ways of using OpenStack and k8s together are:

**1) Use k8s to manage the OpenStack Control Plane**

You can use k8s to manage the OpenStack control plane. Many people don't realize (especially VMWare administrators), that OpenStack has a fairly complex and resource intensive control plane. For example, if you are buying hardware for an OpenStack deployment, you need to buy hardware for the compute nodes AND the control plane. Typically OpenStack Operators use (n/2)+1 nodes for the control plane due to the use of distributed systems for messaging and state.

This would be a methodology used by OpenStack Operators.

**2) Install k8s into an OpenStack Project**

Once you have deployed an OpenStack cloud, you can then deploy k8s into a project within that cloud, ie. you, as an OpenStack end-user, provision virtual machines within OpenStack and deploy k8s onto those virtual machines.

This would be a methodology used by OpenStack end-users.

## k8s and the OpenStack Control Plane

OpenStack is a complex system. It requires many API servers and other services in order to work. APIs and other OpenStacks services are typically Python based servers. Supporting infrastructure such as MySQL/MariaDB and RabbitMQ, among others, are also required. These systems all have to run somewhere, and this somewhere makes up the OpenStack control plane.

For example, Nova, which provides compute in OpenStack, is made up of several servers, a few of which I list below:

~~~bash
/usr/bin/nova-api
/usr/bin/nova-cert
/usr/bin/nova-conductor
/usr/bin/nova-consoleauth
/usr/bin/nova-novncproxy
/usr/bin/nova-scheduler
~~~

Each of those services could be run on a monolithic "controller" server, ie. a single physical server, or they could each be split out into their own container (or virtual machine).

Using containers can make an OpenStack Operators life easier, especially when it comes to upgrading, or when you would like to run different versions of particular services which can often have dependency issues.

If you decide to use containers to deploy OpenStack services, then you will either have to manually schedule them, or you will need some kind of Container Orchestration Engine (COE) such as k8s, Docker Swarm, or something else. (I should note that in talking with other OpenStack Operators, it is not actually that common to use a COE to manage containers. Quite often containers are "manually" scheduled using some kind of configuration management tool such as Puppet or Ansible, both of which can provision containers.)

If you would like to use a COE, or have additional feature requirements, then using k8s may be of value to you. You could use k8s to manage each OpenStack service. Not only can k8s schedule containers, it also has some additional features that helps to keep those containers running. But it also brings additional complexity. k8s still requires care and feeding like any complex system. If an OpenStack Operator is running 60 containers on 3 physical servers, it may not be worth the additional complexity to run k8s. 

In this example, k8s would only run the services required to provide the OpenStack control plane.

Certain OpenStack distributions, such as Mirantis', are working to alter their deployment system to use k8s as the underlying COE for the OpenStack control plane.

## Running k8s in OpenStack

This is likely the more common use of k8s and OpenStack together. OpenStack would be used like any other Infrastructure as a Service system (IaaS), be it public or private cloud. You would somehow provision virtual machines within and OpenStack project/tenant and then deploy k8s to those virtual machines. k8s would then be running *in OpenStack*.

There are many ways to install k8s into OpenStack, from using kubeup.sh (which I believe supports OpenStack) to using the [OpenStack Magnum](https://wiki.openstack.org/wiki/Magnum) project, and many others. I don't want to list them all here, but suffice it to say that there are many ways to get k8s installed *into* an OpenStack cloud. Likely one would use some kind of automated k8s-onto-OpenStack deployment system.

## Conclusion

Hopefully I've demonstrated that there are two major ways that k8s and OpenStack can work together, and that they are quite different. By far the most common way of using k8s and OpenStack together will be to deploy a k8s cluster *into* an OpenStack tenant/project. That said, managing the OpenStack control plane with k8s could also be beneficial, but it does bring additional complexity.

Finally, to make things confusing, one could run the OpenStack control plane on k8s, and then also install another k8s cluster, or multiple k8s clusters, into the OpenStack cloud. Fun!
