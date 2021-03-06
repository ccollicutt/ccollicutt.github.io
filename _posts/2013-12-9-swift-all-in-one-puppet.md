---
layout: post
title: Deploy Swift All in one with Puppet
categories: 
---

# {{ page.title }}

My current employer is deploying [OpenStack Swift object storage](http://docs.openstack.org/developer/swift/). I'm a big fan of object storage, and the deployment we are working on is one that will have two regions in separate timezones, so it's very interesting in terms of having a geo-replicated Swift cluster. 

We would like--or at least some of us would like--to deploy it with the [Puppet configuration management system](http://puppetlabs.com). While I am quite familiar with [Ansible](http://ansibleworks.com) I have hardly used Puppet, so in order to deploy our Swift cluster I need to learn how to use it.

## Learning Puppet

Puppet is a mature configuration management system, which means it has a lot of what I would call "best practices"...things like [style guides](http://docs.puppetlabs.com/guides/style_guide.html), [blog posts on writing good modules](http://puppetlabs.com/blog/writing-great-modules-an-introduction), etc, etc.

The way I decided to get into Puppet was to configure puppet manifests to deploy [OpenStack Swift All-in-one](http://docs.openstack.org/developer/swift/development_saio.html). Swift All-in-one is a way to run the an entire Swift system off of one virtual machine, and to deploy the Swift code from Git.

### Deploy Swift-all-in-one with the puppet-saio module

I called the module SAIO and it can be found on the [Puppet Forge](http://forge.puppetlabs.com/serverascode/saio). 

Usually I would write a bit about how to use it, but that should all be in the [README](https://github.com/ccollicutt/puppet-saio/blob/master/modules/saio/README.md) which should be much more up to date than this blog post.

## Vagrant

One part that is not included in the Puppet module on the forge site is that there is also a Vagrant configuration file and some Puppet bootstrapping that will allow a simple <code>vagrant up</code> and a Swift All-in-one virtual machine will be built automatically.

<pre>
<code>$ git checkout https://github.com/ccollicutt/puppet-saio
$ cd puppet-saio
$ vagrant up
SNIP!
Notice: Finished catalog run in 195.17 seconds
$ vagrant ssh
# run remakerings
# then run startmain
</code>
</pre>

At this point you can start testing out Swift.

## Incremental puppetism...

So far I have moved the module from a single <code>init.pp</code> file to breaking it out into a couple of functions, as well as adding <code>params.pp</code> file (which seems to be a best practice). Also I have run <code>puppet-lint</code> on it and fixed most of the warnings, though I have some work to do in my text editor to make sure that it lets me know about code style issues so I can fix them as I'm editing. 

Surprisingly, even with a small module (likely less than 400 lines once it fully matures) there is a lot of work in terms of determining order of operations and in breaking it out into ever smaller chunks, both of which are very puppety things to do, and in fact pretty much define the puppet paradigm. 

I'm not sure Puppet would be considered officially object oriented but it is certainly object-like, and I have spent more time thinking about order, "chunking", and best practices than I did writing the tasks. That said, the theory is this will lead to better modules.
