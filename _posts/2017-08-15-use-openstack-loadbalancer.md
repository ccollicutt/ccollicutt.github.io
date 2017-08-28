---
layout: post
title: Create an OpenStack Load Balancer
categories:
header_image: /img/balance.jpg
header_permalink: https://unsplash.com/search/photos/balance?photo=FO7bKvgETgQ
---

# {{ page.title }}

In a recent post I discussed the [OpenStack Octavia](http://serverascode.com/2017/08/11/install-openstack-octavia-loadbalancer.html) project which provides a backed to Neutron's LBaaS system.

In this post I'll just go over a quick example of creating and using a load balancer and using it in OpenStack.

The [official docs](https://docs.openstack.org/mitaka/networking-guide/config-lbaas.html) are pretty good on this, but it's worthwhile to cover them in a real environment.

## Create Two Webservers

I've already created two web servers, web-1 and web-2.

```
$ os server list | grep web-
| 05f0bb14-2895-4c7b-a493-ef2a1b57c721 | web-2 | ACTIVE  | vxlan1=10.50.0.15 | xenial     |
| 3ff621cd-46cb-4fcf-8d9d-2193718a11f5 | web-1 | ACTIVE  | vxlan1=10.50.0.35 | xenial     |
```
<br />

I've also changed each of the /var/www/html/index.html pages to have only the hostname in it, so that if you connect to port 80 on either of them they respond with their hostname.

## Create a Load Balancer

First, we create a load balancer and put it onto a specific network.

```
$ neutron lbaas-loadbalancer-create --name web-lb vxlan1-subnet
Created a new loadbalancer:
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| admin_state_up      | True                                 |
| description         |                                      |
| id                  | 6b9c75c1-4af2-424d-8b20-681926de4e0d |
| listeners           |                                      |
| name                | web-lb                               |
| operating_status    | OFFLINE                              |
| pools               |                                      |
| provider            | octavia                              |
| provisioning_status | PENDING_CREATE                       |
| tenant_id           | 3b29434130cd487186f1da0b5831232f     |
| vip_address         | 10.50.0.11                           |
| vip_port_id         | b88dd055-e963-4227-86e1-558df52dc946 |
| vip_subnet_id       | 5ce133ce-cce6-4142-89d4-a71da87bbde6 |
+---------------------+--------------------------------------+
```

<br />

Show the newly created loadbalancer. Note the "octavia" provider, and the IP the load balancer received.

```
$ neutron lbaas-loadbalancer-show web-lb
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| admin_state_up      | True                                 |
| description         |                                      |
| id                  | 6b9c75c1-4af2-424d-8b20-681926de4e0d |
| listeners           |                                      |
| name                | web-lb                               |
| operating_status    | ONLINE                               |
| pools               |                                      |
| provider            | octavia                              |
| provisioning_status | ACTIVE                               |
| tenant_id           | 3b29434130cd487186f1da0b5831232f     |
| vip_address         | 10.50.0.11                           |
| vip_port_id         | b88dd055-e963-4227-86e1-558df52dc946 |
| vip_subnet_id       | 5ce133ce-cce6-4142-89d4-a71da87bbde6 |
+---------------------+--------------------------------------+
```

<br />

There is also an existing security group called web that allows port 80.

```
$ os security group list | grep web
| db3f7e2d-2453-416d-8651-ba8544502d0f | web           | web                    | 3b29434130cd487186f1da0b5831232f |
```

<br />

The security group must be added to the LB VIP port which was shown above in the LB show.

```
$ neutron port-update --security-group web b88dd055-e963-4227-86e1-558df52dc946
Updated port: b88dd055-e963-4227-86e1-558df52dc946
```

<br />

Now to create a listener.

```
$ neutron lbaas-listener-create --name web-lb-http --loadbalancer web-lb --protocol HTTP --protocol-port 80
Created a new listener:
+---------------------------+------------------------------------------------+
| Field                     | Value                                          |
+---------------------------+------------------------------------------------+
| admin_state_up            | True                                           |
| connection_limit          | -1                                             |
| default_pool_id           |                                                |
| default_tls_container_ref |                                                |
| description               |                                                |
| id                        | 364c08e7-29a6-4c02-b6e6-7b0d18f3e10e           |
| loadbalancers             | {"id": "6b9c75c1-4af2-424d-8b20-681926de4e0d"} |
| name                      | web-lb-http                                    |
| protocol                  | HTTP                                           |
| protocol_port             | 80                                             |
| sni_container_refs        |                                                |
| tenant_id                 | 3b29434130cd487186f1da0b5831232f               |
+---------------------------+------------------------------------------------+
```

<br />

Create a pool.
```
$ neutron lbaas-pool-create \
>   --name web-lb-pool-http \
>   --lb-algorithm ROUND_ROBIN \
>   --listener web-lb-http \
>   --protocol HTTP

Created a new pool:
+---------------------+------------------------------------------------+
| Field               | Value                                          |
+---------------------+------------------------------------------------+
| admin_state_up      | True                                           |
| description         |                                                |
| healthmonitor_id    |                                                |
| id                  | 19e86ff3-aa58-4582-8399-4ad3c9eaeb9d           |
| lb_algorithm        | ROUND_ROBIN                                    |
| listeners           | {"id": "364c08e7-29a6-4c02-b6e6-7b0d18f3e10e"} |
| loadbalancers       | {"id": "6b9c75c1-4af2-424d-8b20-681926de4e0d"} |
| members             |                                                |
| name                | web-lb-pool-http                               |
| protocol            | HTTP                                           |
| session_persistence |                                                |
| tenant_id           | 3b29434130cd487186f1da0b5831232f               |
+---------------------+------------------------------------------------+
```

<br />

Add the first member.

```
$ neutron lbaas-member-create \
>     --subnet vxlan1-subnet \
>     --address 10.50.0.35 \
>     --protocol-port 80 \
>     web-lb-pool-http
Created a new member:
+----------------+--------------------------------------+
| Field          | Value                                |
+----------------+--------------------------------------+
| address        | 10.50.0.35                           |
| admin_state_up | True                                 |
| id             | 38fb7a37-24f4-4fc5-a50b-4e21502d239e |
| name           |                                      |
| protocol_port  | 80                                   |
| subnet_id      | 5ce133ce-cce6-4142-89d4-a71da87bbde6 |
| tenant_id      | 3b29434130cd487186f1da0b5831232f     |
| weight         | 1                                    |
+----------------+--------------------------------------+
```

<br />

Add the second member.

```
$ neutron lbaas-member-create \
>     --subnet vxlan1-subnet \
>     --address 10.50.0.15 \
>     --protocol-port 80 \
>     web-lb-pool-http
Created a new member:
+----------------+--------------------------------------+
| Field          | Value                                |
+----------------+--------------------------------------+
| address        | 10.50.0.15                           |
| admin_state_up | True                                 |
| id             | 02074bde-90b2-4c0d-bfb1-2bb2e215d7fb |
| name           |                                      |
| protocol_port  | 80                                   |
| subnet_id      | 5ce133ce-cce6-4142-89d4-a71da87bbde6 |
| tenant_id      | 3b29434130cd487186f1da0b5831232f     |
| weight         | 1                                    |
+----------------+--------------------------------------+
```

<br />

## Assign a Floating IP to the Load Balancer

A floating IP has already been created.

```
$ neutron floatingip-list
+--------------------------------------+------------------+---------------------+---------+
| id                                   | fixed_ip_address | floating_ip_address | port_id |
+--------------------------------------+------------------+---------------------+---------+
| 9e048a99-8533-4dc6-9e6f-1347c09d28e9 |                  | <floating ip>       |         |
+--------------------------------------+------------------+---------------------+---------+
```

<br />

Actually associate the floating IP. In this case we are associating a floating IP with a port instead of a "server."

```
neutron floatingip-associate 9e048a99-8533-4dc6-9e6f-1347c09d28e9 b88dd055-e963-4227-86e1-558df52dc946
Associated floating IP 9e048a99-8533-4dc6-9e6f-1347c09d28e9
```

<br />

## Testing a Load balancer

Now that the floating IP is up and attached to a working load balancer, and behind it is a couple of working web servers...

```
$ while true; do curl <floating ip>; sleep 1; done
web-2
web-1
web-2
web-1
web-2
web-1
web-2
web-1
^
```

<br />
