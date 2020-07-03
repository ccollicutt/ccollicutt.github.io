---
layout: post
title: Simple NSX-T Design - Dual N-VDS and Edge VM on VDS
categories:
header_image: "/img/nsx-design-1.jpg"
summary: "A simple NSX-T VLAN, virtual switch, and physical NIC layout"

---

# {{ page.title }}

There are a few different ways to "design" an NSX-T deployment. Mostly I think about what VLANs, virtual switches, and physical NICs are available and what gets assigned where.

Please note the inspiration for the diagram and design come from this [great site](https://www.ovnetworks.com/2019/12/nsx-t-edge-vm-design-single-n-vds-edge.html). I loved the layout of the diagram, so it is a huge influence for the image above (though I completely recreated and adapted it as a Power Point slide of all things).

## Design: Dual N-vDS, Edge VM on VDS, Four Physical Nics, Default NSX-T Profiles, and TEP on Access Ports

That's quite a mouthful, but it is what it is. This particular design consists of the following:

* Dual N-vDS: one for the overlay and one for the uplink
* Four physical network interfaces: two for VDS and two for NSX-T
* The NSX-T Edge VM is on vSphere VDS provided interfaces (thus VLAN 0)
* Using the default NSX-T profiles (both using VLAN 0)
* The Transport End Point (TEP) VLAN is on access ports for the physical NICs that make up the NSX-T managed interfaces

I believe in NSX-T 2.5 you can get this down to having a single N-vDS, but then would need to put the Edge VM TEP on a separate VLAN and have traffic routable between the two, i.e. the Edge VM's TEP interface would not be on a VDS port group, as well as some additional advantages. But I'll try to explore that in another post. For now, I like this design because it is quite simple, and is a pretty good design for a PoC of [Tanzu Kubernetes Grid Integrated Edition](https://tanzu.vmware.com/kubernetes-grid) (TKGI, formerly known as PKS).

Ultimately it's pretty easy to create a new NSX-T profile for the ESXi hosts, a few clicks, a few seconds, and that is what most people do as the ESXi host's TEP VLAN is often a trunk port. But in this design I'm just using an access port, so the default profile with VLAN 0 works.

# Conclusion

I haven't created this design for production use. This is what I think is the simplest design for a proof of concept (where NSX-T isn't the PoC focus). I would imagine a production design would look much different. Also this design works great in a nested lab.

Don't forget to set your TEP VLAN MTU >=1600. Most people set it to 9k and are done with it. :)


