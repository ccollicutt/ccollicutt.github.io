---
layout: post
title: Install the Ocatvia Loadbalancing System into an OpenStack Cloud
categories:
header_image: /img/road.jpg
header_permalink: https://unsplash.com/search/lines?photo=_v6mhdOK2g0
---

# {{ page.title }}

This post discusses OpenStack as Infrastructure as a Service (IaaS) and--although not completely--how to deploy the Octavia load balancing system into an OpenStack cloud. To be forthcoming, I don't think you could get Octavia running just by reading this post, and it wasn't necessarily the reason for it, but there might be some helpful notes and links somewhere in this text.

## The Usefulness of Load Balancers in IaaS

Most public clouds have a load balancing service. In Amazon Web Services (AWS) it's the Elastic Load Balancer service. In Google's Cloud, it's the Google Cloud Load Balancer. Azure also has a load balancer. Fortunatley so does OpenStack.

Technically a load balancer service is not necessary to run an OpenStack cloud, but for all intents and purposes in order to deploy applications on top of OpenStack in a "cloud native" fashion, load balancers are a practical requirement.

## OpenStack and Load Balancer as a Service

People who run OpenStack clouds have some choices they can make in terms of how the LBaaS operates within their cloud. One of the newer methods for providing LBaaS in an OpenStack cloud is the [Octavia](https://docs.openstack.org/octavia/latest/reference/introduction.html) project.

**Octavia**

There are a couple of backends that can be used to support LBaaS within an OpenStack cloud. One of them is Octavia, and that is what will be discussed in this post.

>Octavia accomplishes its delivery of load balancing services by managing a fleet of virtual machines, containers, or bare metal servers—collectively known as amphorae—which it spins up on demand. This on-demand, horizontal scaling feature differentiates Octavia from other load balancing solutions, thereby making Octavia truly suited “for the cloud.”

**Service Tenant / Service VM Model**

One of the things that I like most about Octavia is the concept of the "service tenant" or "service virtual machine." It's important that the systems underpinning an OpenStack cloud be scalable. The way that Octavia approaches this is to create a specific virtual machine (or machines) for each load balancer that is created, and this VM is only used by a specific tenant. Essentially, hidden behind the scenes is a service tenant that is used to create virtual machines for end-user services.

For example, if LBaaS + Octavia has already been deployed and we create a load balancer, we can see the virtual machine running in the Octavia service tenant. (This VM is not visible or accessible by the actual end user, other than as a load balancer endpoint.)

```
# NOTE: As the octavia service tenant user...
$ openstack server list
+--------------------------------------+----------------------------------------------+--------+--------------------------------------------------+------------+
| ID                                   | Name                                         | Status | Networks                                         | Image Name |
+--------------------------------------+----------------------------------------------+--------+--------------------------------------------------+------------+
| 9135a037-d67d-450e-b809-5a28a26b8b74 | amphora-3521b6fa-1824-448d-85d0-5f0cf1e87f67 | ACTIVE | test-vxlan=10.50.0.30; octavia-mgmt=172.16.31.20 |            |
+--------------------------------------+----------------------------------------------+--------+--------------------------------------------------+------------+
```

<br />

## Environment

The environment I'm working in is an OpenStack deployment based on Ubuntu Xenial/16.04 and the Ubuntu cloud archive packages (all though Octavia is not yet packaged and Octavia is installed from pip), and the version of OpenStack deployed is Ocata.

## Octavia and LBaaS  Documentation

At the time I'm writing this blog post, the documentation for Octavia is not that detailed. In order to deploy Neutron LBaaS one would probably have to read the code for some deployment tools such as the devstack plugin for Octavia or the OpenStack Ansible projects role for Octavia. I would suggest that the Ansible role provides the most detailed information, especially their configuration file template.

* [Neutron documentation](https://docs.openstack.org/ocata/networking-guide/config-LBaaS.html)
* [Octavia Documentation](https://docs.openstack.org/octavia/latest/)
* [Devstack Octavia Plugin Script](https://github.com/openstack/octavia/blob/master/devstack/plugin.sh)
* [OpenStack Ansible Octavia Role](https://docs.openstack.org/openstack-ansible-os_octavia/latest/configure-octavia.html)

## Pre-deployment Steps

To use Octavia and Neutron a few things have to exist. Of course these could (and should) be provisioned through your automation system.

The following needs to be done:

1. Create Octavia Neutron management network
1. Create Octavia service user
2. Create an Amphora image
1. Upload the Amphora image into Glance
3. Create Octavia security group
4. Create certificates
5. (Optional) Create SSH keys for admin level troubleshooting Amphora images

Once these tasks have been completed, Octavia and Neutron LBaaS can be deployed.

## Neutron Configuration

Basically the process is to configure Neutron to provide the load balancing API, and then setup Octavia to be the back end for that API.

There are three files that I have altered from a standard deployment. I believe some of them could be converged, but I kind of like the separation.

* neutron.conf
* neutron_lbaas.conf
* services_lbaas.conf

I've also altered `/etc/default/neutron-server` to start `neutron-server` with the additional config files.

```
neutron-api:/etc/neutron# cat /etc/default/neutron-server
# defaults for neutron-server

# path to config file corresponding to the core_plugin specified in
# neutron.conf
NEUTRON_PLUGIN_CONFIG="/etc/neutron/plugins/ml2/ml2_conf.ini"

DAEMON_ARGS="$DAEMON_ARGS --config-file=/etc/neutron/neutron_lbaas.conf --config-file=/etc/neutron/services_lbaas.conf"
neutron-api:/etc/neutron# ps ax | grep neutron-server | head -1
  538 ?        Ss     1:16 /usr/bin/python /usr/bin/neutron-server --config-file=/etc/neutron/neutron.conf --config-file=/etc/neutron/plugins/ml2/ml2_conf.ini --config-file=/etc/neutron/neutron_lbaas.conf --config-file=/etc/neutron/services_lbaas.conf --log-file=/var/log/neutron/neutron-server.log
```

<br />

**neutron.conf**

The first is to add the LBaaS service plugin to `neutron.conf`.

```
service_plugins = router,neutron_lbaas.services.loadbalancer.plugin.LoadBalancerPluginv2
```

<br />

**neutron_lbaas.conf**

Also there is the addition of the `neutron_lbaas.conf` configuration file.

```
[service_providers]
service_provider = service_provider = LOADBALANCERV2:Octavia:neutron_lbaas.drivers.octavia.driver.OctaviaDriver:default


[service_auth]
auth_url = http://<internal keystone endpoint>:5000/v3
admin_user = octavia
admin_tenant_name = service
admin_password = <octavia password>
admin_user_domain = default
admin_project_domain = default
region = tor1
auth_version = 3
endpoint_type = internalURL

# NOTE(curtis): not sure of service name???
#service_name = LBaaS

# Disable server certificate verification (boolean value)
#insecure = false
```

<br />

**services_lbaas.conf**

```
neutron-api:/etc/neutron# grep -v "^#\|^$" services_lbaas.conf
[DEFAULT]
[haproxy]
[octavia]
base_url = http://<Octavia API or internal VIP>:9876
[radwarev2]
[radwarev2_debug]
```

<br />

**Update the Neutron Database**

Once Neutron has been configured to provide the LBaaS API, the database needs some "migrations."

```
neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head
```

<br />

**Ensure neutron-lbaasv2-agent is Stopped and Disabled**

It's not used with Octavia and must not be running.

## Setup Octavia

Besides Neutron changes, Octavia also needs to be installed and configured.

Ubuntu does not have packages for Octavia yet, so Octavia code will be installed via Pip. I'm using 0.10.0. (One can find all the release versions for OpenStack projects [here](https://releases.openstack.org/teams/octavia.html).)

```
# pip install ocatavia==0.10.0
```

Pip installs Octavia services in `/usr/local/bin/`:

```
# ls -1 /usr/local/bin/octavia-*
/usr/local/bin/octavia-api
/usr/local/bin/octavia-db-manage
/usr/local/bin/octavia-health-manager
/usr/local/bin/octavia-housekeeping
/usr/local/bin/octavia-worker
```

Running those services would require some sort of init mechanism, such as systemd but that's beyond the scope of this blog post. Suffice it to say, octavia-api, octavia-health-manager, octavia-housekeeping, and octavia-worker need to be running somewhere.

**Configure octavia.conf**

The octavia conifguration file is fairly complex. One of the best examples is the [OpenStack Ansible Octavia Role's octavia.conf template](https://git.openstack.org/cgit/openstack/openstack-ansible-os_octavia/tree/templates/octavia.conf.j2). Unfortunately it's beyond the scope of this blog post to completely detail the Octavia config file, and it will take some consideration to get correct for your particular environment.

As another example, I deployed a devstack instance based on the Pike release and put up the resulting octavia.conf in a [Github gist](https://gist.github.com/ccollicutt/3f9f5d2cf0d34b55034aef2241106071).

**Create Certificates**

In the [Octavia git repository](git clone https://github.com/openstack/octavia.git
) there is a `create_certificates.sh` script which can be used to generate (example, non-prod) certificates. Production deployments would probably require some consideration in terms of certificate management.

## Conclusion

Hopefully this blog post has provided some useful information, and can get you on your way towards implementing Octavia and OpenStack LBaaS.

Please feel free to email me (curtis at serverascode.com ) or perhaps add a comment to the post if you have any questions, notice any mistakes, or have improvements that could be made.

Thanks for reading.
