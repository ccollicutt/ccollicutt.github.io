---
layout: post
title:  "Software Supply Chain Security Part 1 - Container Images"
categories:
header_image: "/img/sc-1.jpg"
header_permalink: "https://unsplash.com/photos/MzGfNGsONPQ"
summary: "Dockerfiles aren't the only way to create container images..."

---

# {{ page.title }}

**Problem #1: I need to package software**

**Solution: Dockerfiles**

**New Problem: I need to manage Dockerfiles**

**New solution 2: ???**

## Overview

There are many pieces to a modern, secure software supply chain. I say modern because this series of posts will focus on what it takes to build a secure software supply chain when the target is Kubernetes--and Kubernetes means containers...and containers mean, you guessed it, container images. 

## tl;dr

There are many tools to build container images, not just Dockerfiles. In fact, in large organizations, Dockerfiles are IMHO an anti-pattern. You need a tool that can 1) build images without Dockerfiles and 2) separates the OS from the app. [Buildpacks](https://buildpacks.io) solve these, and other, problems.

## Container Images

I talk to many organizations about container images. The reality is that almost everyone equates container images with Dockerfiles, meaning most people believe the only way to create a container image, which they might call a "Docker image" is by using a Dockerfile.

If you get any one thing from this post, it's important to understand what a container image really is. What it *really* is...is an open source specification that defines the "file bundle" that makes up what we call a "container image." (Ultimately, at this time, a container image is a glorified tar file.)

>This specification defines how to create an OCI Image, which will generally be done by a build system, and output an image manifest, a filesystem (layer) serialization, and an image configuration. - [OCI](https://opencontainers.org/about/overview/)

It's important to understand that anyone can build a tool that can create an image which meets this specification. Anyone. It *does not* have to be based on Dockerfiles. In fact I would suggest that while Dockerfiles are great they are not necessarily the best tool to use as part of building a secure software supply chain. There are other solutions, not many, but there are definitely choices that can be made (and I present one of them in this post).

## Examples of OCI Compliant Container Image Build Tools

First, what other tools are out there for building OCI compliant images?

>NOTE: Not all of these tools are maintained, and not all would be usable in production. This is just a list to show that there are several tools one can use build an OCI compliant image, not all of which use Dockerfiles. (However, that said, I don't think there are quite enough tools to show the vibrant OCI image building ecosystem that one would expect given the popularity of containers.)

Here's an incomplete list:

* [Buildpacks](https://buildpacks.io/)
* [img](https://github.com/genuinetools/img)
* Buildah
* Kaniko
* Jib
* [s2i](https://github.com/openshift/source-to-image)
* [Buildkit](https://github.com/moby/buildkit)
* [ko](https://github.com/google/ko)

As can be seen from the above there are several tools which could be used, as opposed to "docker build...". Each of these tools makes different architectural and usability choices. In fact some target only specific runtimes, eg. ko targets golang apps.

## What Makes a Good Secure OCI Image Build Tool?

### No Dockerfiles

My opinion is that a secure supply chain requires that there is, effectively, preferably, no Dockerfile. In my opinion, there's too much power in Dockerfiles, too many ways to make mistakes and create security issues to allow people to have access to them, or for them to even be available.

To me Dockerfiles are an anti-pattern, especially in large organizations with many applications. However, please don't get me wrong, Dockerfiles have been and will continue to be an amazing tool for developers to build container images, bringing that capability to the masses. That said, using then as part of a secure supply chain is challenging...I believe too challenging for most organizations. It's preferable for the image build tool to not use Dockerfiles or at least abstract (hide) them away from the developers and application ops teams. However, when hiding things in technology we know that issues can and will still [leak out](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/), so perhaps best to just not have Dockerfiles at all.

### Separation of Concerns

![hashed layers](/img/hashed.jpg)

I want a OCI build tool that separates, at the very least, the operating system from the application. I want to be able to swap out the OS without breaking, or even affecting, the application. This is because if there is one thing that we can depend on in IT, it's that the OS will have security issues, ie. CVEs, and we'll need to fix those CVEs to remain secure. However, if the application, and its dependencies, and its runtime, and the OS are all hashed together into a container image, and we can't swap any one of those without affecting the other layers, then that is a major security issue, as development and/or application operation teams will be (very) hesitant to update the image because they don't know what will happen to the application.

An easy way to test is this capability is available in an OCI image tool is if we can use it to build an image "out of band" from the build pipeline. Can we update the OS of an image, most likely to fix any CVEs or bugs, and roll that image out to all applications that use it, without having to go through the entire application build pipeline? In most situations that would not be possible because the build pipeline and the images are so tightly intertwined that it is not feasible. But it *should be*.

### Other Needs

There are other things that a great OCI image build tool should have. I'll list a few here, but I don't want this post to go on for too long. (I may tackle these in a future blog post.)

* Reproducable builds
* Built-in intelligence
* Just enough customization capability
* Bill of materials (BoM)
* Minimal attack surface
* Support many runtimes
* Caching
* Build images in unprivileged containers

## Recommended Solution: Buildpacks/Paketo

I think that [Buildpacks](https://buildpacks.io/features/) present a solution to a lot of the problems organizations will encounter when trying to build a secure software supply chain using container images.

With regards to my two main points:

* **No Dockerfiles** - Buildpacks do not use Dockerfiles at all. They are not hidden away through abstraction, they just don't exist. Can't edit what doesn't exist. (Though if you want to create custom buildpacks, you can use Dockerfiles)
* **Separation of Concerns** - The application, dependencies, and OS are separated out and in fact the resulting images can be "rebased" in which one layer is changed without affecting the others

### Using kpack and buildpacks (but not Dockerfiles)

Let's use pack to build a container image.

First, install kpack which can be used to create buildpack based images. I'm doing so on Linux.

```
sudo add-apt-repository ppa:cncf-buildpacks/pack-cli
sudo apt-get update
sudo apt-get install pack-cli
```

Now I've got pack.

```
$ which pack
/usr/bin/pack
$ pack version
0.19.0
```

Build an [app](https://buildpacks.io/docs/app-journey/). 

```
cd /tmp
# clone the repo
git clone https://github.com/buildpacks/samples
# go to the app directory
cd samples/apps/java-maven
# build the app
pack build myapp --builder cnbs/sample-builder:bionic
```

Eg. output:

>NOTE: This is pulling Java dependencies, and, well, there are a lot of those to pull on the first build. 

```
$ pack build myapp --builder cnbs/sample-builder:bionic
bionic: Pulling from cnbs/sample-builder
e7ae86ffe2df: Pull complete 
15999ec6d9f8: Pull complete 
bf5c9240fe8d: Pull complete 
3e1318bc758f: Pull complete 
0bc36cd0f998: Pull complete 
f3822bcb9f02: Pull complete 
357fefdf9bc9: Pull complete 
a697cc8a1d5c: Pull complete 
5c2e4179bee1: Pull complete 
c4e3bdcbb8c3: Pull complete 
04f9e5a54d38: Pull complete 
c238db6a02a5: Pull complete 
0cceee8a8cb0: Pull complete 
53a52c7f9926: Pull complete 
a302059dbdba: Pull complete 
db1bbcc47135: Pull complete 
4f4fb700ef54: Pull complete 
Digest: sha256:b7edf381d738273ae4b382221c49385df28dd551839db584b76f4c65893142e8
Status: Downloaded newer image for cnbs/sample-builder:bionic
bionic: Pulling from cnbs/sample-stack-run
e7ae86ffe2df: Already exists 
15999ec6d9f8: Already exists 
bf5c9240fe8d: Already exists 
Digest: sha256:7606c5881eaa0a1aebb65861f12928f9f253c242d46c4ba836b2c002e2bb3cfb
Status: Downloaded newer image for cnbs/sample-stack-run:bionic
0.11.3: Pulling from buildpacksio/lifecycle
5dea5ec2316d: Pull complete 
6d75e71a489d: Pull complete 
Digest: sha256:d6c578fbdf88f2e2594d9907307b17775e648656f62e1ae810d31c804f928cf9
Status: Downloaded newer image for buildpacksio/lifecycle:0.11.3
===> DETECTING
SNIP!
[builder] [INFO] Installing /workspace/pom.xml to /home/cnb/.m2/repository/io/buildpacks/example/sample/0.0.1-SNAPSHOT/sample-0.0.1-SNAPSHOT.pom
[builder] [INFO] ------------------------------------------------------------------------
[builder] [INFO] BUILD SUCCESS
[builder] [INFO] ------------------------------------------------------------------------
[builder] [INFO] Total time:  6.294 s
[builder] [INFO] Finished at: 2021-07-27T20:11:15Z
[builder] [INFO] ------------------------------------------------------------------------
===> EXPORTING
[exporter] Adding layer 'samples/java-maven:jdk'
[exporter] Adding 1/1 app layer(s)
[exporter] Adding layer 'launcher'
[exporter] Adding layer 'config'
[exporter] Adding layer 'process-types'
[exporter] Adding label 'io.buildpacks.lifecycle.metadata'
[exporter] Adding label 'io.buildpacks.build.metadata'
[exporter] Adding label 'io.buildpacks.project.metadata'
[exporter] Setting default process type 'web'
[exporter] Saving myapp...
[exporter] *** Images (86fc0928e7a1):
[exporter]       myapp
[exporter] Adding cache layer 'samples/java-maven:jdk'
[exporter] Adding cache layer 'samples/java-maven:maven_m2'
Successfully built image myapp
```

That's created this myapp image in my local Docker.

```
$ docker images | grep myapp
myapp                                                                 latest             ffc5bf25a436   41 years ago     300MB
```

>NOTE: It says 41 years ago because it is a reproducable build. More on that maybe in other blog posts.

But where is the Dockerfile?

```
$ tree
.
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    └── main
        ├── java
        │   └── io
        │       └── buildpacks
        │           └── example
        │               └── sample
        │                   └── SampleApplication.java
        └── resources
            ├── application.properties
            ├── banner.txt
            ├── static
            │   ├── buildpacks-logo.svg
            │   └── favicon.png
            └── templates
                └── index.html

10 directories, 9 files
```

There is none! pack uses buildpacks and does NOT use a Dockerfile. Nice!

### Buildpack Rebasing - Out of Band Image Updates

Most customers I talk to have to push an image through the entire build pipeline to build it...where "it" is the OS, dependencies, and application artifacts. This means that any time there is a CVE, the entire build must be run. What this also suggests is that the ability for the application to properly run is also tied to the entirety of the image. This makes updating images when there isn't an application change challenging, as no one is really sure if it's going to work or not...

But with buildpacks, the application, OS, and runtimes, and dependencies (and more) are separated out into individual pieces that can be swapped out without harming the application. With buildpacks this is called [rebasing](https://buildpacks.io/docs/concepts/operations/rebase/).

>Rebase allows app developers or operators to rapidly update an app image when its stack's run image has changed. By using image layer rebasing, this command avoids the need to fully rebuild the app.

The term rebase mostly comes from the world of git:

>Rebasing is the process of moving or combining a sequence of commits to a new base commit. -- [git rebase](https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase)

So let's rebase the image.

Inspect the current version.

```
$ pack inspect myapp
Inspecting image: myapp

REMOTE:
(not present)

LOCAL:

Stack: io.buildpacks.samples.stacks.bionic

Base Image:
  Reference: 0b2076fcd92b5a12e379c33818b97273f0380eba1850d2088fe2ce5576e4c767
  Top Layer: sha256:09ac4cd31335f0b332b1b0718986fafbce0b29d18894c0bd92b2a0dec2aaec4a

Run Images:
  cnbs/sample-stack-run:bionic

Buildpacks:
  ID                        VERSION        HOMEPAGE
  samples/java-maven        0.0.1          https://github.com/buildpacks/samples/tree/main/buildpacks/java-maven

Processes:
  TYPE                 SHELL        COMMAND        ARGS
  web (default)        bash         java -jar target/sample-0.0.1-SNAPSHOT.jar
```

Rebasing using a much older image (and specifying this image by sha256; don't ask me how I found the older image). Of course, this would be done in reverse in the real world, where we would rebase with a newer image (that presumably has the security issues fixed). But for simplicity, given I've already created an image using the most recent run image, I'll go backwards here just for fun.

```
$ pack rebase myapp --run-image cnbs/sample-stack-run:bionic@sha256:2ca156537bad95e4870e5e8a74b10e7b46b722add5ebb6eda9939e8ce8c7143b
docker.io/cnbs/sample-stack-run@sha256:2ca156537bad95e4870e5e8a74b10e7b46b722add5ebb6eda9939e8ce8c7143b: Pulling from cnbs/sample-stack-run
Digest: sha256:2ca156537bad95e4870e5e8a74b10e7b46b722add5ebb6eda9939e8ce8c7143b
Status: Image is up to date for cnbs/sample-stack-run@sha256:2ca156537bad95e4870e5e8a74b10e7b46b722add5ebb6eda9939e8ce8c7143b
Rebasing myapp on run image cnbs/sample-stack-run:bionic@sha256:2ca156537bad95e4870e5e8a74b10e7b46b722add5ebb6eda9939e8ce8c7143b
Saving myapp...
*** Images (5a08cc228d95):
      myapp
Rebased Image: 5a08cc228d95c0880b2d73593d68e5585abe5ae3bd8f8badf47213a50a7508a8
Successfully rebased image myapp
```

Inspect that version:

```
$ pack inspect myapp
Inspecting image: myapp

REMOTE:
(not present)

LOCAL:

Stack: io.buildpacks.samples.stacks.bionic

Base Image:
  Reference: afae70542bd2ad4e5b5bfa0830757f11c9dfc87f1bdd1c91b1ae6a9f5c6825bb
  Top Layer: sha256:d2b6c5c45d095d805d8b7ddab2028fa94be7e8efb2814ac1a75d3e9319ae8244

Run Images:
  cnbs/sample-stack-run:bionic

Buildpacks:
  ID                        VERSION        HOMEPAGE
  samples/java-maven        0.0.1          https://github.com/buildpacks/samples/tree/main/buildpacks/java-maven

Processes:
  TYPE                 SHELL        COMMAND        ARGS
  web (default)        bash         java -jar target/sample-0.0.1-SNAPSHOT.jar
```

If I run that app, which was rebased onto a much older run image...

```
$ docker run --rm -p 8080:8080 myapp
    |'-_ _-'|       ____          _  _      _                      _             _
    |   |   |      |  _ \        (_)| |    | |                    | |           (_)
     '-_|_-'       | |_) | _   _  _ | |  __| | _ __    __ _   ___ | | __ ___     _   ___
|'-_ _-'|'-_ _-'|  |  _ < | | | || || | / _` || '_ \  / _` | / __|| |/ // __|   | | / _ \
|   |   |   |   |  | |_) || |_| || || || (_| || |_) || (_| || (__ |   < \__ \ _ | || (_) |
 '-_|_-' '-_|_-'   |____/  \__,_||_||_| \__,_|| .__/  \__,_| \___||_|\_\|___/(_)|_| \___/
                                              | |
                                              |_|

:: Built with Spring Boot :: 2.1.18.RELEASE
SNIP!
```

I can curl it.

```
$ curl -s localhost:8080 | grep Hello
        <h1>Hello, Buildpacker!</h1>
```

So as you can see it's simple and fast to "rebase" an image, ie. swap out the version of the OS but NOT the application, without having to go through an entire build, which indicates to me that the app and OS are separate components.

## Conclusion

Dockerfiles are great, but not for building a secure software supply chain (not without a lot of extra work at least). There are other ways to build container images that lend themselves more easily to building a secure software supply chain.

I should mention that buildpacks and pack are just part of a full solution for managing images. Please check out [kpack](https://github.com/pivotal/kpack) and the [Tanzu Build service](https://tanzu.vmware.com/build-service) for more thoughts on what else is needed in the Kubernetes ecosystem.

## Thanks

Please note that the container flipping image in the title image is borrowed from the [ko project](https://github.com/google/ko). I'm using it because I think it's hilarious, not because I necessarily am suggesting ko is a great build tool--I haven't used it.