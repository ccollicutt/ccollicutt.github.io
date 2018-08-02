---
layout: post
title: Using OpenShift's Docker Remote Registry
categories:
header_image: /img/dock.jpg
header_permalink: https://unsplash.com/photos/ZofeoSz4F3k
---

# {{ page.title }}

In this post I will cover using an OpenShift Origin deployment that has a registry as a remote registry for Docker images. Note that I'm not covering installing OpenShift--it's already deployed. I'm just going to use its registry.

## Official Documentation

This [doc](https://docs.openshift.com/container-platform/3.9/install_config/registry/accessing_registry.html) is pretty good; won't get you all the way, but it's close.

## About the Deployment

This OpenShift installation is running in AWS (not that it really matters where it's running) and the registry is backed by S3. As far as I'm concerned object storage is the best backend for Docker images. Well, at least it's effectively infinite.

The version of OpenShift Origin being used is 3.9.

## S3 Permissions

OpenShift was deployed using openshift-ansible, and the hosts file was configured to use an AWS user that has been configured with specific permissions.

A bucket, `openshift-1-registry`, was created by an administrative user. The OpenShift AWS user was provided the below permissions to be able to use that bucket.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::openshift-1-registry",
                "arn:aws:s3:::openshift-1-registry/*"
            ]
        }
    ]
}
```

I'm sure that could be firmed up, wildcards are frowned upon, but at least the OpenShift user only has access to his particular bucket. The user can't list all buckets, but can access `openshift-1-registry`.

Below I'm using the AWS CLI as an example of access permissions.

```
$ aws s3 ls

An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied
$ aws s3 ls s3://openshift-1-registry
                           PRE registry/
```

<br />

## Registry Information

As root on the controller instance I can find out the route for the docker registry. Of course this assumes that OpenShift has been setup with a wildcard apps domain.

```
openshift-controller# oc get routes
NAME               HOST/PORT                                   PATH      SERVICES           PORT      TERMINATION   WILDCARD
docker-registry    docker-registry-default.apps.example.com              docker-registry    <all>     passthrough   None
registry-console   registry-console-default.apps.example.com             registry-console   <all>     passthrough   None
```

*.apps.example.com is available through an ELB on AWS.

*NOTE: I've replaced the real URL with example.com.*

## Local Docker

This post assumes you have a local docker instance. In this example, docker 1.13.1 is running in a CentOS 7 host.

We need to add an insecure registry to the local docker (that is, of course, unless you have properly setup all the SSL certificates, which would be a good thing to do for production).

Edit Docker's daemon.json file to add the insecure registry.

```
local-docker$ sudo cat /etc/docker/daemon.json
{
  "insecure-registries" : ["docker-registry-default.apps.example.com:443"]
}
```

The restart it.

## Login

On the local docker, which also has the `oc` command line and access to OpenShift...

```
local-docker$ oc login
local-docker$ docker login -u $(oc whoami) -p $(oc whoami -t) docker-registry-default.apps.example.com:443
```

<br />

## Pull and Push and Image

Check what OC project you are in. Note that I've created a "example-project" project.

```
local-docker$ oc project
Using project "example-project" on server "https://openshift.example.com:443".
```

Pull, tag, and push an image.

```
local-docker$ docker pull docker.io/busybox
local-docker$ docker tag docker.io/busybox docker-registry-default.apps.example.com:443/example-project/busybox
local-docker$ docker push docker-registry-default.apps.example.com:443/example-project/busybox
```

<br />

## Conclusion

Now you're using a remote registry that is managed by OpenShift. The part I found most confusing was the project name and how it relates to the image tag.
