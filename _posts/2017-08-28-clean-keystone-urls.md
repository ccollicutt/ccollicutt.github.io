---
layout: post
title: Clean Keystone Catalog URLs
categories:
header_image: /img/lock.jpg
header_permalink: https://unsplash.com/search/photos/lock?photo=FqaybX9ZiOU
---

# {{ page.title }}

I think the way OpenStack Keystone is deployed demands more consideration than people tend to give it. There are several different architectural models for Keystone and it would be well worth an organizations investment to take some time and think about the possibilities.

I find that OpenStack distros (not that there are that many) limit the possibilities of Keystone models. I am not a huge fan of distros, but I can understand why organizations use them. All I mean is that distros tend to limit architectural possibilities. OpenStack is so flexible that it seems a shame to limit it. But enough ranting...

## Split Keystone

I tend to use a split keystone, one internal and one external. I've [written about it before](http://serverascode.com/2016/06/24/split-keystone-catalog.html). I think this model can lead to a soft-center security issue, but its flexibility in terms of what catalog is presented to the end user makes up for it, IMHO.

## Clean URLs

Because I have an external Keystone I can provide a different catalog to external end users.

```
$ os catalog list
+--------------------+-----------+------------------------------------------------------------------------------------------+
| Name               | Type      | Endpoints                                                                                |
+--------------------+-----------+------------------------------------------------------------------------------------------+
| Image Service      | image     | someregion                                                                                     |
|                    |           |   admin: https://api.somecloud.com/image/                                           |
|                    |           | someregion                                                                                     |
|                    |           |   public: https://api.somecloud.com/image/                                          |
|                    |           | someregion                                                                                     |
|                    |           |   internal: https://api.somecloud.com/image/                                        |
|                    |           |                                                                                          |
| Compute Service    | compute   | someregion                                                                                     |
|                    |           |   admin: https://api.somecloud.com/compute/v2.1/7b61f0aece1b4aa896020f51fd724b1f    |
|                    |           | someregion                                                                                     |
|                    |           |   public: https://api.somecloud.com/compute/v2.1/7b61f0aece1b4aa896020f51fd724b1f   |
|                    |           | someregion                                                                                     |
|                    |           |   internal: https://api.somecloud.com/compute/v2.1/7b61f0aece1b4aa896020f51fd724b1f |
|                    |           |                                                                                          |
| Network Service    | network   | someregion                                                                                     |
|                    |           |   admin: https://api.somecloud.com/network/                                         |
|                    |           | someregion                                                                                     |
|                    |           |   public: https://api.somecloud.com/network/                                        |
|                    |           | someregion                                                                                     |
|                    |           |   internal: https://api.somecloud.com/network/                                      |
|                    |           |                                                                                          |
| Identity Service   | identity  | someregion                                                                                     |
|                    |           |   admin: https://api.somecloud.com/identity/v3                                      |
|                    |           | someregion                                                                                     |
|                    |           |   public: https://api.somecloud.com/identity/v3                                     |
|                    |           | someregion                                                                                     |
|                    |           |   internal: https://api.somecloud.com/identity/v3                                   |
|                    |           |                                                                                          |
| Compute Service V3 | computev3 | someregion                                                                                     |
|                    |           |   admin: https://api.somecloud.com/computev3/v3                                     |
|                    |           | someregion                                                                                     |
|                    |           |   public: https://api.somecloud.com/computev3/v3                                    |
|                    |           | someregion                                                                                     |
|                    |           |   internal: https://api.somecloud.com/computev3/v3                                  |
|                    |           |                                                                                          |
+--------------------+-----------+------------------------------------------------------------------------------------------+
```

As can be seen above, there are no ports other than 443/https being used, nor any api-someserver.somecloud.com hostnames either. Everything is on the same hostname and then each service is identified using the `/servicetype/` model. I think it makes for cleaner URLs.

## How is this accomplished?

There are a few things that have to be altered to accomplish this:

1. Use an external Keystone that reports a different catalog
2. Setup keystone.conf to set the `admin_endpoint` and `public_endpoint` to the external identiy endpoint, but only on the external Keystone nodes
3. Disable **catalog** caching (not all caching just catalog) if both external and internal Keystones are using the same memcache
4. Configure your loadbalancer to recognize `/servicetype/` and forward to a proper backend
4. Configure your loadbalancer to strip the `/servicetype/` when forwarding to the backend service

## Haproxy Example

Forwarding is based on `path_beg`. Below Glance is used as an example.

```
acl frontend-external-glance path_beg /image/
use_backend backend-external-glance if frontend-external-glance
```

Here's an example of haproxy config to strip the `/servicetype/` in the backend definition:

```
reqrep ^([^\ ]*\ /)image[/]?(.*)     \1\2
```

The above would have to be setup for each service in the external catalog.

## Conclusion

I've been testing this for a while and can't find any issues (yet). There may be better ways to do this that I'm not aware of, and if so please let me know.

I'm very happy with these clean URLs and I feel like it makes OpenStack look a bit less complicated to those who might be writing code using these APIs.
