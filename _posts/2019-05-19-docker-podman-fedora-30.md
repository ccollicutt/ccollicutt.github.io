---
layout: post
title: Install and Use Podman (Instead of Docker)
categories:
header_image: "/img/pod.jpg"
header_permalink: "https://unsplash.com/photos/XlA2994Txhw"

---

# {{ page.title }}

## tl;dr

I reinstalled my laptop with Fedora 30. I tried to install a stable Docker, but the Docker repo for Fedora 30 stable doesn't exist. I realized podman is available and is a command for command replacement for Docker. So far I'm quite happy with it and am actually kind of glad I was forced into it.

## What Distro to Run?

Recently I reinstall my laptop with Fedora 30. I won't mention what version of Fedora I "upgraded" from, but suffice it to say I was using my old install for quite a while. I wouldn't have minded using Fedora Silverblue, Fedora CoreOS, or CentOS 8. However, two of those don't have anything to install yet, and Silverblue...it seems a bit too early for me. So I stuck with good old Fedora.

I don't actually need much from Linux. I use the i3 window manager, Firefox browser, smartcd, VSCode (from Microsoft) and a few other tools. As long as I can apt/yum/dnf install common tools the Linux distribution I use doesn't really matter. I don't actually even use Docker locally that much, but I do use it to manage this blog, and it's nice to have a container runtime available.

## Docker on Fedora 30 (or lack thereof)

After reinstalling my laptop, I went to [install docker](https://docs.docker.com/install/linux/docker-ce/fedora/). In retrospect I should have read the docs better:

>To install Docker, you need the 64-bit version of one of these Fedora versions:
>    28
>    29

30 is not 28 or 29. :) Regardless, I kept going.

I installed the official repo.

```
$ sudo dnf config-manager \
>     --add-repo \
>     https://download.docker.com/linux/fedora/docker-ce.repo
Adding repo from: https://download.docker.com/linux/fedora/docker-ce.repo
$ head /etc/yum.repos.d/docker-ce.repo 
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/fedora/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://download.docker.com/linux/fedora/$releasever/debug-$basearch/stable
```

Then I tried to install docker.

```
$ sudo dnf install docker-ce docker-ce-cli containerd.io
Docker CE Stable - x86_64                                                                                                                   1.7 kB/s | 577  B     00:00    
Failed to synchronize cache for repo 'docker-ce-stable'
Fedora Modular 30 - x86_64                                                                                                                   46 kB/s |  17 kB     00:00    
Fedora Modular 30 - x86_64 - Updates                                                                                                         69 kB/s |  16 kB     00:00    
Fedora 30 - x86_64 - Updates                                                                                                                 55 kB/s |  17 kB     00:00    
Fedora 30 - x86_64 - Updates                                                                                                                645 kB/s | 625 kB     00:00    
Fedora 30 - x86_64                                                                                                                           52 kB/s |  17 kB     00:00    
RPM Fusion for Fedora 30 - Free - Updates                                                                                                   592  B/s | 3.0 kB     00:05    
RPM Fusion for Fedora 30 - Free                                                                                                             9.8 kB/s | 3.2 kB     00:00    
RPM Fusion for Fedora 30 - Nonfree - Updates                                                                                                4.7 kB/s | 3.0 kB     00:00    
RPM Fusion for Fedora 30 - Nonfree                                                                                                          4.4 kB/s | 3.2 kB     00:00    
Ignoring repositories: docker-ce-stable
No match for argument: docker-ce
No match for argument: docker-ce-cli
No match for argument: containerd.io
Error: Unable to find a match
```

Doh. Seems like no "docker-ce-stable" repo. I don't want to run nightly or test. What to do? Get docker from somewhere else? Or is there an alternative...I remember something about podman...

I removed the Docker repo.

```
 sudo rm docker-ce.repo 
```

Onto podman.

## Install podman

podman, short for "pod manager" I believe, is:

>...a daemonless container engine for developing, managing, and running OCI Containers on your Linux System. Containers can either be run as root or in rootless mode. Simply put: `alias docker=podman`. 

Install.

```
sudo dnf install podman
```

This is the version I have:

```
$ rpm -q podman
podman-1.2.0-2.git3bd528e.fc30.x86_64
```

Now, the question is can I use it just like I use docker? 

## The first test: Jekyll

As I mentioned I used to use Docker to create a preview of this blog using Jekyll.

Below is the command I previously used to build a preview site. I've been using this same command for at least a couple years.

```
export JEKYLL_VERSION=3.5
docker run --rm \
  --volume="$PWD:/srv/jekyll" \
  -p 127.0.0.1:4000:4000 \
  -it jekyll/jekyll:pages \
  jekyll serve 
```

That would server up my Jekyll based blog on port 4000, and use the local directory as a volume. Would this command simply work if I replaced Docker with podman?

```
$ podman run --rm \
>   --volume="$PWD:/srv/jekyll" \
>   -p 127.0.0.1:4000:4000 \
>   -it jekyll/jekyll:pages \
>   jekyll serve 
ruby 2.6.3p62 (2019-04-16 revision 67580) [x86_64-linux-musl]
Configuration file: /srv/jekyll/_config.yml
            Source: /srv/jekyll
       Destination: /srv/jekyll/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
     Build Warning: Layout 'nil' requested in atom.xml does not exist.
                    done in 4.247 seconds.
 Auto-regeneration: enabled for '/srv/jekyll'
    Server address: http://0.0.0.0:4000
  Server running... press ctrl-c to stop.
```

I was surprised when this worked without a problem.

Same command to exec in.

```
$ podman exec -it 895a3ca0845c /bin/bash
bash-4.4# ps ax
  PID TTY      STAT   TIME COMMAND
    1 pts/0    Ss+    0:00 /bin/sh /usr/jekyll/bin/jekyll serve
   15 pts/0    Sl+    0:20 ruby -r github-pages /usr/gem/bin/jekyll serve -H 0.0.0.0
   52 pts/1    Ss     0:00 /bin/bash
   58 pts/1    R+     0:00 ps ax
bash-4.4# 
```

How about good old hello-world?

```
$ podman run hello-world
Trying to pull docker.io/library/hello-world...Getting image source signatures
Copying blob 1b930d010525 done
Copying config fce289e99e done
Writing manifest to image destination
Storing signatures

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

It "just works."

## A few notes

Even though podman is command for command the same as docker, there are some major differences, especially in philosophy.

* [podman uses runc](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users/)

>The Podman approach is simply to directly interact with the image registry, with the container and image storage, and with the Linux kernel through the runC container runtime process (not a daemon).

```
$ runc -h | head -6
NAME:
   runc - Open Container Initiative runtime

runc is a command line client for running applications packaged according to
the Open Container Initiative (OCI) format and is a compliant implementation of the
Open Container Initiative specification.
```

* You do not need to be root to run podman
* As mentioned, [same commands](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users/)

>When building Podman, the goal was to make sure that Docker users could easily adapt. So all the commands you are familiar with also exist with Podman. In fact, the claim is made that if you have existing scripts that run Docker you can create a docker alias for podman and all your scripts should work (alias docker=podman).

* "[Podman’s local repository is in /var/lib/containers instead of /var/lib/docker](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users/)"
* "[Podman uses a traditional fork/exec model (vs. a client/server model) for running containers.](https://opensource.com/article/18/10/podman-more-secure-way-run-containers)"
* You won't be able to use docker-compose with podman, that could be an issue for some. There seems to be some [work in making transitioning to podman](https://developers.redhat.com/blog/2019/01/29/podman-kubernetes-yaml/) easier
* [Surprise! podman can manage pods](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/):

>The ability for Podman to handle pod deployment is a clear differentiator to other container runtimes.  As a libpod maintainer, I am still realizing the advantages of having pods even in a localized runtime. There will most certainly be more development in Podman around pods as we learn how users exploit the use of them.

* There is a introductory [course](https://www.katacoda.com/courses/containers-without-docker/running-containers-with-podman) on Katacoda, so you can try it out without even installing


## Conclusion

I like that podman is a command for command replacement for Docker. I also like the focus on security, and the fact that there is no docker server running. Not having docker compose could be problem for developers who have to install things like databases to get a development environment. I like that the community and RedHat have written quite a few blog posts about podman.

Overall, podman, while it could maybe use a better name, is interesting because it moves the container ecosystem forward and provides some diversity. I have no problems with Docker, it's a simple fact that I just wasn't able to install a stable version on Fedora 30 (yet) and thus ended up exploring podman.

I barely touched the tip of the iceberg with the podman ecosystem, in future posts I'll take a more in-depth look. I'm sure there are some edge cases. :)