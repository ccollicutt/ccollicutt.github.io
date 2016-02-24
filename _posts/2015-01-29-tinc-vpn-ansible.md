---
layout: post
title: Tinc VPN (and Ansible)
header_image: /img/tinc-ansible.jpg
categories:
---

# {{ page.title }}

VPNs are a pain. The more I work with technology the more I see how some things like VPNs are just poor constructs and are, in a way, workarounds. Maybe that's all IT is...workarounds. I don't know. This is philosophical. At any rate, VPNs are difficult to setup, secure, and use, so sometimes it's fun to play with technologies that try to make things easier, and I think tinc fits into that category. Would I use it in production? Probably not, but it's still good to take a look at other implementations and see what can be done.

## Tinc

I think the most interesting thing about [tinc](http://www.tinc-vpn.org/) is that it sets up a mesh.

> Regardless of how you set up the tinc daemons to connect to each other, VPN traffic is always (if possible) sent directly to the destination, without going through intermediate hops.

Also, it's easy to extend.

> When you want to add nodes to your VPN, all you have to do is add an extra configuration file, there is no need to start new daemons or create and configure new devices or network interfaces

So those are a couple of good reasons to try out tinc.

## Tinc with Ansible

I was playing around with tinc a few days ago and came up with an [Ansible playbook](https://github.com/ccollicutt/ansible-tinc) to setup a tinc VPN between several Ubuntu Trusty servers.

Right now I have four virtual machines running in the same OpenStack infrastructure, which is a FlatDHCP system, so the tenants don't share a private network...unless we set one up for them with Tinc. (This is not something you'd normally want for production systems...I don't think, I'm just messin' around.)

<pre>
<code>curtis@parker:~/tinc-testing$ cat hosts
trusty1 ansible_ssh_host=x.y.z.21
trusty2 ansible_ssh_host=x.y.z.22
trusty3 ansible_ssh_host=x.y.z.23
trusty4 ansible_ssh_host=x.y.z.24
</code>
</pre>

I can ping all those with Ansible.

<pre>
<code>curtis@parker:~/tinc-testing$ ansible -m ping all
trusty3 | success >> {
    "changed": false,
    "ping": "pong"
}

trusty4 | success >> {
    "changed": false,
    "ping": "pong"
}

trusty1 | success >> {
    "changed": false,
    "ping": "pong"
}

trusty2 | success >> {
    "changed": false,
    "ping": "pong"
}
</code>
</pre>

And I can run the Ansible playbook that sets up one VPN, called "vpnone."

<pre>
<code>curtis@parker:~/tinc-testing$ ansible-playbook site.yml
SNIP!
PLAY RECAP ********************************************************************
trusty1                    : ok=17   changed=1    unreachable=0    failed=0
trusty2                    : ok=17   changed=1    unreachable=0    failed=0
trusty3                    : ok=17   changed=1    unreachable=0    failed=0
trusty4                    : ok=17   changed=1    unreachable=0    failed=0
</code>
</pre>

Now Tinc has setup a tun0 on each node.

<pre>
<code>curtis@parker:~/tinc-testing$ ansible -a "ip ad sh tun0" all
trusty3 | success | rc=0 >>
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 500
    link/none
    inet 10.0.0.23/24 scope global tun0
       valid_lft forever preferred_lft forever

trusty2 | success | rc=0 >>
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 500
    link/none
    inet 10.0.0.22/24 scope global tun0
       valid_lft forever preferred_lft forever

trusty1 | success | rc=0 >>
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 500
    link/none
    inet 10.0.0.21/24 scope global tun0
       valid_lft forever preferred_lft forever

trusty4 | success | rc=0 >>
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 500
    link/none
    inet 10.0.0.24/24 scope global tun0
       valid_lft forever preferred_lft forever
</code>
</pre>

And I can ping nodes from one another. In this example I just ping 10.0.0.24 from all four nodes.

<pre>
<code>curtis@parker:~/tinc-testing$ ansible -m shell -a "ping -c 1 -w 1 10.0.0.24" all
trusty2 | success | rc=0 >>
PING 10.0.0.24 (10.0.0.24) 56(84) bytes of data.
64 bytes from 10.0.0.24: icmp_seq=1 ttl=64 time=0.685 ms

--- 10.0.0.24 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.685/0.685/0.685/0.000 ms

trusty4 | success | rc=0 >>
PING 10.0.0.24 (10.0.0.24) 56(84) bytes of data.
64 bytes from 10.0.0.24: icmp_seq=1 ttl=64 time=0.029 ms

--- 10.0.0.24 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.029/0.029/0.029/0.000 ms

trusty3 | success | rc=0 >>
PING 10.0.0.24 (10.0.0.24) 56(84) bytes of data.
64 bytes from 10.0.0.24: icmp_seq=1 ttl=64 time=0.501 ms

--- 10.0.0.24 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.501/0.501/0.501/0.000 ms

trusty1 | success | rc=0 >>
PING 10.0.0.24 (10.0.0.24) 56(84) bytes of data.
64 bytes from 10.0.0.24: icmp_seq=1 ttl=64 time=0.702 ms

--- 10.0.0.24 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.702/0.702/0.702/0.000 ms

</code>
</pre>

By default the playbook sets up "vpnone" but the name is configurable. But it only does one VPN right now, though tinc can support many.

<pre>
<code>ubuntu@trusty1:~$ ls /etc/tinc/vpnone/
hosts  rsa_key.priv  tinc.conf  tinc-down  tinc-up
ubuntu@trusty1:~$ ls /etc/tinc/vpnone/hosts
trusty1  trusty2  trusty3  trusty4
ubuntu@trusty1:~$ ls /etc/tinc/
nets.boot  vpnone
</code>
</pre>

## Firewall rules

I should note that tcp/udp on port 655 needs to be open between the nodes.

## So there you go

If you want a VPN setup between several hosts, then tinc is a good way to do that, and if you want to use my [playbook](https://github.com/ccollicutt/ansible-tinc) to automatically setup a single vpn between several hosts, then that'd be great. The playbook is not perfect, so if you see something you'd like changed, just let me know or send a pull request. :)

Happy tincing!
