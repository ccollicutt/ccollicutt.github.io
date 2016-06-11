---
layout: post
title: OpenStack Provider Networks
categories:
header_image: /img/pyramid.jpg
header_permalink: https://twitter.com/archillect/status/741590822320254976
---


# {{ page.title }}

There are many ways to deploy networking in OpenStack. I've deployed it old-school with nova-network, new-school with Neutron and Midokura's Midonet, and just recently I put up a lab deployment of Neutron + [provider networks](http://docs.openstack.org/mitaka/networking-guide/scenario-provider-ovs.html).

To me, provider networks are kind of like nova-network in terms of their simplicity of deployment, where simplcity means your network team probably doesn't have to do anything new, and can rely on their (potentially) tried and true network designs. I mean, let's face it--many network architects are going to dislike SDN and/or overlays, etc. Using provider networks will at least allow OpenStack to be deployed in somewhat hostile network environments.

From the [OpenStack Networking Guide](http://docs.openstack.org/mitaka/networking-guide/scenario-provider-ovs.html):

>Provider networks generally offer simplicity, performance, and reliability at the cost of flexibility. Unlike other scenarios, only administrators can manage provider networks because they require configuration of physical network infrastructure...In many cases, operators who are already familiar with network architectures that rely on the physical network infrastructure can easily deploy OpenStack Networking on it.

The goal of this post is to go over the deployment and include some snippets of configuration to give a cursory example of how this deployment is working and what it looks like while running. Please note this is a lab deployment and is not necessarily meant for production use. Oh, and I'm not a "network architect" by any stretch, but I have deployed some fairly complicated networks in relation to OpenStack.

## The Stack

1. Ubuntu 14.04
1. Edgecore 5712
1. Cumulus Linux 2.5.7
1. Open vSwitch 2.5.0 from Ubuntu's Cloud Archive
1. OpenStack Mitaka from Ubuntu's Cloud Archive
1. A single controller running LXC 2.0 and a bunch of containers
1. A couple of baremetal compute nodes
1. 100% managed by Ansible

## Cumulus Linux

In my case I control the physical network and it consists of an Edgecore 5712 with Cumulus Linux loaded on it.

~~~ bash
cumulus@oc-sw-02$ cat /etc/lsb-release 
DISTRIB_ID="Cumulus Linux"
DISTRIB_RELEASE=2.5.7
DISTRIB_DESCRIPTION=2.5.7-753304d-201603071654-build
~~~

Recently Cumulus released 3.0 so I should load that on soon, but for now I'm back on 2.5.7.

Cumulus has an interesting feature called [VLAN aware bridge mode](https://docs.cumulusnetworks.com/display/DOCS/VLAN-aware+Bridge+Mode+for+Large-scale+Layer+2+Environments) and that is what I'm using.

Here's a snippet of my interfaces file, which is managed by Ansible. VLANs 12 and 13 are meant to be the provider networks.

~~~
# VLANS
auto bridge0
iface bridge0
    bridge-ports controller-01-bond0 compute-01-bond0 compute-02-bond0
    bridge-vlan-aware yes
    bridge-vids 10 12 13 11
    bridge-pvid 1
    bridge-stp on

auto bridge0.10
iface bridge0.10
    address 172.17.3.1/24
auto bridge0.12
iface bridge0.12
    address 172.16.5.33/27
auto bridge0.13
iface bridge0.13
    address 172.16.5.65/27
auto bridge0.11
iface bridge0.11
    address 172.16.5.1/28
~~~

I quite like working with Cumulus linux. The Edgecore 5712 + Cumulus is a compelling offer.

# Neutron

I deployed Neutron exactly like the Open vSwitch and provider networks is shown in the networking guide. One difference from the guide is that neutron-server by default doesn't use the ml2_conf.ini file, only the openvswitch_agent.ini file.

My deployment has the physical network providing layer 2 and layer 3, but Neutron is handling DHCP. So on the neutron-api node it sets up some namespaces where the DHCP server listens.

~~~
root@uc-neutron-api-01:/etc/neutron/plugins/ml2# cat openvswitch_agent.ini 
[ml2]
type_drivers = flat,vlan
# empty because we don't support project/private networks
tenant_network_types = 
mechanism_drivers = openvswitch
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[ml2_type_vlan]
network_vlan_ranges = provider 

[securitygroup]
firewall_driver = iptables_hybrid 
enable_security_group = True

[ovs]
bridge_mappings = provider:br-provider
~~~

There are a couple of networks setup in neutron.

~~~
ubuntu@uc-osclient-01:~$ neutron net-list
+--------------------------------------+------------+-----------------------------------------------------+
| id                                   | name       | subnets                                             |
+--------------------------------------+------------+-----------------------------------------------------+
| 5e73a2f5-a52b-4833-9978-531fc98cd783 | vlan_13 | 725c3dbe-65f2-4dc5-b9ec-596309bc2229 172.16.5.64/27 |
| eb175122-2500-4234-becb-030da538893a | vlan_12 | 7b799b92-ba55-454c-8db7-558476559b4e 172.16.5.32/27 |
+--------------------------------------+------------+-----------------------------------------------------+
ubuntu@uc-osclient-01:~$ neutron subnet-list
+--------------------------------------+-------------------+----------------+------------------------------------------------+
| id                                   | name              | cidr           | allocation_pools                               |
+--------------------------------------+-------------------+----------------+------------------------------------------------+
| 725c3dbe-65f2-4dc5-b9ec-596309bc2229 | vlan_13-subnet | 172.16.5.64/27 | {"start": "172.16.5.66", "end": "172.16.5.94"} |
| 7b799b92-ba55-454c-8db7-558476559b4e | vlan_12-subnet | 172.16.5.32/27 | {"start": "172.16.5.34", "end": "172.16.5.62"} |
+--------------------------------------+-------------------+----------------+------------------------------------------------+
~~~

We can see their network namespaces.

~~~
root@uc-neutron-api-01:~# ip netns list
qdhcp-5e73a2f5-a52b-4833-9978-531fc98cd783
qdhcp-eb175122-2500-4234-becb-030da538893a
~~~

And with a couple of virtual machines deployed we can see what interfaces OVS has created.

I should note that the neutron-api server is an lxc container. eth2 in the container is bridged to a bonded interface in the baremetal OS.

~~~
root@uc-neutron-api-01:~# ovs-vsctl show
58669c80-e22a-4ea8-9b2d-2d21d1c95163
    Bridge br-provider
        Port phy-br-provider
            Interface phy-br-provider
                type: patch
                options: {peer=int-br-provider}
        Port "eth2"
            Interface "eth2"
        Port br-provider
            Interface br-provider
                type: internal
    Bridge br-int
        fail_mode: secure
        Port br-int
            Interface br-int
                type: internal
        Port int-br-provider
            Interface int-br-provider
                type: patch
                options: {peer=phy-br-provider}
        Port "tapd404a48d-df"
            tag: 3
            Interface "tapd404a48d-df"
                type: internal
        Port "tapa02b5817-23"
            tag: 2
            Interface "tapa02b5817-23"
                type: internal
    ovs_version: "2.5.0"
~~~

On the compute node OVS looks like this. br-provider sits on bond0 and adds VLAN tags.

~~~
root@compute-01:/home/ubuntu# ovs-vsctl show
543d058b-8452-4902-bc4c-77bd62ae3e07
    Bridge br-int
        fail_mode: secure
        Port int-br-provider
            Interface int-br-provider
                type: patch
                options: {peer=phy-br-provider}
        Port br-int
            Interface br-int
                type: internal
        Port "qvo60f707e9-0d"
            tag: 3
            Interface "qvo60f707e9-0d"
        Port "qvoe77743d9-e1"
            tag: 2
            Interface "qvoe77743d9-e1"
        Port "qvoaad386ab-2f"
            tag: 2
            Interface "qvoaad386ab-2f"
    Bridge br-provider
        Port "bond0"
            Interface "bond0"
        Port phy-br-provider
            Interface phy-br-provider
                type: patch
                options: {peer=int-br-provider}
        Port br-provider
            Interface br-provider
                type: internal
    ovs_version: "2.5.0"
~~~

If I tcpdump bond0 on the compute node and ping an instance on that provider network...

~~~
root@compute-01:/home/ubuntu# tcpdump -n -e -ttt -i bond0 | grep "vlan 12"
tcpdump: WARNING: bond0: no IPv4 address assigned
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on bond0, link-type EN10MB (Ethernet), capture size 65535 bytes

00:00:00.000848 fa:16:3e:c3:6f:17 > cc:37:ab:2c:9c:92, ethertype 802.1Q (0x8100), length 102: vlan 12, p 0, ethertype IPv4, 172.16.5.35 > 172.16.4.4: ICMP echo reply, id 19867, seq 1, length 64
00:00:00.136563 cc:37:ab:2c:9c:92 > fa:16:3e:c3:6f:17, ethertype 802.1Q (0x8100), length 102: vlan 12, p 0, ethertype IPv4, 172.16.4.4 > 172.16.5.35: ICMP echo request, id 19867, seq 2, length 64
00:00:00.000225 fa:16:3e:c3:6f:17 > cc:37:ab:2c:9c:92, ethertype 802.1Q (0x8100), length 102: vlan 12, p 0, ethertype IPv4, 172.16.5.35 > 172.16.4.4: ICMP echo reply, id 19867, seq 2, length 64
00:00:00.999631 cc:37:ab:2c:9c:92 > fa:16:3e:c3:6f:17, ethertype 802.1Q (0x8100), length 102: vlan 12, p 0, ethertype IPv4, 172.16.4.4 > 172.16.5.35: ICMP echo request, id 19867, seq 3, length 64
~~~

Looks good to me. :)

## Ansible OpenStack network modules

Ansible 2.1 has modules for OpenStack networks and subnets that work quite well. The modules have been around for a while, but 2.1 added a couple required features for adding a provider network.

Here's a snippet of creating a provider network. The networks dict is a yaml dict that I'm working on to define the entire network stack.

{% raw %}
~~~
    - name: ensure provider networks exist
      os_network:
        name: "{{ item.key }}"
        provider_network_type: "vlan"
        provider_physical_network: "provider"
        provider_segmentation_id: "{{ item.value.vlan_id }}"
        shared: True
      delegate_to: uc-osclient-01
      when: item.value.type == "provider" 
      with_dict: networks

    - name: ensure provider subnets exist
      os_subnet:
        network_name: "{{ item.key }}"
        name: "{{ item.key }}-subnet" 
        cidr: "{{ item.value.network }}{{ item.value.cidr }}" 
        gateway_ip: "{{ item.value.address }}"
      delegate_to: uc-osclient-01
      when: item.value.type == "provider" 
      with_dict: networks
~~~
{% endraw %}

It's so nice to be able to automate the creation of the entire network stack, from Cumulus and bonds all the way up to neutron subnets.

## Interfaces down?

One thing that really confused me when working on this deployment was that the interfaces are marked down.

~~~
root@compute-01:/home/ubuntu# ip ad sh | grep DOWN | grep "ovs\|br-"
15: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default 
16: br-int: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default 
17: br-provider: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default 
~~~

I'm not clear on why these are down, but everything is working fine. This is something I would like to understand.

~~~
root@compute-01:/home/ubuntu# ovs-ofctl show br-provider
OFPT_FEATURES_REPLY (xid=0x2): dpid:000090e2babace6c
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
 1(bond0): addr:90:e2:ba:ba:ce:6c
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 2(phy-br-provider): addr:ea:dc:cb:d8:23:25
     config:     0
     state:      0
     speed: 0 Mbps now, 0 Mbps max
 LOCAL(br-provider): addr:90:e2:ba:ba:ce:6c
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
~~~

Again, not sure why the LOCAL(br-povider) is down, but it is, and everything works fine.

## Instances

Here's the routing table of a virtual machine running in the OpenStack cloud using a provider network.

~~~
ubuntu@t4:~$ netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         172.16.5.33     0.0.0.0         UG        0 0          0 eth0
169.254.169.254 172.16.5.34     255.255.255.255 UGH       0 0          0 eth0
172.16.5.32     0.0.0.0         255.255.255.224 U         0 0          0 eth0
~~~

Note the metadata server route at 169.254.169.254.

## Conclusion

If you are deploying a private cloud into an environment where software defined networking and/or overlays are not welcome, then provider networks might be your only option. If you are old-school OpenStack and liked nova-network, then provider networks will seem similar. I do like their simplicity. 

Not every OpenStack deployment is going to require hundreds or thousands of private tenant networks. If you have an OpenStack-hostile network environment, which is quite common I assure you, then this might help. Hopefully they'll at least allow you to have neutron manage DHCP on the provider networks.

Next up I need to look into IPv6 and Open vSwitch DPDK as well as performance testing and a host of other items.
