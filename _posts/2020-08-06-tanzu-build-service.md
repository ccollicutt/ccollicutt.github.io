---
layout: post
title:  Quick Look at the Tanzu Build Service
categories:
header_image: "/img/tbs.jpg"
summary: "Use Kubernetes to automatically build container images on git commit"

---

# {{ page.title }}

Building container images isn't easy. Dockerfiles make it look easy, but it's not. Not in the real world anyways. Container image security is hard. Managing dependencies is hard. Building pipelines (good ones)...is hard. Building pipelines on your own to do the same thing that everyone deploying apps on Kubernetes has to do--i.e. build images--likely doesn't make (business) sense. Why build it on your own with bash and CI/CD systems when you can use something like kpack or the Tanzu Build Service...and make it declarative ala Kubernetes (you know, the thing you are actually deploying the app to).

*Aside: I use the term pipelines all the time, but often CI/CD pipelines are just poorly tested bash spaghetti code that ties together a few actions in a row. See my homelab automation haha. That sounds harsh, but it's fair to say that the quality level of pipeline code is a wide, wide spectrum. And, of course, is usually not declarative.*

## Tanzu Build Service

Tanzu Build Service (TBS) helps with automating image creation, as well as using buildpacks as the base image. To quote our [Tanzu Build Service](https://tanzu.vmware.com/build-service) site:

>Tanzu Build Service uses the magic of Cloud Native Buildpacks to rebase app images when specialized contractual base images are updated in a registry. This means you can resolve certain common vulnerabilities and exposures (CVE) without a rebuild. 

TBS uses Kubernetes CRDs.

> Tanzu Build Service follows a Kubernetes-native declarative model and executes builds automatically against user-defined configuration. It includes kpack, a set of open-source resource controllers for Kubernetes maintained by VMware Tanzu.

It's based on the upstream, open source tool [kpack](https://github.com/pivotal/kpack).

If you want to review the features, checkout this [page](https://buildpacks.io/features/).

## How to Get the Tanzu Build Service

As of today, TBS is not GA (generally available) but if you sign up for the [Tanzu Network](https://network.pivotal.io/) you can download the 0.2 non-GA release. 

## Install

I'm not going to go over the install as it currently uses Duffle. There's a great [blog post](https://tanzu.vmware.com/content/practitioners/getting-started-with-vmware-tanzu-build-service-0-2-0-beta) that covers the Duffle-based install, configuring git repos, credentials, ect. This is exactly what I followed for this blog post.

In my case, I'm using Docker Hub as the image repository, but I have a local Gitlab install that I'm pulling code from. That said TBS can use most any image registry or git server.

## Build an Image

I've gone ahead and installed TBS. Now that it's installed, we have several CRDs available.

```
$ k get crd | grep pivotal
builders.build.pivotal.io                             2020-08-04T19:50:47Z
builds.build.pivotal.io                               2020-08-04T19:50:47Z
clusterbuilders.build.pivotal.io                      2020-08-04T19:50:47Z
custombuilders.experimental.kpack.pivotal.io          2020-08-04T19:50:47Z
customclusterbuilders.experimental.kpack.pivotal.io   2020-08-04T19:50:47Z
images.build.pivotal.io                               2020-08-04T19:50:47Z
sourceresolvers.build.pivotal.io                      2020-08-04T19:50:47Z
stacks.experimental.kpack.pivotal.io                  2020-08-04T19:50:47Z
stores.experimental.kpack.pivotal.io                  2020-08-04T19:50:47Z
```

I've already built an image based on the Spring Petclinic code.

```
$ k get img
NAME               LATESTIMAGE                                                                                                                READY
spring-petclinic   index.docker.io/ccollicutttanzu/spring-petclinic@sha256:994c9fb87f591c734a86c2a8d18ca4f0beb339ee9cbe1e6ae19c989b541fd773   True
```

TBS comes with a CLI called kp.

To create an image:

```
kp image create spring-petclinic ccollicutttanzu/spring-petclinic --git git@gitlab.example.com:root/test-project.git
```

I've built it a few times already.

```
$ kp build list spring-petclinic
BUILD    STATUS     IMAGE                                                                                                                       STARTED                FINISHED               REASON
SNIP!
6        SUCCESS    index.docker.io/ccollicutttanzu/spring-petclinic@sha256:eaed806fb9027a1b12d507464faab637afe92e76e3f995cb318a55275b3f9fa4    2020-08-06 10:00:51    2020-08-06 10:21:28    TRIGGER
7        SUCCESS    index.docker.io/ccollicutttanzu/spring-petclinic@sha256:994c9fb87f591c734a86c2a8d18ca4f0beb339ee9cbe1e6ae19c989b541fd773    2020-08-06 10:59:25    2020-08-06 11:03:47    COMMIT

```

For build 6 I triggered it manually with the kp CLI.

```
kp image trigger spring-petclinic
```

For build 7 I simply committed a change to the Spring Petclinic git repo (my git repo) and TBS noticed the commit and built the new image. So on every commit you'll get a new image. Of course that sounds like a lot, but most organizations would have some kind of production branch, maybe it's main, and code only makes it to that branch once it's tested.

Now that you have an image, you could use some kind of gitops workflow to deploy based on that new image, and you could use something like the [kapp-controller](https://github.com/k14s/kapp-controller) to automate that.


Here's a snippet from the build logs.

```
$ kp build logs spring-petclinic -b 7 -n build-service | tail -20
Reusing layer 'launcher'
Reusing layer 'paketo-buildpacks/bellsoft-liberica:class-counter'
Reusing layer 'paketo-buildpacks/bellsoft-liberica:java-security-properties'
Reusing layer 'paketo-buildpacks/bellsoft-liberica:jre'
Reusing layer 'paketo-buildpacks/bellsoft-liberica:jvmkill'
Reusing layer 'paketo-buildpacks/bellsoft-liberica:link-local-dns'
Reusing layer 'paketo-buildpacks/bellsoft-liberica:memory-calculator'
Reusing layer 'paketo-buildpacks/bellsoft-liberica:openssl-security-provider'
Reusing layer 'paketo-buildpacks/bellsoft-liberica:security-providers-configurer'
Reusing layer 'paketo-buildpacks/executable-jar:class-path'
Adding 1/1 app layer(s)
Adding layer 'config'
*** Images (sha256:994c9fb87f591c734a86c2a8d18ca4f0beb339ee9cbe1e6ae19c989b541fd773):
      ccollicutttanzu/spring-petclinic
      index.docker.io/ccollicutttanzu/spring-petclinic:b7.20200806.145925
Reusing cache layer 'paketo-buildpacks/bellsoft-liberica:jdk'
Adding cache layer 'paketo-buildpacks/maven:application'
Reusing cache layer 'paketo-buildpacks/maven:cache'
===> COMPLETION
Build successful
```

Above you can see [buildpacks](https://buildpacks.io/) in use by TBS to create the container image.

## Conclusion

This is just a quick example of TBS. While this looks very simple, and sometimes simple things look like they have no value, I can assure you there are a lot of things happening in the background, and without something like TBS organizations are typically left on their own to build it, and often fail, or only partially succeed.
