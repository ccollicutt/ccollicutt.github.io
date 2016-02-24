---
layout: post
title: Trying OSPF with Quagga and OpenBGP
header_image: /img/quagga.jpg
categories:
---

# {{ page.title }}

I've recently being doing some research into Equal Cost Multipath Routing (ECMP) as well as Open Shortest Path First (OSPF).

The first thing that I should note is that I have no idea really what I'm doing...I'm just messing around with OSPF really.

I put together a quick [Vagrant + Ansible](https://github.com/ccollicutt/ansible-ospf-test) test setup that uses a single OpenBSD router, two Ubuntu Trusty servers and a Trusty client instance. That means the Vagrantfile I created will setup four virtual machines. I used OpenBSD's ospfd (part of [OpenBGP](http://www.openbgpd.org/)) and [Quagga](http://www.nongnu.org/quagga/) on the Ubuntu hosts.

## Get started

<pre>
<code>you@workstation$ git clone https://github.com/ccollicutt/ansible-ospf-test
you@workstation$ cd ansible-ospf-test
you@workstation$ vagrant up
SNIP!
you@workstation$ $ ansible -m ping all
router | success >> {
    "changed": false,
    "ping": "pong"
}

client | success >> {
    "changed": false,
    "ping": "pong"
}

zone0 | success >> {
    "changed": false,
    "ping": "pong"
}

zone1 | success >> {
    "changed": false,
    "ping": "pong"
}
you@workstation$ ansible-playbook site.yml
SNIP!
PLAY RECAP ********************************************************************
client                     : ok=3    changed=1    unreachable=0    failed=0
router                     : ok=11   changed=8    unreachable=0    failed=0
zone0                      : ok=9    changed=7    unreachable=0    failed=0
zone1                      : ok=9    changed=7    unreachable=0    failed=0
</code>
</pre>

## Routes

Once that is done we should be able to login to the client and see the default route has changed to 10.0.10.2.

<pre>
<code>curtis# vagrant ssh client
SNIP!
vagrant@client:~$ ip ro sh
default via 10.0.10.2 dev eth1
10.0.2.0/24 dev eth0  proto kernel  scope link  src 10.0.2.15
10.0.10.0/24 dev eth1  proto kernel  scope link  src 10.0.10.10
</code>
</pre>

We should be able to ping 172.0.3.10 from the client, and it'll go through the OpenBSD router, which is at 10.0.10.2.

<pre>
<code>vagrant@client:~$ ping -c 1 -w 1 172.0.3.10
PING 172.0.3.10 (172.0.3.10) 56(84) bytes of data.
64 bytes from 172.0.3.10: icmp_seq=1 ttl=63 time=0.696 ms

--- 172.0.3.10 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.696/0.696/0.696/0.000 ms
</code>
</pre>

The zone servers should have their default gateway set to the internal IP of the OpenBSD router.

<pre>
<code>vagrant@zone0:~$ ip ro sh
default via 172.0.1.2 dev eth1
10.0.2.0/24 dev eth0  proto kernel  scope link  src 10.0.2.15
172.0.1.0/24 dev eth1  proto kernel  scope link  src 172.0.1.10
</code>
</pre>

The router's table ends up looking like this:

<pre>
<code>$ route -nv show -inet
Routing tables

Internet:
Destination        Gateway            Flags   Refs      Use   Mtu  Prio Iface Label
default            10.0.2.2           UGS        0     6499     -     8 em0   DHCLIENT 16899
10.0.2/24          link#1             UC         1        0     -     4 em0
10.0.2.2           52:54:00:12:35:02  UHLc       2     1060     -     4 em0
10.0.2.15          08:00:27:53:fe:30  UHLl       0        0     -     1 lo0
10.0.10/24         link#2             UC         0        0     -     4 em1
10.0.10.2          08:00:27:b5:8a:87  UHLl       0        0     -     1 lo0
127/8              127.0.0.1          UGRS       0        4 32768     8 lo0
127.0.0.1          127.0.0.1          UH         1        0 32768     4 lo0
172.0.1/24         link#3             UC         2        0     -     4 em2
172.0.1/24         172.0.1.2          UG         0        0     -    32 em2
172.0.1.2          08:00:27:99:55:cd  UHLl       1        0     -     1 lo0
172.0.1.10         08:00:27:e0:98:1b  UHLc       1       12     -     4 em2
172.0.1.11         08:00:27:b6:ef:74  UHLc       1        9     -     4 em2
172.0.3.10/32      172.0.1.10         UGP        0       27     -    32 em2
172.0.3.10/32      172.0.1.11         UGP        0        0     -    32 em2
224/4              127.0.0.1          URS        0        0 32768     8 lo0
</code>
</pre>


## Configuration

Once Ansible has finished, the quagga ospfd.conf file looks like this on both zone servers:

<pre>
<code>$ vagrant ssh zone1
SNIP!
vagrant@zone1:~$ sudo cat /etc/quagga/ospfd.conf
! -*- ospf v10 -*-
![]()
hostname zone1
password zebra
enable password zebra
![]()
interface eth1
![]()
router ospf
  router-id 172.0.1.11
  network 172.0.1.11/24 area 0.0.0.0
  network 172.0.3.0/24 area 0.0.0.0
log file /var/log/quagga/ospfd.log

</code>
</pre>

The ospfd.conf file on the OpenBSD box looks like this:

<pre>
<code>$ sudo cat /etc/ospfd.conf
router-id 5.5.5.5

area 0.0.0.0 {
  interface em2 {
  }
}
</code>
</pre>

## ospf neighbors

<pre>
<code>$ sudo ospfctl show neighbor
ID              Pri State        DeadTime Address         Iface     Uptime
172.0.1.11      1   FULL/BCKUP   00:00:33 172.0.1.11      em2       02:19:25
172.0.1.10      1   FULL/OTHER   00:00:33 172.0.1.10      em2       02:19:25
</code>
</pre>


## The fun part

Now with all that setup I can start pinging 172.0.3.11, an IP that is on both of the zone servers, then delete it from one and pings will switchover to the other zone server.

<pre>
<code>vagrant@client:~$ cat ping.sh
#!/bin/bash
COUNTER=0
while true; do
  if ping -c 1 -w 1 172.0.3.10 > /dev/null; then
  echo "$COUNTER - 172.0.3.10 is alive"
  else
  echo "$COUNTER - 172.0.3.10 is dead"
  fi
  let COUNTER=COUNTER+1
  sleep 1
done
vagrant@client:~$ ./ping.sh
0 - 172.0.3.10 is alive
1 - 172.0.3.10 is alive
2 - 172.0.3.10 is alive
3 - 172.0.3.10 is alive
4 - 172.0.3.10 is alive
5 - 172.0.3.10 is alive
6 - 172.0.3.10 is alive
7 - 172.0.3.10 is alive
8 - 172.0.3.10 is dead
9 - 172.0.3.10 is alive
10 - 172.0.3.10 is alive
11 - 172.0.3.10 is alive
12 - 172.0.3.10 is alive
13 - 172.0.3.10 is alive
^C
</code>
</pre>

The part where we cat 172.0.3.10 being dead is when I drop it on the zone server currently being used by the OpenBSD router table.

<pre>
<code>vagrant@zone1:~$ sudo ip add del 172.0.3.10/32 dev eth1
</code>
</pre>

Routing table looks like this now:

<pre>
<code># route -nv show -inet | grep "172.0.3.10"
172.0.3.10/32      172.0.1.10         UGP        0        9     -    32 em2
</code>
</pre>

## tcpdump and ospf

Dumping OSPF is pretty easy...

<pre>
<code># tcpdump -n -e -ttt -i em2 proto ospf
tcpdump: listening on em2, link-type EN10MB
tcpdump: WARNING: compensating for unaligned libpcap packets
Feb 02 23:22:47.527622 08:00:27:b6:ef:74 01:00:5e:00:00:05 0800 78: 172.0.1.11 > 224.0.0.5: OSPFv2-hello  44: rtrid 172.0.1.11 backbone [tos 0xc0] [ttl 1]
Feb 02 23:22:47.527988 08:00:27:99:55:cd 01:00:5e:00:00:05 0800 86: 172.0.1.2 > 224.0.0.5: OSPFv2-hello  52: rtrid 5.5.5.5 backbone dr 172.0.1.2 bdr 172.0.1.10 [tos 0xc0] [ttl 1]
Feb 02 23:22:47.528136 08:00:27:99:55:cd 01:00:5e:00:00:05 0800 94: 172.0.1.2 > 224.0.0.5: OSPFv2-ls_upd  60: rtrid 5.5.5.5 backbone [tos 0xc0] [ttl 1]
SNIP!
</code>
</pre>

## Issues

- I could not get md5/crypt authentication working between the OpenBSD server running ospfd (part of [OpenBGP](http://www.openbgpd.org/)) and [Quagga](http://www.nongnu.org/quagga/). Not sure what the problem was, there are no examples I could find of getting this working. Ran out of time to test, so for now there is no authentication.

## Ping fun

So that was a lot of work to do a simple ping. Kinda fun though. I haven't done a lot with networking, but as I work more with systems such as OpenStack and containers and such I realize how important it is and how much I'm missing out by not knowing more. This was a nice, quick little foray into OSPF.

I don't know how much anyone will get from reading this post, but I did learn a few things myself, and perhaps at least it'll show that getting a quick OSPF environment up for playing around with isn't that difficult. (Though perhaps interacting between Linux and Quagga and OpenBSD and OpenBGP isn't that easy, maybe I should have just gone 100% Ubuntu for this example.)

I had hoped to learn a bit about ECMP as well, but that didn't quite materialize. OpenBSD does support ECMP, but I'm not sure how it would work in this example. Certainly with OpenBSD it's possible to route over multiple equal interfaces; most of the examples are for when you have multiple uplinks, eg. more than one Internet provider.

Hopefully I don't lead anyone down the wrong path with this blog post, but I like to write up little projects like this to help me determine what I learned and what I did not. Sometimes you get into something to find out how much you don't know. :)
