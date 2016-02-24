---
layout: post
title: The Layers of OpenStack
categories:
header_image: /img/layers.jpg
header_permalink: https://www.flickr.com/photos/druclimb/16993017815/in/photolist-rTBEaM-6Phcvq-6HcbL7-rZRug6-6Pd3MX-dRFpDz-rdipAS-6p68Fp-6Pd1Y2-6PhbRy-chf6Nj-6Pd2Yg-6Pd3pn-m3HCvV-i4jj1o-gEQ1C4-8bYXJD-33eXpZ-8x8BrP-6cwgNE-6zC1Gj-NHoxN-deHLxy-dRmzxE-47BCi2-8KN5N5-PZCPX-8Gt1of-n3bcaP-n3bjC4-kJa29V-k2DQvE-jEU4ag-jETYnR-n3baFX-8co2Nn-e8ycAC-78ZCsg-7smnJq-ibKewk-5DjhcZ-pFHhN3-kzR3JU-3Y2cN-3Y2c8-4TUz4Y-4YwJ9X-fgbdXn-dbY7Yp-eFTBEY
---

# {{ page.title }}

I think anyone who has looked into OpenStack has read that it is complicated and difficult to install let alone operate over time. However, that's not the point of this post--rather the point of this short post is just to discuss what pieces are involved in OpenStack and how I tend to approach deploying it, an approach that I feel simplifies the system somewhat.

Computering is complicated. Sometimes it helps to simplify it from a high level, and I use "layers" to do that. What's more this model can help people who don't have to understand every single component of the entire system to get in idea of what it looks like. An OpenStack production system can be very large, especially when considering all the ancilliary infrastructure required.

I do want to note that once it's deployed the order of the layers is not all that relevant, but I think putting the layers in order prior to deployment can help people understand the components of OpenStack.

## Prior to installation

Before installing all kinds of decisions have to be made, things like what server vendor will be used, same for network, datacenter requirements, what OpenStack will look like (eg. are you using Ceph for block and object, booting from volumes, nova network or neutron, neutron with a plugin, etc, etc) but I'm not going to cover any of that, rather I will just simply list what I usually install and in what order.

Before starting these layers I have applied the base OS to every server, and there is basic network connectivity on the managment network that Ansible operates over.

## The Layers

I use Ansible to deploy OpenStack. Perhaps that's why I've settled on the layer strategy. Ansible pretty much works in a serial fashion, one task after another, one playbook after another. Ansible is applied "top to bottom" versus something like Puppet where modules are compiled and then applied. So with Puppet order isn't defined unless it's specifically defined.

The basic layers are below. Some of the layers are more complicated than others, ie. have more tasks to complete.

1. ssh-keys - Setup operator ssh keys
1. node-network - Setup more comlicated /etc/network/interfaces
1. sshd - Setup sshd securely, eg. only listen on specific interfaces, no password logins, etc
1. baremetal - Some basic common requirements and security settings
1. rsyslog - Setup rsyslog server and clients
1. etc-hosts - Configure /etc/hosts
1. lxc-hosts - Configure servers that will be lxc hosts
1. lxc-containers - Start various lxc-containers
1. mariadb-galera - MySQL Galera cluster
1. haproxy - haproxy does all the API load balancing
1. rabbitmq-cluster - Rabbit cluster
1. memcached - Setup memcached servers 
1. openstack-repo - If installing OpenStack from packages, setup the repo (eg. Juno, Kilo, etc) 
1. keystone - Keystone authentication service
1. swift-common - Next up, Swift because it's backing Glance
1. swift-object - Setup Swift storage nodes
1. swift-ring - Create Swift ring
1. swift-fetch-ring-files - Obtain the created Swift ring files
1. swift-distribute-ring-files - Distribute the ring files across all Swift nodes
1. swift-proxy - Configure Swift-proxy servers
1. glance - Setup Glance
1. nova-common - Next up, Nova for compute
1. nova-controller
1. nova-compute
1. neutron-common - Networking!
1. neutron-controller
1. neutron-network
1. neutron-compute
1. cinder-common - Block storage!
1. cinder-controller
1. cinder-storage
1. heat - And Heat (heat is nice and easy to install)
1. horizon - Finally the GUI 

Obviously there are more pieces (metrics, monitoring, cron jobs, backups, alerting, more logging, DNS, etc) but I won't cover those in this post. And of course there are many other OpenStack projects that could be used (and more every day).

## OpenStack deployed!

Now with all those layers laid down, you have an OpenStack system. At this point the layers don't really exist in a defined order and you are going to be operating this system over a long period of time. And that, my Internet friend, is a whole other story. :)

