---
layout: post
title: OpenStack - Bridge not replying to ARP Requests
categories:
header_image: /img/hw-ss-network.png
---

_(a super simple diagram to give you an idea of the network layout)_

# {{ page.title }}

I've been working on my [SimpleStack](https://github.com/ccollicutt/simplestack) OpenStack install lately. This design currently uses a FlatDHCP nova-network.

I have a couple of hardware servers that I'm deploying it on at work, and I was having a ton of trouble with the public and flat interface being able to receive arp requests, but not sending them back.

It turns out the answer was simple: __the vlan coming down the port to the interface was tagged.__

This is part of the nova.conf file:

<pre>
<code>flat_network_bridge = br100
flat_interface = em2
public_interface = em2
</code>
</pre>

Here's some ARP traffic off the em2 interface.

<pre>
<code>curtis@compute1:~$ sudo tcpdump -n -e -ttt -i br100 arp
00:00:00.888602 5c:5e:ab:d9:e0:f0 > ff:ff:ff:ff:ff:ff, ethertype 802.1Q (0x8100), length 60: vlan 601, p 0, ethertype ARP, Request who-has x.y.z.22 tell x.y.z.18, length 42
00:00:00.900232 5c:5e:ab:d9:e0:f0 > ff:ff:ff:ff:ff:ff, ethertype 802.1Q (0x8100), length 60: vlan 601, p 0, ethertype ARP, Request who-has x.y.z.22 tell x.y.z.18, length 42
00:00:00.600620 5c:5e:ab:d9:e0:f0 > ff:ff:ff:ff:ff:ff, ethertype 802.1Q (0x8100), length 60: vlan 601, p 0, ethertype ARP, Request who-has x.y.z.22 tell x.y.z.18, length 42
00:00:00.703113 5c:5e:ab:d9:e0:f0 > ff:ff:ff:ff:ff:ff, ethertype 802.1Q (0x8100), length 60: vlan 601, p 0, ethertype ARP, Request who-has x.y.z.22 tell x.y.z.18, length 42
</code>
</pre>

As you can see, we have "ethertype 802.1Q" and vlan 601. It took me quite a while to notice that, but once I did everything made sense.

The arp requests were getting to em2, but because they were tagged with vlan 601 the server wouldn't reply because it wasn't setup to use that vlan.

As soon as I added the 601 vlan and configured nova to use it, everything started working.

<pre>
<code>curtis@compute1:~$ sudo modprobe 8021q
curtis@compute1:~$ sudo vconfig add em2 601
Added VLAN with VID == 601 to IF -:em2:-
curtis@compute1:~$ sudo ifconfig em2.601 up
</code>
</pre>
The I changed the nova.conf file to look like this:

<pre>
<code>flat_network_bridge = br100
flat_interface = em2.601
public_interface = em2.601
</code>
</pre>

Then I restarted nova-network.

Finally I ended up with this bridge config:

<pre>
<code>curtis@compute1:/etc/nova# sudo brctl show
bridge name	bridge id		STP enabled	interfaces
br100		8000.782bcb617008	no		em2.601
              vnet0
virbr0		8000.000000000000	yes
</code>
</pre>

And I was able to have my flat network, which is all public IPs, working.

## Tagged and untagged, getting lost in details

Now, there's no reason (in this situation) for this port to have the vlan tagged on it, we just need that one network on the physical interface, but it was tagged and in this case it was just easier to go with that then to try to have any changes made on the network operator side.

For a while we were chasing red herrings on this problem, and finally I just took a hard look at the arp packets and realized it was a tagged vlan. Obvious now, but it took a while to figure out just how obvious.

It was a good feeling to realize the problem, fix it, and to get SimpleStack up and running on physical computers with public IPv4 addresses so that I can continue to experiment with small OpenStack installations. I've already realized this network design probably wouldn't work for anyone who doesn't have a lot of IPv4 addresses or can't support IPv6. IPv6 would be so great for this. When will we really start using it...ISPs I'm looking at you...
