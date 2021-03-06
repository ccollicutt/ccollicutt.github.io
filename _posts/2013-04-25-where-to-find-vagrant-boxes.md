---
layout: post
title: Where to find vagrant boxes
categories:
---

# {{ page.title }}

Update 2: [Vagrant Cloud](https://vagrantcloud.com) has launched and I believe is now the best way to find Vagrant boxes.

Update: Added Phusion Passenger url.

This is just a quick post on a couple of places I know to find "vagrant":htp://vagrantup.com boxes. 

## Ubuntu cloud images

Today ubuntu 13.04, aka raring ringtail, was [released](https://wiki.ubuntu.com/RaringRingtail/ReleaseNotes?action=show&redirect=RaringRingtail%2FTechnicalOverview) But did you know that ubuntu actually provides vagrant specific boxes, ones that are built every day? They sure do!

- [http://cloud-images.ubuntu.com/vagrant/raring/current/](http://cloud-images.ubuntu.com/vagrant/raring/current/)

So it's quite simple to try out raring just by using vagrant and ubuntu's cloud images.

<pre>
<code>$ mkdir raring; cd raring
$ vagrant init
# Edit Vagrantfile and add the below
$ grep box Vagrantfile | grep -v "#"
  config.vm.box = "raring"
  config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/raring/current/raring-server-cloudimg-amd64-vagrant-disk1.box"

$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
SNIP!

#
# Now we can ssh into the box
#

$ vagrant ssh
Welcome to Ubuntu 13.04 (GNU/Linux 3.8.0-19-generic x86_64)
SNIP!
vagrant@vagrant-ubuntu-raring-64:~$ cat /etc/lsb-release 
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=13.04
DISTRIB_CODENAME=raring
DISTRIB_DESCRIPTION="Ubuntu 13.04"
</code>
</pre>

Nice. That was easy.

## Vagrantbox.es

Most vagrant users will know about this site, but I add it here for completeness.

- [http://www.vagrantbox.es/](http://www.vagrantbox.es/)

Obviously for testing/development using these images is just fine, but most shops will want to build their own production images. I think.

## Opscode Bento

This [github repo](https://github.com/opscode/bento) has tons of boxes, and also Packer templates to create your own.

## Phusion Passenger

Phusion Passenger also creates and distributes some [vagrant boxes](https://oss-binaries.phusionpassenger.com/vagrant/boxes/latest/).

## Make your own

[Veewee](https://github.com/jedi4ever/veewee) is a good way to automate image creation. I've used it quite a bit, but that was a few months ago. It can take a bit of work to get it up and running.

Puppet labs also [publishes](https://github.com/puppetlabs/puppet-vagrant-boxes) some information on creating vagrant boxes, as well as [several pre-built boxes](http://puppet-vagrant-boxes.puppetlabs.com/).


## Fedora

Unfortunately I can't seem to find official fedora or redhat vagrant boxes, which is too bad.

But, fedora is working on it!

- [http://fedoraproject.org/wiki/Features/FirstClassCloudImages](http://fedoraproject.org/wiki/Features/FirstClassCloudImages)

I'll try to update this page as I find more resources. Please feel free to comment with suggestions. :)
