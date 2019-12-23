---
layout: post
title: Cloud Native Buildpacks
categories:
header_image: "/img/wolfpack.jpg"

---

# {{ page.title }}

How does code turn into an application running on Kubernetes? Good question. Many things have to happen: Dockerfiles written and rewritten, base images picked, builds pushed to repositories, and, of course, much k8s YAML wrangled. These simple phrases, e.g. "Dockerfile written," represent considerable complexity. 

How can we make getting code into production simpler for everyone? 

One way is to use [buildpacks](https://buildpacks.io), which help to make the generation of [OCI images](https://www.opencontainers.org/) easier. What if I told you that with buildpacks you don't even *NEED* a Dockerfile?

## Building Container Images is Hard - Use Buildpacks

For quite a while Docker was the only easy way to build container images. It's still responsible for building the vast majority container images. However, with the creation of the [OCI image spec](https://github.com/opencontainers/image-spec) other tools have been developed.

It can't be denied that giving development teams the ability to create their own runtime images improved the developer experience. They could run the same image locally as what would, in theory, go into production. They knew the apps dependencies and could add them to the Dockerfile. 

However, in the socio-technical realm of enterprise organizations, operations typically does not allow arbitrary container images in production, for various reasons, some valid, some not. Further to that, many ops organizations will try to manage container images exactly like they manage virtual machine templates (*if* they manage them at all) which is an anti-pattern.

Ultimately Dockerfiles make building operationally resilient images **look** easy, but it's not.

Here are a few considerations one has to make when building container images:

* Keeping image sizes small
* Dealing with dependencies
* Picking the right base image
* Ensuring appropriate application memory settings
* Day 2 ops - e.g. How to update the JDK without breaking the image cache or rebasing on a new base image
* Defining who is responsible for the images - Ops, Dev, or DevOps?
* Dealing with CVEs

Issues with typical container images:

* Lack of app awareness
* Not composable - How do we combine images? (Only have multistage builds in Dockerfiles)
* Leaky abstraction - e.g. Container images mix operational concerns with application development concerns
* Non-reproducible/untestable builds
* Security - e.g. many Dockerfiles assume running as root
* Treating containers images as VMs

## Enter Buildpacks.io

>Buildpacks are pluggable, modular tools that translate source code into OCI images.

Buildpacks are currently a CNCF Sandbox project supported by companies like Pivotal and Heroku.

>[Buildpacks](https://medium.com/buildpacks/cloud-native-buildpacks-hit-beta-4d9f2c85dd22s) have always been about the developer experience. We want buildpacks to make your job easier by eliminating operational and platform concerns that you might otherwise need to consider when using containers.

Buildpacks are a higher level abstraction than Dockerfiles and are really meant for developers.

What do you [get](https://buildpacks.io/#learn-more)?

> * Provide a balance of control that reduces the operational burden on developers and supports enterprise operators who manage apps at scale
> * Ensure that apps meet security and compliance requirements without developer intervention
> * Provide automated delivery of both OS-level and application-level dependency upgrades, efficiently handling day-2 app operations that are often difficult to manage with Dockerfiles
> * Rely on compatibility guarantees to safely apply patches without rebuilding artifacts and without unintentionally changing application behavior

## Using Buildpacks

I run on Linux so I just [downloaded the binary](https://github.com/buildpacks/pack/releases) into my home bin directory and made it executable.

```
$ which pack
~/bin/pack
```

Let's clone a repo, *spring-music*, and build an image.

```
$ git clone https://github.com/cloudfoundry-samples/spring-music
```

Now cd into spring-music. Note no Dockerfiles exist. Just plain code and build files.

```
$ cd spring-music
$ ls
build.gradle  gradle  gradle.properties  gradlew  gradlew.bat  LICENSE  manifest.yml  README.md  src
```

Now, with one simple command, let's build a hardened container image that has been run millions of times on the Pivotal Platform and Heroku.

```
$ pack build spring-music
```

Here's the resulting image...264MB:

```
$ docker images spring-music
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
spring-music        latest              0ea601302547        15 minutes ago      264MB
```

Run it.

```
$ docker run --rm -d -p 8080:8080 spring-music
```

Curl localhost:8080 to test.

```
 $ curl -s localhost:8080 | grep title
    <meta name="title" content="Spring Music">
    <title>Spring Music</title>
```

Well, that was pretty easy...and no Dockerfiles!

## A Bit More

While there is a lot to...er..."unpack," here, I'd like to point out a couple of things that I think are interesting and important about buildpacks.

* Disconnection of Operating System from Application

With standard Docker images when a security issue is discovered in a operating system package many, if not all, of the image layers have to be rebuilt. This makes redeploying applications slow. It also handcuffs the OS requirements to the app requirements. With buildpacks these concerns are separated.

* Java BuildPack Memory Calculator

I find that it's quite easy to ignore or forget JVM memory settings...especially in a container centric world. Buildpacks use the [Java BuildPack Memory Calculator](https://github.com/cloudfoundry/java-buildpack-memory-calculator) to dynamically set requirements for memory. I have yet to see a Dockerfile that implements this or anything like it. *

## Conclusion

Building hardened, operationally reliable container images is difficult. Using Buildpacks not only makes Dockerfiles unnecessary, but provides access to images that have been in constant, heavy production use for years.

Using pack and buildpacks you don't need to:

* Write your own Dockerfiles
* Add individual files before app code to improve caching
* Dance around for user permissions and root access
* Force docker to rebuild all layers for a security patch 
* Understand the base operating system 
* Couple applications to the build pipeline
* Burn in images over time to feel that they are trustworthy

Running applications in production isn't easy. Using buildpacks can help to reduce the operational burden, for everyone.

I recommend watching a [couple](https://www.youtube.com/watch?v=spW9ZlJpobM&t=407s) of [videos](https://www.youtube.com/watch?v=KiDK5C0kzJs) for more in-depth information on buildpacks.

----

\* Hat tip to my colleague Adib Saikali for information on the Java memory calculator. Watch his Toronto Java meetup talk on [Spring and Kubernetes](https://www.youtube.com/watch?v=meE888uPLyc).