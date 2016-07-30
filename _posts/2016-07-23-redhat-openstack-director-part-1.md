---
layout: post
title: RedHat OpenStack Director - Part 1 - Overview
categories:
header_image: /img/iceberg.jpg
header_permalink: https://flic.kr/p/9dVtm9
---

# {{ page.title }}

Due to the complexity, number of services, clustered systems, and other components that comprise an OpenStack cloud, some kind of tooling is required to manage its deployment (and hopefully life-cycle over time). There are several systems available to manage an OpenStack cloud, such as Mirantis' Fuel, [OpenStack Ansible](http://docs.openstack.org/developer/openstack-ansible/install-guide/), upstream [Tripleo](http://docs.openstack.org/developer/tripleo-docs/), to name only a few. 

RedHat also offers such a product: [Director](https://access.redhat.com/documentation/en/red-hat-openstack-platform/8/director-installation-and-usage/director-installation-and-usage). At least I think it's called Director, and I believe is part of the RedHat OpenStack Platform.

In this blog post I'll do a quick overview of RedHat Director 8 with regards to using it to install an OpenStack cloud. I should note that I am still working to understand some functionality of Director, especially around custom modification and post-deployment changes--though things are not looking well on those two points at this time. However, more investigation and experimentation is required. So there will be a second post on Director in the near future.

I should also note that I usually prefer to deploy OpenStack myself, most commonly using Cobbler and Ansible, though at some point I'd prefer to try to use Ironic over Cobbler. I find that I can deploy a complex OpenStack cloud in a couple thousand lines of Ansible. Whether or not to use an OpenStack deployment tool is a difficult discussion, though I can understand why deployment tools such as Director are chosen. It's similar to discussions on web frameworks. Some people like them, some people don't. Occasionally it's not possible to use a framework, but most of the time it's a good decision--as long as they are flexible.

## Features of OpenStack Deployment Tools

Most OpenStack deployment systems do at least the following in some manner:

* Baremetal operating system deployment
* Network design/deployment
* Configuration management
* Orchestration of services
* Vetted OpenStack configuration options
* OpenStack version upgrades
* Monolithic or container based deployment
* Plugin based architecture
* Engineering decisions made for you
* Reduction of complexity through abstraction layer
* Occasionally Day 2 operational tooling
* HA deployment using clustered/distributed systems

Each of these systems will make different, though similar decisions. For example RedHat Director uses Ironic to manage baremetal operating systems, whereas Mirantis Fuel uses Cobbler. While Director's configuration management component is mostly based on upstream OpenStack Puppet modules, OpenStack Ansible uses...you guessed it...Ansible. Regarding OpenStack control services, Director currently takes a monolithic approach and deploys all control services onto a single operating system instance (though can have multiple controllers), OpenStack Ansible deploys tens of LXC based containers with each major service running in its own container.

##What I like about Director

There are a few things that I like about RedHat's Director:

1. It's command line and template based and overall quite sparse
1. It uses Ironic for baremetal OS management
2. It works in terms of a repeatable installation
3. That it's based on midstream and upstream opensource projects
3. Undercloud/overcloud model

**Sparse**

Director is quite sparse. There is no fancy web interface, or at least not one that I'm aware of. I don't mind because I prefer to use the command line, and more specifically, if there is no web GUI, and the deployment is based on templates and the _openstack overcloud_ command, then the deployment definition can be stored in a source code repository. This makes it much easier to have repeatable installations.

**Ironic**

[OpenStack Ironic](http://docs.openstack.org/developer/ironic/index.html) is an interesting project used to manage baremetal (ie. physical servers) instead of virtual machines.

>Ironic is an OpenStack project which provisions bare metal (as opposed to virtual) machines by leveraging common technologies such as PXE boot and IPMI to cover a wide range of hardware, while supporting pluggable drivers to allow vendor-specific functionality to be added.

As mentioned, I typically use Cobbler for this part of an OpenStack deployment, but I'm enjoying using Ironic with Director. That said, Ironic is much more complicated than Cobbler, and things that are easy with Cobbler are not so easy with Ironic. However, they are much different systems and even though they effectively do the same thing, I'm not sure it's possible to compare them. Given my preference for an undercloud, it would make sense to incorporate Ironic there.

**Repeatable**

Repeatable deployments are important but perhaps not for the reason you would expect. All of these deployment tools need to be developed using modern continuous integration tools. And in fact all of the examples I provided, Director, Fuel, and OpenStack Ansible, do just that. This improves their quality because every change to their code is checked via CI. Though, I'm not sure how many of them test an HA deployment, as they typically only have the resources for some kind of all-in-one virtual deployment.

During my own investigation I have probably deployed the Overcloud 40 times. Each time make slight changes to the Director templates, gradually improving my deployment. It's important that a deployment be based in code, not in clicking buttons in a graphical interface.

For example, a new deployment requires these steps:

~~~bash
director$ heat stack-delete overcloud -y #delete the current cloud
director$ #edit templates if required
director$ ./bin/overcloud-deploy.sh #this contains the rather long deployment command referencing several templates
director$ #wait for it to complete
director$ heat stack-list
+--------------------------------------+------------+-----------------+---------------------+--------------+
| id                                   | stack_name | stack_status    | creation_time       | updated_time |
+--------------------------------------+------------+-----------------+---------------------+--------------+
| e24a92a9-dc9f-462b-af16-c8329714f238 | overcloud  | CREATE_COMPLETE | 2016-07-22T19:18:11 | None         |
+--------------------------------------+------------+-----------------+---------------------+--------------+
director$ #run validation tests, if everything looks good, check in the changes
~~~

Repeatable deployments are extremely important, and that is one area Director (and Tripleo) shines.

I should note that the Director node itself is quite "pet-like" and unless you do some work to automate its deployment, it is a system of record ([in the parlance of our times](https://www.youtube.com/watch?v=Wbv3gJ76NT8)...lol).

**Based on Opensource Code**

Director is mostly Tripleo. In fact I'm not completely sure where it differs from the upstream code. Specifically, Director consumes the output of the [RDO](https://www.rdoproject.org/) project. In turn, RDO consumes upstream Tripleo--thus it's "midstream", in between RedHat and Tripleo.

One issue is that the upstream code is considerably farther ahead than Directors. There are features in Tripleo trunk right now that I would love to have, and in fact seem completely necessary, but are not in Director because it's a few releases behind upstream. If I had my choice I would probably use the RDO output instead of Director, or perhaps even upstream.

**Undercloud/Overcloud**

[Tripleo](http://docs.openstack.org/developer/tripleo-docs/) stands for "OpenStack on OpenStack."

>TripleO is a project aimed at installing, upgrading and operating OpenStack clouds using OpenStack’s own cloud facilities as the foundation - building on Nova, Ironic, Neutron and Heat to automate cloud management at datacenter scale.

When you use Tripleo you have two clouds:

1. Undercloud, ie. Director
2. Overcloud

The Undercloud is an OpenStack instance where Ironic and Heat (as well as all the other required services) run so that you can deploy the Overcloud.

Most people don't grasp, at the beginning anyways, that an OpenStack cloud requires considerable infrastructure to run. You will probably need three or four potentially highly available or clustered virtual machines to run services such as Jenkins and other operational components, perhaps software defined network controllers (SDN), virtual machine management nodes for other traditional systems (SolidFire I'm looking at you), and more. 

You will need an Undercloud whether you give it that name or not. In fact in my lab I have at least three clouds: first, a cloud to run Director(s) in, then Director, then finally the Overcloud. It's fairly involved, and over and above the hardware you need for the HA plane of the overcloud. This is why container based deployments make a lot of sense, and unfortunately that is not something Tripleo supports at this time.

##Things I don't like

1. The Undercloud instance is a pet
1. Tripleo does not have much in the way of "life-cycle" management
1. Seemingly inflexible
1. Limited documentation
1. Hundreds of thousands of lines of Puppet
1. Orchestration?
1. Scaling controller services
1. Lack of containerization

**Undercloud: System of Record**

My impression of the Director system itself is that it is not "cloud native" for lack of a better phrase and would take some work to make it highly available in a fashion that would be typical of an application in an OpenStack cloud. Whether you call it a "pet" or a "system of record" or "not cloud native" it is an awkward system to operate.

**Lack of Life-cycle**

This is the biggest issue I have with Director. At this time I can find little having to do with operating a cloud over time, what some people call Day 2 (where Day 1 is the installation). Certainly there is some mention of upgrades and the like, but I have been working with Director for a few weeks and still really have no idea what it does in this area, despite looking through all the documentation. I have also heard that other organizations, HPe specifically, started out with Tripleo but dropped it because they realized the need to provide a life cycle for OpenStack deployments. Setting and forgetting an OpenStack installation is the easiest way to fail.

Currently I would be concerned at having to run a long-term cloud using Director, and I'm not even sure if you could. That said I have more to learn, and also Director is a young product. But so far it doesn't look good for Day 2 operations, let alone day 300.

**Lots of Puppet**

There is a ton of puppet.

~~~bash
[stack@director-01 ~]$ sudo find / -name "*.pp" | xargs wc -l
SNIP!
   1952 /home/stack/openstack-tripleo-heat-templates-0.8.14-14/puppet/manifests/overcloud_controller_pacemaker.pp
     57 /home/stack/openstack-tripleo-heat-templates-0.8.14-14/puppet/manifests/overcloud_object.pp
     61 /home/stack/openstack-tripleo-heat-templates-0.8.14-14/puppet/manifests/overcloud_volume.pp
     96 /home/stack/openstack-tripleo-heat-templates-0.8.14-14/puppet/manifests/ringbuilder.pp
      0 /opt/stack/selinux-policy/ipxe.pp
 115712 total
~~~

Eep. Certainly not all that Puppet is getting executed with every deploy, but even if it's 50%, that is still a lot of Puppet to understand. I can deploy a production OpenStack cloud with 3-5K lines of Ansible. If you don't understand it, then it's an abstraction. 

**Seemingly Inflexible**

I have tried to update an overcloud twice post-deploy. Both have failed and in fact broke the overcloud. This is pretty concerning.

Further, on one deploy I attemped to disable Swift. The default deployment of swift configures a storage device on each of the controllers.

~~~bash
[heat-admin@overcloud-controller-2 swift]$ swift-ring-builder object.builder 
object.builder, build version 3
1024 partitions, 3.000000 replicas, 1 regions, 1 zones, 3 devices, 0.00 balance, 0.00 dispersion
The minimum number of hours before a partition can be reassigned is 1
The overload factor is 0.00% (0.000000)
Devices:    id  region  zone      ip address  port  replication ip  replication port      name weight partitions balance meta
             0       1     1    172.17.19.12  6000    172.17.19.12              6000        d1 100.00       1024    0.00 
             1       1     1    172.17.19.13  6000    172.17.19.13              6000        d1 100.00       1024    0.00 
             2       1     1    172.17.19.11  6000    172.17.19.11              6000        d1 100.00       1024    0.00 
~~~

In hopes of reducing the complexity of the deployment somewhat, and not relying on a somewhat unusual Swift deployment, I attempted to turn it off by setting *ControllerEnableSwiftStorage* to false.

~~~bash
director$ grep -A 2 "  ControllerEnableSwift" overcloud.yaml 
  ControllerEnableSwiftStorage:
    default: true 
    description: Whether to enable Swift Storage on the Controller
~~~

Disabling that still had Swift deployed, it just didn't work because the storage component was not configured. lol. It seems that in Tripleo trunk there is the ability to [select services](https://github.com/openstack/tripleo-heat-templates/blob/master/overcloud.yaml#L367) which I could really use...like right now.

Further, I can't find a way to disable Ceilometer. By default Ceilometer is deployed with a MongoDB backend, all on the controllers. In a busy OpenStack cloud that will surely fail.

**Limited Documentation**

The RedHat provided documentation is limited. Further, it seems that the suggestions made in the documentation are not really what is used in production. Also, by default the Director templates implement things like bonding using Open vSwitch, but then in the documentation it suggests not using OVS bonding. The upstream documentation is much better, and you could then ask questions on the RDO or Tripleo mailing lists or IRC channels...not so much for RedHat.

In fact, in order to really understand how Tripleo works, I think you'd have to run upstream in the lab and learn from that. Then perhaps you could drop down to Director.

**Orchestration**

Orchestration is an overloaded term. In this context I mean the ability to perform complex actions on a cloud once it's already deployed. For example upgrades. Or a simpler example, restart a clustered service. I am not clear on how to performan these kinds of actions using Director. Fuel performs these kind of actions using MCollective, and OpenStack Ansible can just use Ansible. But for Director? I'm not exactly sure. My guess is that it is done via the various custom os-* commands the project has created. More investigation is required.

**Scaling Controller Services**

How do I add a controller? Or provide more nova-conductor services? According to the [RedHat docs](https://access.redhat.com/documentation/en/red-hat-openstack-platform/8/director-installation-and-usage/chapter-9-scaling-the-overcloud) you cannot scale up, or down, the controller nodes.

**Lack of Containerization**

Monolithic OpenStack controllers have largely been left behind. Most people are using containers to split out OpenStack services. I would not, if I had the choice, deploy monolithic controllers. I've made that mistake before. At the very least LXC should be used. I believe Director/Tripleo is starting to support Docker, but RedHat lists it as a tech preview.

It seems counter-intuitive, but using something like [Kubernetes](http://kubernetes.io/) makes a lot of sense for the OpenStack control plane. Or, as mentioned, at least LXC, which I am using in my own undercloud. LXC 2.0 recently came out and while I have issues with it, overall I'm happy.

##Conclusion

I've deployed large, production, high SLA OpenStack clouds before, and run them over time. Deploying is easy, operating is bloody difficult. I know what is required...what works and what doesn't. Today my impression of Director is that it is fairly behind in terms of what successful, modern OpenStack operators are doing to manage their clouds. I have a lot more to learn about Director and Tripleo, but three or four weeks in I feel I have a good idea of what they offer.

Hopefully in the next post I'll know more about Day 2 capabilities, and perhaps I'll be more comfortable with it. Certainly there is no denying that behind the scenes Director, based on Tripleo, is quite complicated and has many components to learn and understand. Perhaps those components allow for operational proficiency. Or perhaps not.
