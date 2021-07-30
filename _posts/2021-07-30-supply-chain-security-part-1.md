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

There are many tools to build container images, not just Dockerfiles. In fact, in large organizations, Dockerfiles are IMHO an anti-pattern. You need a tool that can 1) build images without Dockerfiles and 2) separates the OS from the app. [Buildpacks](https://buildpacks.io) and more specifically [Paketo](https://paketo.io) solve these, and other, problems.

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

## Recommended Solution: Buildpacks and Paketo

I think that [Buildpacks](https://buildpacks.io/features/)--and more specifically [Paketo](https://paketo.io)--present a solution to many of the problems organizations will encounter when trying to build a secure software supply chain using container images.

With regards to my two main points:

* **No Dockerfiles** - Buildpacks do not use Dockerfiles at all. They are not hidden away through abstraction, they just don't exist. Can't edit what doesn't exist. (Though if you want to create custom buildpacks, you can use Dockerfiles)
* **Separation of Concerns** - The application, dependencies, and OS are separated out and in fact the resulting images can be "rebased" in which one layer is changed without affecting the others

### Using Pack and Paketo Buildpacks (but not Dockerfiles)

#### Paketo

First, what's Paketo? I would say that Paketo is a project that uses Buildpacks to provide container images that can run anywhere, including Kubernetes. As well they are supporting many runtimes. So, for the purposes of this post, we'll use Paketo Buildpacks to build apps. I would almost consider Paketo a distribution of usable, community generated buildpacks.

#### Pack

Pack is the tool that actually generates the Buildpack images so let's use pack to build a container image.

First, install pack which can be used to create buildpack based images. I'm doing so on Linux.

```
sudo add-apt-repository ppa:cncf-buildpacks/pack-cli
sudo apt-get update
sudo apt-get install pack-cli
```

Now I've got the pack cli.

```
$ which pack
/usr/bin/pack
$ pack version
0.19.0
```

Next, build an [app](https://buildpacks.io/docs/app-journey/). First checkout the sample.

```
git clone https://github.com/paketo-buildpacks/samples
cd samples/java/maven/
```

Now build the app. 

>NOTE: Here I'm calling the image "applications/maven".

>NOTE: I'm using the Paketo buildpack found at "paketobuildpacks/builder:base".

```
pack build applications/maven --builder paketobuildpacks/builder:base
```

Eg. output:

>NOTE: This is pulling Java dependencies, and, well, there are many of those to pull on the first build. 

```
$ pack build applications/maven --builder paketobuildpacks/builder:base
base: Pulling from paketobuildpacks/builder
Digest: sha256:4fae5e2abab118ca9a37bf94ab42aa17fef7c306296b0364f5a0e176702ab5cb
Status: Downloaded newer image for paketobuildpacks/builder:base
base-cnb: Pulling from paketobuildpacks/run
Digest: sha256:a285e73bc3697bc58c228b22938bc81e9b11700e087fd9d44da5f42f14861812
Status: Downloaded newer image for paketobuildpacks/run:base-cnb
===> DETECTING
7 of 18 buildpacks participating
paketo-buildpacks/ca-certificates   2.3.2
paketo-buildpacks/bellsoft-liberica 8.2.0
paketo-buildpacks/maven             5.3.2
paketo-buildpacks/executable-jar    5.1.2
paketo-buildpacks/apache-tomcat     5.6.1
paketo-buildpacks/dist-zip          4.1.2
paketo-buildpacks/spring-boot       4.4.2
===> ANALYZING
Previous image with name "applications/maven" not found
===> RESTORING
===> BUILDING

Paketo CA Certificates Buildpack 2.3.2
  https://github.com/paketo-buildpacks/ca-certificates
  Launch Helper: Contributing to layer
    Creating /layers/paketo-buildpacks_ca-certificates/helper/exec.d/ca-certificates-helper

Paketo BellSoft Liberica Buildpack 8.2.0
SNIP!
Paketo Spring Boot Buildpack 4.4.2
  https://github.com/paketo-buildpacks/spring-boot
  Creating slices from layers index
    dependencies
    spring-boot-loader
    snapshot-dependencies
    application
  Launch Helper: Contributing to layer
    Creating /layers/paketo-buildpacks_spring-boot/helper/exec.d/spring-cloud-bindings
  Spring Cloud Bindings 1.7.1: Contributing to layer
    Downloading from https://repo.spring.io/release/org/springframework/cloud/spring-cloud-bindings/1.7.1/spring-cloud-bindings-1.7.1.jar
    Verifying checksum
    Copying to /layers/paketo-buildpacks_spring-boot/spring-cloud-bindings
  Web Application Type: Contributing to layer
    Reactive web application detected
    Writing env.launch/BPL_JVM_THREAD_COUNT.default
  4 application slices
  Image labels:
    org.opencontainers.image.title
    org.opencontainers.image.version
    org.springframework.boot.version
===> EXPORTING
Adding layer 'paketo-buildpacks/ca-certificates:helper'
Adding layer 'paketo-buildpacks/bellsoft-liberica:helper'
Adding layer 'paketo-buildpacks/bellsoft-liberica:java-security-properties'
Adding layer 'paketo-buildpacks/bellsoft-liberica:jre'
Adding layer 'paketo-buildpacks/bellsoft-liberica:jvmkill'
Adding layer 'paketo-buildpacks/executable-jar:classpath'
Adding layer 'paketo-buildpacks/spring-boot:helper'
Adding layer 'paketo-buildpacks/spring-boot:spring-cloud-bindings'
Adding layer 'paketo-buildpacks/spring-boot:web-application-type'
Adding 5/5 app layer(s)
Adding layer 'launcher'
Adding layer 'config'
Adding layer 'process-types'
Adding label 'io.buildpacks.lifecycle.metadata'
Adding label 'io.buildpacks.build.metadata'
Adding label 'io.buildpacks.project.metadata'
Adding label 'org.opencontainers.image.title'
Adding label 'org.opencontainers.image.version'
Adding label 'org.springframework.boot.version'
Setting default process type 'web'
Saving applications/maven...
*** Images (d7dcc3fd9295):
      applications/maven
Adding cache layer 'paketo-buildpacks/bellsoft-liberica:jdk'
Adding cache layer 'paketo-buildpacks/maven:application'
Adding cache layer 'paketo-buildpacks/maven:cache'
Successfully built image applications/maven
```

That's created this "applications/maven" image in my local Docker.

```
$ docker images | grep application
applications/maven                                                    latest             d7dcc3fd9295   41 years ago    269MB
```

>NOTE: It says 41 years ago because it is a reproducable build. More on that maybe in other blog posts.

But where is the Dockerfile?

```
$ tree
.
├── bindings
│   └── maven
│       ├── settings.xml
│       └── type
├── mvnw
├── mvnw.cmd
├── pom.xml
├── README.md
└── src
    ├── main
    │   ├── java
    │   │   └── io
    │   │       └── paketo
    │   │           └── demo
    │   │               └── DemoApplication.java
    │   └── resources
    │       └── application.properties
    └── test
        └── java
            └── io
                └── paketo
                    └── demo
                        └── DemoApplicationTests.java

14 directories, 9 files
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
$ pack inspect applications/maven
Inspecting image: applications/maven

REMOTE:
(not present)

LOCAL:

Stack: io.buildpacks.stacks.bionic

Base Image:
  Reference: 5eaa2a599cd59e0e1d67132de78d590ef0f34512ede6acefd09416548f52a994
  Top Layer: sha256:10dd4d5e8186feb5b6ab2a877c80e1616e426ed383b7f19358b7703686fa4f9a

Run Images:
  index.docker.io/paketobuildpacks/run:base-cnb
  gcr.io/paketo-buildpacks/run:base-cnb

Buildpacks:
  ID                                         VERSION        HOMEPAGE
  paketo-buildpacks/ca-certificates          2.3.2          https://github.com/paketo-buildpacks/ca-certificates
  paketo-buildpacks/bellsoft-liberica        8.2.0          https://github.com/paketo-buildpacks/bellsoft-liberica
  paketo-buildpacks/maven                    5.3.2          https://github.com/paketo-buildpacks/maven
  paketo-buildpacks/executable-jar           5.1.2          https://github.com/paketo-buildpacks/executable-jar
  paketo-buildpacks/apache-tomcat            5.6.1          https://github.com/paketo-buildpacks/apache-tomcat
  paketo-buildpacks/dist-zip                 4.1.2          https://github.com/paketo-buildpacks/dist-zip
  paketo-buildpacks/spring-boot              4.4.2          https://github.com/paketo-buildpacks/spring-boot

Processes:
  TYPE                  SHELL        COMMAND        ARGS
  web (default)                      java           org.springframework.boot.loader.JarLauncher
  executable-jar                     java           org.springframework.boot.loader.JarLauncher
  task                               java           org.springframework.boot.loader.JarLauncher
```

Rebasing using a much older image. Of course, this would be done in reverse in the real world, where we would rebase with a newer image (that presumably has the security issues fixed). But for simplicity, given I've already created an image using the most recent run image, I'll go backwards here just for fun. Same idea no matter which way we go.

```
$ pack rebase applications/maven --run-image paketobuildpacks/builder:0.1.135-base
0.1.135-base: Pulling from paketobuildpacks/builder
71c12072e01c: Already exists 
8ac523e239f0: Pulling fs layer 
SNIP!
72ad9888618d: Pull complete 
4f4fb700ef54: Pull complete 
Digest: sha256:06fc9acb3b8098f7b717420d35f9cd8485ea1f92ce540769a2924ad7a161dad7
Status: Downloaded newer image for paketobuildpacks/builder:0.1.135-base
Rebasing applications/maven on run image paketobuildpacks/builder:0.1.135-base
Saving applications/maven...
*** Images (b72546026b22):
      applications/maven
Rebased Image: b72546026b22fff4797625e36b8f4a6c0e4a5386fcd5460c12d173cb1000718e
Successfully rebased image applications/maven
```

Inspect that version:

```
$ pack inspect applications/maven
Inspecting image: applications/maven

REMOTE:
(not present)

LOCAL:

Stack: io.buildpacks.stacks.bionic

Base Image:
  Reference: a8b66bfbe49565ffa1c74374ed0a38fb91adb43fa4a7a7c740b3f099b93a9c78
  Top Layer: sha256:5f70bf18a086007016e948b04aed3b82103a36bea41755b6cddfaf10ace3c6ef

Run Images:
  index.docker.io/paketobuildpacks/run:base-cnb
  gcr.io/paketo-buildpacks/run:base-cnb

Buildpacks:
  ID                                         VERSION        HOMEPAGE
  paketo-buildpacks/ca-certificates          2.3.2          https://github.com/paketo-buildpacks/ca-certificates
  paketo-buildpacks/bellsoft-liberica        8.2.0          https://github.com/paketo-buildpacks/bellsoft-liberica
  paketo-buildpacks/maven                    5.3.2          https://github.com/paketo-buildpacks/maven
  paketo-buildpacks/executable-jar           5.1.2          https://github.com/paketo-buildpacks/executable-jar
  paketo-buildpacks/apache-tomcat            5.6.1          https://github.com/paketo-buildpacks/apache-tomcat
  paketo-buildpacks/dist-zip                 4.1.2          https://github.com/paketo-buildpacks/dist-zip
  paketo-buildpacks/spring-boot              4.4.2          https://github.com/paketo-buildpacks/spring-boot

Processes:
  TYPE                  SHELL        COMMAND        ARGS
  web (default)                      java           org.springframework.boot.loader.JarLauncher
  executable-jar                     java           org.springframework.boot.loader.JarLauncher
  task                               java           org.springframework.boot.loader.JarLauncher
```

If I run that app, which was rebased onto a much older run image...

```
$ docker run --rm -p 8080:8080 applications/maven
Setting Active Processor Count to 12
Calculating JVM memory based on 38994352K available memory
Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -Xmx38598153K -XX:MaxMetaspaceSize=88998K -XX:ReservedCodeCacheSize=240M -Xss1M (Total Memory: 38994352K, Thread Count: 50, Loaded Class Count: 13299, Headroom: 0%)
Adding 129 container CA certificates to JVM truststore
Spring Cloud Bindings Enabled
Picked up JAVA_TOOL_OPTIONS: -Djava.security.properties=/layers/paketo-buildpacks_bellsoft-liberica/java-security-properties/java-security.properties -agentpath:/layers/paketo-buildpacks_bellsoft-liberica/jvmkill/jvmkill-1.16.0-RELEASE.so=printHeapHistogram=1 -XX:ActiveProcessorCount=12 -XX:MaxDirectMemorySize=10M -Xmx38598153K -XX:MaxMetaspaceSize=88998K -XX:ReservedCodeCacheSize=240M -Xss1M -Dorg.springframework.cloud.bindings.boot.enable=true

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.3)

2021-07-30 13:44:52.093  INFO 1 --- [           main] io.paketo.demo.DemoApplication           : Starting DemoApplication v0.0.1-SNAPSHOT using Java 11.0.12 on 331541f6d651 with PID 1 (/workspace/BOOT-INF/classes started by cnb in /workspace)
2021-07-30 13:44:52.096  INFO 1 --- [           main] io.paketo.demo.DemoApplication           : No active profile set, falling back to default profiles: default
2021-07-30 13:44:52.906  INFO 1 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 1 endpoint(s) beneath base path '/actuator'
2021-07-30 13:44:53.184  INFO 1 --- [           main] o.s.b.web.embedded.netty.NettyWebServer  : Netty started on port 8080
2021-07-30 13:44:53.196  INFO 1 --- [           main] io.paketo.demo.DemoApplication           : Started DemoApplication in 1.376 seconds (JVM running for 1.646)
```

So as you can see it's simple and fast to "rebase" an image, ie. swap out the version of the OS but NOT the application, without having to go through an entire build.

## Conclusion

Dockerfiles are great, but, IMHO, not for building a secure software supply chain (not without considerable extra work at least). There are other ways to build container images that lend themselves more easily to building a secure software supply chain.

I should mention that buildpacks and pack are just part of a full solution for managing images. Please check out [kpack](https://github.com/pivotal/kpack) and the [Tanzu Build service](https://tanzu.vmware.com/build-service) for more thoughts on what else is needed in the Kubernetes ecosystem. More on that in future posts.

## Thanks

Please note that the container flipping image in the title image is borrowed from the [ko project](https://github.com/google/ko). I'm using it because I think it's hilarious, not because I necessarily am suggesting ko is a great build tool--I haven't used it.