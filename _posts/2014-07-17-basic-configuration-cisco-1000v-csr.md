---
layout: post
title: Basic Configuration of a Cisco 1000V CSR
categories: networking

---

![](/img/virtnet.png)
 
# {{ page.title }}

In a "previous post":/2014/07/14/cisco-1000v-csr-libvirt-kvm.html I looked at installing the Cisco 1000V Cloud Services Router into a KVM environment. Now let's do some basic configuration. Again I have to note that I'm not a professional Cisco network administrator by any stretch, and in fact it would be great if readers noticed mistakes or better ways of doing things and let me know in the comments. :)

## %{color:red}Issue%

At this time TCP seems busted with CSR + KVM. I'm not sure why at this time. ICMP seems to work but not TCP.

Troubleshooting steps so far:

1. Insert a simple Linux router: This worked fine, ~20 gbps with iperf
2. Try out this process in virtual box on a Windows computer: Also worked as expected
3. Try out on a Ubuntu 12.04.4 system--still no dice. I would have expected this to work as it is a supported environment for the CSR. Perhaps some configuration issue with KVM...
4. Also tried the qcow2 image instead of installing from the ISO, no joy

So I'm not sure at this time what the issue is. Something to do with the CSR in this particular environment, be it misconfiguration on my part, or something else. I'll update when I figure out what is going on. If you do try it and get the expected performance (such as 50000 kbps when the premium license is enabled) please let me know.

Even though the KVM environments aren't yet working, all the configuration information below should be Ok (I think). I did run this exact same process in Windows with Virtualbox and that worked fine. Also, like I said above, a simple Linux router works as well. However, when using the CSR and KVM, everything runs Ok, it's just that TCP isn't working. Very weird.

My plan is to pass this post around a bit and see if anyone has any ideas.

## The environment

I'm working in an Ubuntu Trusty 14.04 KVM single host environment. I'm also using he default Openvswitch that comes with Trusty, and of course I'm using libvirt.

For now I'll connect the CSR to three virtual networks within the KVM host, two of which are managed by openvswitch and one by libvirt.

<pre>
<code># ovs-vsctl list-br
corebr0
distbr0
# brctl show  |grep virbr0
virbr0		8000.fe540021831d	yes		vnet0
</code>
</pre>

So the router will have three interfaces:

1. "virbr0" - Default libvirt network, Linux bridge
2. "corebr0" - Openvswitch
3. "distbr0" - Openvswitch

I altered the libvirt default network to start providing DHCP addresses at 192.168.122.3 instead of the default 192.168.122.2 so that the router can have 192.168.122.2 as it's IP on the default virbr0 bridge.

## Basic configuration

First, turn off domain lookups, otherwise the route will think mistyped commands are potential domains and spend time looking them up.

<pre>
<code>Router>enable
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#no ip domain-lookup
</code>
</pre>

Here we can see all the interfaces and their mapping. (Note that the MAC addresses might be different from other examples in this post--that's Ok, I've done this same process several times.)

<pre>
<code>Router#show platform software vnic-if interface-mapping
-------------------------------------------------------------
 Interface Name        Driver Name         Mac Addr
-------------------------------------------------------------
 GigabitEthernet3       virtio             5254.0064.67e2 
 GigabitEthernet2       virtio             5254.0039.2c47 
 GigabitEthernet1       virtio             5254.0006.685d 
-------------------------------------------------------------
</code>
</pre>

Configure the first interface.

<pre>
<code>Router(config)#int gigabit 1
Router(config-if)#ip address 192.168.122.2 255.255.255.0
Router(config-if)#no shutdown
Router(config-if)#exit
</code>
</pre>

Try pinging 192.168.122.2.

<pre>
<code>Router#ping 192.168.122.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.122.2, timeout is 2 seconds:
![](!!!)
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms
</code>
</pre>

Configure the second interface.

<pre>
<code>Router(config)#int gigabit 2
Router(config-if)#ip address 10.100.0.1 255.255.255.0
Router(config-if)#no shutdown
Router(config-if)#exit
</code>
</pre>

Configure the third interface.

<pre>
<code>Router(config)#int gigabit 3
Router(config-if)#ip address 10.100.1.1 255.255.255.0
Router(config-if)#no shutdown
Router(config-if)#exit
</code>
</pre>

Show the routes.

<pre>
<code>Router#sh ip route   
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        10.100.0.0/24 is directly connected, GigabitEthernet2
L        10.100.0.1/32 is directly connected, GigabitEthernet2
C        10.100.1.0/24 is directly connected, GigabitEthernet3
L        10.100.1.1/32 is directly connected, GigabitEthernet3
      192.168.122.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.122.0/24 is directly connected, GigabitEthernet1
L        192.168.122.2/32 is directly connected, GigabitEthernet1
</code>
</pre>

## Configure DHCP servers

I'm going to put DHCP servers on gigabit 2 and gigabit 3, core and dist networks respectfully.

<pre>
<code>Router(config)#ip dhcp pool core
Router(dhcp-config)#network 10.100.0.0 /24
Router(dhcp-config)#default-router 10.100.0.1
Router(dhcp-config)#exit
Router(config)#ip dhcp excluded-address 10.100.0.1 10.100.0.100
</code>
</pre>

Check the pool.

<pre>
<code>Router#sh ip dhcp pool

Pool core :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 0
 Excluded addresses             : 100
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased/Excluded/Total
 10.100.0.1           10.100.0.1       - 10.100.0.254      0     / 100   / 254  
</code>
</pre>

Do the same on gigabit 3 with network 10.100.1.0/24.

## Add virtual machines to the networks

I want to add a server onto the core network, expecting that the CSR will hand it a DHCP address.

<pre>
<code># virsh list  |grep core
 4     core-srv1                      running
</code>
</pre>

Now that that's up, let's see if it gets an IP from the router.

<pre>
<code>Router#sh ip dhcp pool   

Pool core :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 1
 Excluded addresses             : 100
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased/Excluded/Total
 10.100.0.102         10.100.0.1       - 10.100.0.254      1     / 100   / 254 
Router#sh arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.100.0.1              -   5254.0013.bd53  ARPA   GigabitEthernet2
Internet  10.100.0.2              1   b60d.1368.3044  ARPA   GigabitEthernet2
Internet  10.100.0.101            1   5254.00db.4454  ARPA   GigabitEthernet2
Internet  10.100.1.1              -   5254.001e.fceb  ARPA   GigabitEthernet3
Internet  192.168.122.2           -   5254.0021.831d  ARPA   GigabitEthernet1
</code>
</pre>

As can be seen the KVM virtual machine received IP 10.100.0.101, and we can ping that from the router.

<pre>
<code>Router#ping 10.100.0.101
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.0.101, timeout is 2 seconds:
![](!!!)
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
</code>
</pre>

I can access the console of the vm using "virsh console".

<pre>
<code># virsh console core-srv1
</code>
</pre>

I've also started a second virtual machine on the dist network.

<pre>
<code># virsh list | grep dist
 13    dist-srv1                      running
</code>
</pre>

From core-srv1 I can ping dist-srv1, and we are using the router to manage the traffic. Notice I am using -I due to the TCP issue I mention at the start of the post.

<pre>
<code>ubuntu@core-srv1:~$ traceroute -I 10.100.1.101
traceroute to 10.100.1.101 (10.100.1.101), 64 hops max
  1   10.100.0.1  0.736ms  0.601ms  0.494ms 
  2   10.100.1.101  0.896ms  0.663ms  0.646ms 
ubuntu@core-srv1:~$ ip route show
default via 10.100.0.1 dev eth0 
10.100.0.0/24 dev eth0  proto kernel  scope link  src 10.100.0.101 
</code>
</pre>

We can see the mac addresses of the virtual machines in the routers arp table, including dist-srv1 and core-srv1.

<pre>
<code>Router#sh arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.100.0.1              -   5254.0013.bd53  ARPA   GigabitEthernet2
Internet  10.100.0.101          172   5254.00db.4454  ARPA   GigabitEthernet2
Internet  10.100.1.1              -   5254.001e.fceb  ARPA   GigabitEthernet3
Internet  10.100.1.2             52   7e65.7502.9f41  ARPA   GigabitEthernet3
Internet  10.100.1.101            4   5254.00ec.b81e  ARPA   GigabitEthernet3
Internet  192.168.122.1          91   5254.0073.6d2e  ARPA   GigabitEthernet1
Internet  192.168.122.2           -   5254.0021.831d  ARPA   GigabitEthernet1
</code>
</pre>

## License

By default the unlicensed CSR is pretty limited bandwidth-wise.

<pre>
<code>Router#show platform hardware throughput level
The current throughput level is 2500 kb/s
</code>
</pre>

Use the standard level evaluation license. (I think.)

<pre>
<code>Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#license boot level standard
	 Feature Name:prem_eval

SNIP!

ACCEPT? (yes/[no]): yes

*Jul 18 19:08:12.085: %LICENSE-6-EULA_ACCEPTED: EULA for feature prem_eval 1.0 has been accepted. UDI=CSR1000V:9L7UG7XECKE; StoreIndex=0:Built-In License Storage% use 'write' command to make license boot config take effect on next boot
</code>
</pre>

Reload.

<pre>
<code>Router#reload
</code>
</pre>

As can be seen, the license level has been changed, throughput is now 50000 kbps.

<pre>
<code>SNIP!
*Jul 18 19:10:51.476: %SMART_LIC-6-AGENT_READY: Smart Agent for Licensing is initialized
*Jul 18 19:10:51.481: %VUDI-6-EVENT: [serial number: 9L7UG7XECKE], [vUDI: ], vUDI is successfully retrieved from license file
*Jul 18 19:10:51.684: %IOS_LICENSE_IMAGE_APPLICATION-6-LICENSE_LEVEL: Module name = csr1000v Next reboot level = standard and License = prem_eval
*Jul 18 19:10:52.557: %VXE_THROUGHPUT-6-LEVEL: Throughput level has been set to 50000 kbps
SNIP!
Router#sh platform hardware throughput level
The current throughput level is 50000 kb/s
</code>
</pre>

## Shutdown

I just wanted to know that "virsh shutdown" will halt this router just fine, and this message will appear on the router console.

<pre>
<code>*Jul 19 16:36:44.836: %IOSXE-5-PLATFORM: F0: shutdown: shutting down for system halt
</code>
</pre>


## Iperf

I would like to run iperf between the hosts, but given the weird issue I'm seeing on KVM hosts, I can't.

## Conclusion

So here we have a super basic virtual network setup that is routed by a Cisco 1000V CSR router. What I've done here could easily be accomplished with a simple firewall or other basic router, but I think it's a good step towards getting a test environment setup with a Cisco router.

Again, it must be noted that this is not working yet in a KVM environment (Ubuntu 12.04 or 14.04) but does work with Windows + Virtualbox.

