---
layout: post
title: Glance with Multiple Backend Stores
categories:
header_image: /img/glitch-mountain.jpg
header_permalink: https://flic.kr/p/pAh5DE
---

# {{ page.title }}

Every OpenStack cloud needs a Glance deployment. Thus, I've deployed it many times--but every time I've only put one backend into use: either simple file based storage or object storage using Swift. 

Recently I was surprised to learn that you can have multiple backends, or stores as Glance calls it, and can upload images into whichever store you prefer using an option on the Glance command line.

## Devstack 

If you want to test it out it's very easy--just deploy a [DevStack](http://docs.openstack.org/developer/devstack/) instance. All you need is vm with a few gigs of ram.

Here is the local.conf I used. It enables Swift.

~~~bash
ubuntu@devstack:~/devstack$ cat local.conf 
[[local|localrc]]
# Credentials
ADMIN_PASSWORD=password
DATABASE_PASSWORD=password
RABBIT_PASSWORD=password
SERVICE_PASSWORD=password
SERVICE_TOKEN=password
SWIFT_HASH=password
SWIFT_TEMPURL_KEY=password

# Enable Neutron
disable_service n-net
disable_service n-novnc
enable_service q-svc
enable_service q-agt
enable_service q-dhcp
enable_service q-l3
enable_service q-meta
enable_service neutron

# Enable Swift
enable_service s-proxy
enable_service s-object
enable_service s-container
enable_service s-account

# Disable Horizon
disable_service horizon

# Disable Heat
disable_service heat h-api h-api-cfn h-api-cw h-eng

# Disable Cinder
disable_service cinder c-sch c-api c-vol

# Swift temp URL's are required for agent_* drivers.
SWIFT_ENABLE_TEMPURLS=True

# By default, DevStack creates a 10.0.0.0/24 network for instances.
# If this overlaps with the hosts network, you may adjust with the
# following.
NETWORK_GATEWAY=10.1.0.1
FIXED_RANGE=10.1.0.0/24
FIXED_NETWORK_SIZE=256

# Log all output to files
LOGFILE=$HOME/devstack.log
LOGDIR=$HOME/logs
~~~

Ah, good old DevStack.

## Glance config

Once DevStack is done, the _/etc/glance/glance-api.conf_ file will contain this section:

~~~bash
[glance_store]
stores = file, http, swift
default_swift_reference = ref1
swift_store_config_file = /etc/glance/glance-swift-store.conf
swift_store_create_container_on_put = True
default_store = swift
filesystem_store_datadir = /opt/stack/data/glance/images/
~~~

Note the "stores" option, and that "file, http, swift" are configured.

## Glance API V1

One caveat at this time is that the Glance V2 API does not seem to allow for setting an option when uploading an image as to which backend to use. Thus, with Glance V2 you will always be using the default_store, which in this example is Swift.

But, if you use V1, you can set the "--store" option and choose a backend.

~~~bash
ubuntu@uc-osclient-01:~$ glance --os-image-api 2 help image-create | grep "\-\-store" 
# No option
ubuntu@uc-osclient-01:~$ glance --os-image-api 1 help image-create | grep "\-\-store" 
usage: glance image-create [--id <IMAGE_ID>] [--name <NAME>] [--store <STORE>]
  --store <STORE>       Store to upload image to.
~~~

Perhaps in the future V2 will also have a backend store option.

## Uploading images to multiple backend stores

Ok, lets use the Glance CLI to upload images to the file and Swift backends.

First, I downloaded the Cirros image.

~~~bash
ubuntu@devstack:~$ wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
--2016-07-09 03:04:11--  http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
Resolving download.cirros-cloud.net (download.cirros-cloud.net)... 64.90.42.85
Connecting to download.cirros-cloud.net (download.cirros-cloud.net)|64.90.42.85|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 13287936 (13M) [text/plain]
Saving to: ‘cirros-0.3.4-x86_64-disk.img’

100%[==================================================================================================================================>] 13,287,936  3.61MB/s   in 4.2s   

2016-07-09 03:04:15 (3.04 MB/s) - ‘cirros-0.3.4-x86_64-disk.img’ saved [13287936/13287936]
~~~

Cubswin:).

As a note, I'm using the Glance CLI version 2.1.0.

~~~bash
ubuntu@devstack:~$ glance --version
2.1.0
~~~

Next, upload an image to the Swift backend.

~~~bash
ubuntu@devstack:~$ glance --os-image-api-version 1 image-create --store swift --name swift-cirros --disk-format qcow2 --container-format bare --is-public True --file cirros-0.3.4-x86_64-disk.img 
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | ee1eca47dc88f4879d8a229cc70a07c6     |
| container_format | bare                                 |
| created_at       | 2016-07-09T03:07:09.000000           |
| deleted          | False                                |
| deleted_at       | None                                 |
| disk_format      | qcow2                                |
| id               | a8dc5259-9579-4cff-be0a-f175278f60f3 |
| is_public        | True                                 |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | swift-cirros                         |
| owner            | fc42df31d6c84c04b55cf116e06c2b38     |
| protected        | False                                |
| size             | 13287936                             |
| status           | active                               |
| updated_at       | 2016-07-09T03:07:11.000000           |
| virtual_size     | None                                 |
+------------------+--------------------------------------+
~~~

Now upload another image, this time using the file backend.

~~~bash
ubuntu@devstack:~$ glance --os-image-api-version 1 image-create --store file --name file-cirros --disk-format qcow2 --container-format bare --is-public True --file cirros-0.3.4-x86_64-disk.img 
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | ee1eca47dc88f4879d8a229cc70a07c6     |
| container_format | bare                                 |
| created_at       | 2016-07-09T03:07:55.000000           |
| deleted          | False                                |
| deleted_at       | None                                 |
| disk_format      | qcow2                                |
| id               | e361ad97-6336-45d9-9c6a-e461ed91126d |
| is_public        | True                                 |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | file-cirros                          |
| owner            | fc42df31d6c84c04b55cf116e06c2b38     |
| protected        | False                                |
| size             | 13287936                             |
| status           | active                               |
| updated_at       | 2016-07-09T03:07:55.000000           |
| virtual_size     | None                                 |
+------------------+--------------------------------------+
~~~

We can peek into the Glance database to see where the image_location is.

~~~bash
mysql> select value from image_locations where image_id = "e361ad97-6336-45d9-9c6a-e461ed91126d";
+---------------------------------------------------------------------------+
| value                                                                     |
+---------------------------------------------------------------------------+
| file:///opt/stack/data/glance/images/e361ad97-6336-45d9-9c6a-e461ed91126d |
+---------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> select value from image_locations where image_id = "a8dc5259-9579-4cff-be0a-f175278f60f3";
+-----------------------------------------------------------------+
| value                                                           |
+-----------------------------------------------------------------+
| swift+config://ref1/glance/a8dc5259-9579-4cff-be0a-f175278f60f3 |
+-----------------------------------------------------------------+
1 row in set (0.01 sec)
~~~

As can be seen above, the swift-cirros image is stored in the Swift backend, and the file-cirros image is stored in the file based backend. I was quite surprised to learn this was possible, and even more surprised to see that the option does not seem to be in the Glance V2 API...and even more surprised that the Glance CLI has different options based on the API used.

There's always something to learn about OpenStack. :)
