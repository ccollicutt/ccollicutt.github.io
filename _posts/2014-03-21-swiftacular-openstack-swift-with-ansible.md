---
layout: post
title: Swiftacular - deploy OpenStack Swift with Ansible
categories: 
---

# {{ page.title }}

A few months ago I put a good amount of working into learning more about OpenStack Swift and how to deploy it. I used Ansible as my configuration management system, and called the whole project [Swiftacular](https://github.com/ccollicutt/swiftacular).

Recently I realized that I forgot to blog about it, so I decided it was time to fix that. :)

## OpenStack Swift

OpenStack Swift is an object storage service made up of many components. From their [documentation](http://docs.openstack.org/developer/swift/:)

> Swift is a highly available, distributed, eventually consistent object/blob store. Organizations can use Swift to store lots of data efficiently, safely, and cheaply.

In several ways it is analogous to Amazon's S3 service, which has billions and billions of files/blobs/objects (whatever you want to call them) stored in it. 

Most people who use the Internet or smartphones with mobile apps use object storage, they just don't know it because it's used in the background by the applications, such as ones that need to store many, many files, like pictures. 

## New features in Swift: Replication network and regions

Swift has a couple new features which are used in Swiftacular. Though I should note that by default Swiftacular only sets up one region.

[Replication network](http://docs.openstack.org/developer/swift/replication_network.html:)

> Swift’s replication process is essential for consistency and availability of data. By default, replication activity will use the same network interface as other cluster operations. However, if a replication interface is set in the ring for a node, that node will send replication traffic on its designated separate replication network interface.

The idea is that you could do some quality of service on the replication network, or split it off entirely from the standard Swift network, as likely the replication network would go across the link between regions, which are in most cases going to be in different data centers. The link would likely be lower bandwidth and higher latency. 

[Regions](https://www.swiftstack.com/docs/admin/cluster_management/regions.html:)

> Whereas Zones are designed to distribute replicas among nodes and drives such that there is no single point of hardware / networking failure, Regions are conceptually designed to distribute those replicas among different geographical areas.

Regions are great because you can then deploy one large OpenStack Swift cluster across multiple geographically separated data centers. The organization I currently work for is deploying Swift in Calgary, Alberta and Kelowna, B.C. Actually these are in different timezones too!

## Deploying OpenStack Swift with Ansible

I like to use Ansible because it's straightforward to understand and executes over ssh instead of some custom RPC type system requiring a client running on the remote system and certificates. ssh is awesome. Ansible is awesome. Swift is awesome. Radical!

First off, you can find the repository containing all the Ansible playbooks and roles needed to deploy Swift [here](https://github.com/ccollicutt/swiftacular). That repository also contains a README file that will likely be more up to date than this blog post.

Requirements:

- Vagrant
- Virtualbox
- Ansible
- Internet connection

Once you have those requirements, this is how to quickly deploy OpenStack Swift:

<pre>
<code>$ git clone git@github.com:ccollicutt/swiftacular.git
$ cd swiftacular
$ mkdir library
# Checkout some modules to help with managing openstack 
$ git clone https://github.com/openstack-ansible/openstack-ansible-modules \
library/openstack
$ vagrant up 
$ cp group_vars/all.example group_vars/all
$ vi group_vars/all # ie. edit the CHANGEMEs in the file, if desired
# Source aliases, etc
$ . ansiblerc
# Test connectivity to virtual machines
$ ans -m ping all
# Run the playbook to deploy Swift!
$ pb site.yml
</code>
</pre>


After those commands have completed, you should end up with several virtual machines running.

<pre>
<code>$ vagrant status
SNIP!
swift-package-cache-01    running (virtualbox)
swift-keystone-01         running (virtualbox)
swift-lbssl-01            running (virtualbox)
swift-proxy-01            running (virtualbox)
swift-storage-01          running (virtualbox)
swift-storage-02          running (virtualbox)
swift-storage-03          running (virtualbox)
SNIP!
</code>
</pre>

Those virtual machines comprise an OpenStack Swift cluster. While there is only one region setup by default, the storage servers are setup with a replication network. Both regions and the replication network are new features of OpenStack Swift. In /etc/swift of each of the storage servers there is both a replication configuration file and a standard server configuration.

<pre>
<code>root@swift-storage-01:~# ls /etc/swift/*-server
/etc/swift/account-server:
account-replication.conf  account-server.conf

/etc/swift/container-server:
container-replication.conf  container-server.conf

/etc/swift/object-server:
object-replication.conf  object-server.conf
</code>
</pre>

The replication server configuration file looks like this:

<pre>
<code>root@swift-storage-01:~# cat /etc/swift/object-server/object-replication.conf 
[DEFAULT]
devices = /srv/node
bind_ip = 10.0.30.200
workers = 2

[pipeline:main]
pipeline = object-server

[app:object-server]
use = egg:swift#object
replication_server = True

[object-replicator]

[object-updater]

[object-auditor]
</code>
</pre>

As can be seen above the replication server is listening on 10.0.30.200 where 10.0.30.0/24 is the replication network configured. Also the "replication_server" option is set to True.

This is what the regular server config file looks like:

<pre>
<code>root@swift-storage-01:~# cat /etc/swift/object-server/object-server.conf      
[DEFAULT]
devices = /srv/node
bind_ip = 10.0.20.200
workers = 2

[pipeline:main]
pipeline = object-server

[app:object-server]
use = egg:swift#object

[object-replicator]

[object-updater]

[object-auditor]

[object-expirer]
</code>
</pre>

It's listening on 10.0.20.200 and is not setup as a replication only server.

All three of the object, container, and account servers are setup in the same fashion.

## Using Ansible delegation to setup the rings

Part of configuring OpenStack Swift involves adding devices to the ring. Thankfully Ansible supports delegating commands. So while some configuration management systems need to have centralized metadata (think PuppetDB) in order to configure all of the devices, with Ansible we can simply use a delegation command, which means that when a storage server is being configured, we can actually delegate a configuration command to run on the proxy server.

That sounds complicated but it's fairly simple. Maybe a better example is if using Ansible to configure a webserver, and then using a delegation command to add the webserver to a loadbalancer.

As an example, when a storage server is configured we can tell the proxy to add it's devices to the ring. Below is an Ansible task that is part of Swiftacular that delegates configuration of a ring device when a storage node is being configured. The command actually runs on the Swift proxy.

(Sorry the example swift-ring-builder command below will probably stretch across the screen. It's long and kinda complicated and is really meant to be invoked programatically.)

 <pre>
 <code>{% raw  %}

 - name: build account ring
   command: swift-ring-builder account.builder \
   add r{{ region }}z{{ zone }}-{{ ansible_eth3.ipv4.address }}:6002R{{ ansible_eth4.ipv4.address }}:6002/{{ disk_prefix }}{{ item }} 100
           chdir=/etc/swift 
   delegate_to: "{{ swift_proxy_server }}"
   with_sequence: count={{ disks }}
   when: "losetup.rc > 0"
 
 {% endraw  %}
 </code>
 </pre>

Delegation is very handy, especially with OpenStack Swift where the proxy needs to know what devices each storage server has.

## Documentation of SSL/TLS in OpenStack

I don't know why, but much of the OpenStack documentation, especially around the authentication system Keystone, avoids discussing how to deploy SSL/TLS enabled services. I'm not talking about Swift here specifically, rather the rest of OpenStack--authentication, endpoints, and other services that should be TLS enabled. 

On one hand most organizations deploying OpenStack know that there is going to be a layer of TLS termination in front of most services, but on the other it's not obvious from that general documentation that this layer should exist. The [OpenStack Security Guide](http://docs.openstack.org/sec/) goes into more detail about the TLS layer, but don't go very deep: 

> OpenStack endpoints are HTTP services providing APIs to both end-users on public networks and to other OpenStack services within the same deployment operating over the management network. It is highly recommended these requests, both those internal and external, operate over SSL.

I guess what I'm saying is that it would be good to know how larger OpenStack providers are securing OpenStack services with TLS. :)

Swiftacular sets up Keystone and a loadbalancer with TLS enabled to front swift-proxy. Certainly it's not setup as you would do it in production (likely with with a dedicated set of TLS termination servers) but it's a good example. I don't think the OpenStack documentation should show deploying systems such as Keystone without SSL.

## Conclusion

Using the Swiftacular Ansible playbooks and roles, and using the provided Vagrantfile, it's fairly easy to get a small OpenStack cluster going on a good laptop or workstation. It also includes regions and a replication network setup. This could be a good basis for starting out with OpenStack Swift. 

As usual--if you have any questions, concerns, comments or criticism, do let me know. It's quite likely I've made a mistake somewhere here, small or large. :)

## Updates

- Tried to clarify SSL/TLS section

