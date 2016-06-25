---
layout: post
title: Split OpenStack Keystone Catalog
categories:
header_image: /img/split-keystone.jpg
---

# {{ page.title }}

OpenStack Keystone is aptly named--it's a service almost everyone who deploys OpenStack need to run. From a high level it provides authentication.

>Keystone is an OpenStack project that provides Identity, Token, Catalog and Policy services for use specifically by projects in the OpenStack family. It implements OpenStack’s Identity API. -- [Keystone documentation](http://docs.openstack.org/developer/keystone/)

However, as the above quote mentions it also provides a catalog of OpenStack service endpoints, where endpoints essentially means where you make REST API calls to.

## Listing the catalog

When Keystone is deployed, you can use the OpenStack command line client to list the endpoints.

Below I just show the endpoints for the compute service. Note that the endpoint starts with https, and thus these endpoints are TLS enabled. (Note that I alias _openstack_ to _os_.)

~~~bash
ubuntu@uc-external-osclient-01:~$ os catalog show compute
+-----------+-----------------------------------------------------------------------+
| Field     | Value                                                                 |
+-----------+-----------------------------------------------------------------------+
| endpoints | yeg-uc-1                                                              |
|           |   admin: https://uc-                                                  |
|           | api.lab.example.com:8774/v2/9a6a0815d6e74146bb76f19fd580bc31     |
|           | yeg-uc-1                                                              |
|           |   public: https://uc-                                                 |
|           | api.lab.example.com:8774/v2/9a6a0815d6e74146bb76f19fd580bc31     |
|           | yeg-uc-1                                                              |
|           |   internal: https://uc-                                               |
|           | api.lab.example.com:8774/v2/9a6a0815d6e74146bb76f19fd580bc31     |
|           |                                                                       |
| name      | Compute Service                                                       |
| type      | compute                                                               |
+-----------+-----------------------------------------------------------------------+
~~~

But if I run the same command from an internal client we see slightly different endpoints.

~~~bash
ubuntu@uc-internal-osclient-01:~$ os catalog show compute
+-----------+-----------------------------------------------------------------------------------------+
| Field     | Value                                                                                   |
+-----------+-----------------------------------------------------------------------------------------+
| endpoints | yeg-uc-1                                                                                |
|           |   admin: http://uc-api.lab.example.com:8774/v2/9a6a0815d6e74146bb76f19fd580bc31    |
|           | yeg-uc-1                                                                                |
|           |   public: http://uc-api.lab.example.com:8774/v2/9a6a0815d6e74146bb76f19fd580bc31   |
|           | yeg-uc-1                                                                                |
|           |   internal: http://uc-api.lab.example.com:8774/v2/9a6a0815d6e74146bb76f19fd580bc31 |
|           |                                                                                         |
| name      | Compute Service                                                                         |
| type      | compute                                                                                 |
+-----------+-----------------------------------------------------------------------------------------+
~~~

Note that it is _http_ not _https_.

## Split Catalog

In this example there are actually two Keystone servers running, and they are identical except for their default_catalog.template file. In fact that file is identical except that the internal Keystone server will use _http_ as the protocol, and the external Keystone server provides a catalog of endpoints with TLS enabled.

Why would we do this?

Well, in some cases we want internal OpenStack services to simply use http/plaintext, perhaps for performance reasons, but obviously external access, even if it's only accessible from a companies internal network, should be TLS enabled. Separating the Keystone catalog using two Keystone servers and perhaps a load balancer makes this split Keystone setup possible.

In short, external is encrypted and internal is not.

## default_catalog.template

There are two major ways to define the Keystone catalog: 1) via the database and 2) default_catalog.template.

The template file is just that--a configuration file. Using this configuration file, different Keystone servers can return different catalogs. This is, in my opinion, the easiest way to manage the catalog, and what's more, is much easier to maintain using configuration management tools. (To all developers: don't put configuration information into the database if you can avoid it. LXC 2.0 I'm looking at you.)

## Split DNS

In this particular example internal and external access uses the same Keystone endpoint host name: _uc-api.lab.example.com_ but the internal systems have a different IP address associated with that hostname than the external servers do. 

The load balancer in front of the Keystone servers directs traffic to the correct Keystone backend based on the IP address that is being accessed by the client. So external systems access the external Keystone server and vice-versa for internal. 

## Configuration management

Typically I use Ansible to manage configurations. The Keystone servers are separated into two groups: internal and external. 

The external servers have their protocol set to https.

~~~bash
[ansible-openstack]$ cat group_vars/external-keystone-api 
---
keystone_endpoint_protocol: "https"
~~~

By default the protocol in the default_catalog.template file is http.

{% raw %}
~~~bash
[ansible-openstack]$ head roles/keystone/templates/default_catalog.templates 
# keystone
catalog.{{ region }}.identity.publicURL = {{ keystone_endpoint_protocol | default('http') }}://{{ api_fqdn }}:$(public_port)s/v2.0
catalog.{{ region }}.identity.adminURL = {{ keystone_endpoint_protocol | default('http') }}://{{ api_fqdn }}:$(admin_port)s/v2.0
catalog.{{ region }}.identity.internalURL = {{ keystone_endpoint_protocol | default('http') }}://{{ api_fqdn }}:$(public_port)s/v2.0
catalog.{{ region }}.identity.name = Identity Service

# nova
catalog.{{ region }}.compute.publicURL = {{ keystone_endpoint_protocol | default('http') }}://{{ api_fqdn }}:8774/v2/$(tenant_id)s
catalog.{{ region }}.compute.adminURL = {{ keystone_endpoint_protocol | default('http') }}://{{ api_fqdn }}:8774/v2/$(tenant_id)s
catalog.{{ region }}.compute.internalURL = {{ keystone_endpoint_protocol | default('http') }}://{{ api_fqdn }}:8774/v2/$(tenant_id)s
~~~
{% endraw %}

Certainly there are many other things that the configuration management tool will have to do, but the basics are shown above. There has to be some way to set the external Keystone template to use https and http for internal. This is just one way to accomplish that.

## Issues with this model

Using this model we are classifying traffic into internal and external. Further, we are suggesting that internal traffic for the OpenStack services does not require encryption. This is a big decision. Certainly it's a common model, but it creates a "chewy center" so to speak, meaning that external access is a hard shell, but that once you get through that hard shell, everything within is easier to crack. Sometimes people thing that "defense in depth" means that it's okay for each ring to be less secure, but that is not necesarily the goal.

Another way to secure the API endpoints is to assume that all traffic, regardless of origin, may be malicious. With that assumption splitting Keystone would not be desirable, and instead it would be worthwhile to secure Keystone as much as possible regardless of the origin of the traffic, and in fact there would only be one API endpoint used by external end users and internal OpenStack services. Something to consider.

That said, it's sometimes difficult to get internal OpenStack services to use https, or at least it has been previously.
