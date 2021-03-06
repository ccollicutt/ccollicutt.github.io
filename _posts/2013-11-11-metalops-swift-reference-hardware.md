---
layout: post
title: MetalOps - OpenStack Swift reference hardware
categories:
---

# {{ page.title }}

I can't remember where I first heard the phrase metalops but I think it's an interesting term, much like devops and all the other ops (opops?). Technology rapidly changes, and so must definitions and labels and job titles, etc, etc. Whether these labels are correct or not, they are often useful. (As an aside, I really like the [Gartner Hype Cycle](http://www.gartner.com/technology/research/methodologies/hype-cycle.jsp).)

I think metalops is interesting because it defines an important role in "the cloud" which is that there is always hardware running beneath it, and someone needs to take care of it. Certainly one of the major features of "the cloud" is that the users don't have to worry about the underlying hardware. But cloud providers do.

My current employer has an interesting role in that while it's an advocate for the use of "the cloud" (ie. in most cases users *not* running the physical servers) we also provide several small cloud systems based on OpenStack. This means that we also have to maintain and administer the underlying hardware. Thus, while with one hand I am  using devops tools and methodologies, with the other I am trying to figure out where to get smaller hard-drive screws and whether or not serial-over-lan is going to work on the new hardware so that I don't have to load up a virtual machine and run a GUI java interface. While I can delete a semi-colon I can't remove 1.5mm of metal from 80 too long hdd screws.

But enough about that, let's talk OpenStack Swift hardware.

## Swift hardware

First let me note that we are not yet in production with this hardware. I'll come back and update this post once we are.

We bought two types of servers: 

# 4x proxy nodes
# 6x storage nodes 

We will be deploying the small cluster in two separate geographical areas. We purchased the hardware from [Silicon Mechanics](http://www.siliconmechanics.com/) who have been extremely helpful throughout the process, especially with regards to getting us parts (the parts we forgot) fast, usually in a couple of days. Their servers are based on Supermicro gear.

### Proxy Nodes

The proxy nodes are simple 1U servers that will act at the front-ends to the Swift system. Each region will have two proxy nodes, and while we haven't exactly determined how they will be used, each region will likely end up with the pairs being highly available in an active/passive setup, though it’s possible we may change our mind and have them active/active by being load balanced by a third system.

Proxy node hardware:

- Silicon Mechanics Rackform iServ R335.v3 "Proxy Nodes"   
- CPU:  1x Intel Xeon E5-2630L, 2.0GHz (6-Core, HT, 15MB Cache, 60W) 32nm
- RAM:  64GB (4 x 16GB DDR3-1600 ECC Registered 2R DIMMs) Operating at 1600 MT/s Max
- 8x 2.5” hot swap drive slots
- 2x GB on-board NICS and 1x IPMI/BMC shared LAN port
- PCIe 3.0 x16 - 2:  Intel X520-DA2 10GbE Dual-Port Server Adapter (82599ES) 10GBASE-CR - SFP+ Direct Attach

To start we only put one CPU and 64GB of RAM in the proxy nodes, but if we find out they are underpowered we can add a CPU and double the RAM quite easily. We also decided to use a chassis with eight drive slots in case we decide to re-use these servers in the future for a completely different purpose. With eight slots they could easily become OpenStack compute nodes. If they only had four drive slots they might not be as useful.

### Storage nodes

The storage nodes are interesting beasts. Each 4U box has 36x 3.5” drive slots. There are 24x slots on the front of the server and 12x in the back. This is dense storage. 

To start we only loaded each storage node with 10x 3TB drives, so we can add 26x more drives as we require more storage.

- Silicon Mechanics Swift Storage Node
- CPU: 2 x Intel Xeon E5-2630L, 2.0GHz (6-Core, HT, 15MB Cache, 60W) 32nm
- RAM: 128GB (8 x 16GB DDR3-1600 ECC Registered 2R DIMMs) 
- 2x GB on-board NICS and 1x IPMI/BMC shared LAN port
- Controller: I350 Dual-Port Ethernet, 2 Ports 6Gb/s SATA, and 8 Ports 3Gb/s SATA
- LP PCIe 3.0 x16: LSI 9207-8i (8-Port Int, PCIe 3.0)
- LP PCIe 3.0 x8 - 1: Intel X520-DA2 10GbE Dual-Port Server Adapter (82599ES) - 10GBASE-CR - SFP+ Direct Attach
- Front Drive Set: 10 x 3TB Seagate Constellation CS (6Gb/s, 7.2K RPM, 64MB Cache) 3.5-inch SATA

We went with 128GB of memory and lower wattage CPUs that still have 6 cores. We will be using 2x of the hot swap slots for the OS drive. These chassis have 2x internal hard drive slots for OS drives, but getting them out requires pulling the entire server out of the rack to get at them, so we aren't going to use them.

Silicon Mechanics also offers an SSD cache-drive option, but we aren't going to deploy Swift using cache drives, though I think some organizations do. Perhaps we will in the future. SSD caching is certainly on our list of technologies to investigate.

### MetalOps Issues

We did have some issues with these servers, though not due to the vendor whatsoever.

- We are reusing some 2.5" hard drives from another project as the OS drives for all these servers. I forgot to order 2.5" adapters for the 3.5" hot swap sleds, so we couldn't install the OS drives.
- The hdd screws that come with the 3.5" sleds in the storage nodes and the 2.5" sleds in the proxy nodes are too long for our 2.5" drives. So we had to order smaller screws.
- We didn't order c13-c14 power cables to plugin to the standard rack power. Fortunately our datacenter was able to provide them or we wouldn't have been able to power up the servers.
- The proxy rack rails are identical for the left and right rails. This means that in order to pull the server out of the rack the left rail release has to be pushed up, and the right release down. It took us a few minutes to figure this out. The storage node rails have distinct left and right rails.
