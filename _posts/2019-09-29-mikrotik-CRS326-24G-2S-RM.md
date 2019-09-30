---
layout: post
title: Mikrotik CRS326-24G-2S+RM
categories:
header_image: "/img/CRS326-24G-2S+RM.png"

---

# {{ page.title }}

I have a bunch of loud noisy switches...*cough* Juniper EX4200s *cough*. I recently moved and haven't had the time yet to setup proper sound proofing for all my homelab gear and thus I need to be quieter where I can be. Lower power consumption would help too.

So I bought a [Mikrotik CRS326-24G-2S+RM](https://mikrotik.com/product/CRS326-24G-2SplusRM) to be my main 1GB switch. I purchased the switch from [Solimedia](https://solimedia.net/) and it was about $250 CDN.

## Power Consumption and Volume

Right now the switch has about 20 ports populated and is doing a fair amount of work. It's pulling only 17 watts. My EX4200 pull about 135 watts; that's a substantial difference. Based on my calculations, running an EX4200 24/7 would cost about $8.70/month in Ontario and the CRS326 about $1.50.  

The switch also has no fans, so it's virtually silent.

## Configuration

The challenging part of this switch is that Mikrotik has many different products, and they use many different CPUs and ASICs. 

In their C3XX line you configure VLANs differently than their other switches, as they do hardware VLANS. They have some [docs](https://wiki.mikrotik.com/wiki/Manual:Interface/Bridge#VLAN_Example_.231_.28Trunk_and_Access_Ports.29) on configuration. These short docs are not detailed, but they are accurate in my experience.

Example 3 is probably the one most people would use..."VLAN Example #3 (InterVLAN Routing by Bridge)":

```
/interface bridge
add name=bridge1 vlan-filtering=yes

/interface bridge port
add bridge=bridge1 interface=ether6 pvid=200
add bridge=bridge1 interface=ether7 pvid=300
add bridge=bridge1 interface=ether8 pvid=400

/interface bridge vlan
add bridge=bridge1 tagged=bridge1 untagged=ether6 vlan-ids=200
add bridge=bridge1 tagged=bridge1 untagged=ether7 vlan-ids=300
add bridge=bridge1 tagged=bridge1 untagged=ether8 vlan-ids=400

/interface vlan
add interface=bridge1 name=VLAN200 vlan-id=200
add interface=bridge1 name=VLAN300 vlan-id=300
add interface=bridge1 name=VLAN400 vlan-id=400

/ip address
add address=20.0.0.1/24 interface=VLAN200
add address=30.0.0.1/24 interface=VLAN300
add address=40.0.0.1/24 interface=VLAN400
```

A few notes:

* The bridge concept is an abstraction you don't see in a lot of other network gear, so it will take a few minutes to get used to it
* Make sure you are reading configuration examples for the C3XX line of products as other products are configured differently
* For access ports set the PVID (as shown and documented), otherwise they will get dynamically added to PVID 1, for trunk ports leaving off the PVID seems the proper config
* Notice how the bridge "bridge1" is being added as a tagged port; don't forget that if you want to route inter-VLAN
* Set each VLAN ID separately so you can edit them easier later
* `/interface bridge port` and `/interface bridge vlan` will be your friend
* RouterOS reports "current tagged" and "current untagged" which can be confusing, use "print detail" to see what is actually tagged and untagged
* Note "vlan-filtering=yes" during bridge creation. Many of the docs show starting with "vlan-filtering=no" while building the configuration because they don't want you to lock yourself out of the switch (if you are connected over the network) before you are done. So their examples are create the bridge, configure the switch, then finally turn vlan filtering on once you are done. But it would be easy to forget to turn it on. I changed the example above to set it to yes at the start.

Also note that if you don't have an "H" beside a port then it's not being managed in hardware and will be slow.

```
[admin@MikroTik] > /interface bridge port print 
Flags: X - disabled, I - inactive, D - dynamic, H - hw-offload 
 #     INTERFACE                                           BRIDGE                                          HW  PVID PRIORITY  PATH-COST INTERNAL-PATH-COST    HORIZON
 0   H ether1                                              bridge1                                         yes  102     0x80         10                 10       none
 1   H ether2                                              bridge1                                         yes  103     0x80         10                 10       none
 2   H ether3                                              bridge1                                         yes  104     0x80         10                 10       none
 3   H ether4                                              bridge1                                         yes    1     0x80         10                 10       none
 4   H ether5                                              bridge1                                         yes  106     0x80         10                 10       none
 SNIP!
 ```

## Conclusion

For some reason I like Mikrotik gear. I have quite a bit of it, including some wireless (don't get me started on the whole capsman thing). What I don't like about Mikrotik is that the documentation is somewhat lacking. Configuring a Juniper switch can be done easily with some surface level googling as pretty much every Juniper question has been answered succinctly and search engine indexed. Not so with Mikrotik. Their large and changing product line, part of their value proposition, makes this even more difficult.

One thing I do like about Mikrotik is that the OS comes with many features, even MPLS. This system is not just a dumb switch, it is a full featured router.

Finally, the switch is only 24 ports. Technically I'd have to have two of them to match the 48 ports of the EX4200. When you have many boxes ports will run out quickly.

You can certainly find cheaper, older used switches, but they will likely be loud and power hungry.
