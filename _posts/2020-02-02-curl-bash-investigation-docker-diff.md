---
layout: post
title: Investigating Curl Bash Installs with Docker
categories:
header_image: "/img/glitch-docker.jpg"
header_permalink: "https://unsplash.com/photos/YeUVDKZWSZ4"
summary: "What does curl http://somesite.com/bash.php | bash do to your system?"

---

# {{ page.title }}

Many sytems install with some version of `curl https://install.example.com | bash`. Using this model for installation is typically DIScouraged. But what if you don't have much of an option for installation? How can you investigate what this bash script does?

One simple way is to use docker diff.

## sdkman Install Inspection

Recently I have started working with Java. Pretty new to it all. I'm used to basic Python development and often use virtual environments. What provides a similar pattern in the Java ecosystem? One example may be sdkman. I'm not sure, haven't used it. However, to install it requires the curl bash (anti?) pattern. I'd like to investigate what that curl bash does.

First, an image with the base requirements for sdkman. It uses which to check for zip, ect.

```
localhost$ cat Dockerfile 
FROM fedora
RUN dnf install -y unzip zip curl which
```

```
localhost$ docker build . -t sdkman-test
```

Run it.

```
docker run -it --name sdkman-test-run sdkman-test /bin/bash
```

Manually install sdkman into that container through the curl bash.

```
container$ curl -s "https://get.sdkman.io" | bash 
```

Inspect the differences in the filesystem in the RUNNING container.

```
localhost$ docker diff sdkman-test-run
C /root
C /root/.bashrc
A /root/.sdkman
A /root/.sdkman/ext
A /root/.sdkman/src
A /root/.sdkman/src/sdkman-offline.sh
A /root/.sdkman/src/sdkman-selfupdate.sh
A /root/.sdkman/src/sdkman-uninstall.sh
A /root/.sdkman/src/sdkman-use.sh
A /root/.sdkman/src/sdkman-availability.sh
A /root/.sdkman/src/sdkman-env-helpers.sh
A /root/.sdkman/src/sdkman-install.sh
A /root/.sdkman/src/sdkman-list.sh
A /root/.sdkman/src/sdkman-update.sh
A /root/.sdkman/src/sdkman-version.sh
A /root/.sdkman/src/sdkman-current.sh
A /root/.sdkman/src/sdkman-default.sh
A /root/.sdkman/src/sdkman-help.sh
A /root/.sdkman/src/sdkman-main.sh
A /root/.sdkman/src/sdkman-broadcast.sh
A /root/.sdkman/src/sdkman-flush.sh
A /root/.sdkman/src/sdkman-upgrade.sh
A /root/.sdkman/src/sdkman-cache.sh
A /root/.sdkman/src/sdkman-path-helpers.sh
A /root/.sdkman/src/sdkman-utils.sh
A /root/.sdkman/tmp
A /root/.sdkman/tmp/sdkman-5.7.4+362.zip
A /root/.sdkman/tmp/stage
A /root/.sdkman/var
A /root/.sdkman/var/candidates
A /root/.sdkman/var/version
A /root/.sdkman/archives
A /root/.sdkman/bin
A /root/.sdkman/bin/sdkman-init.sh
A /root/.sdkman/candidates
A /root/.sdkman/etc
A /root/.sdkman/etc/config
A /root/.zshrc
```

Seems like /root/.bashrc has changed. Unfortunately docker diff can't diff individual files. So I'll need to build a new image and diff the base Fedora file.

Let's add the curl bash to the image so we can diff it.

```
localhost$ cat Dockerfile 
FROM fedora
RUN dnf install -y unzip zip curl which
RUN curl -s "https://get.sdkman.io" | bash 
```

Build again.

```
localhost$ docker build . -t sdkman-test
```

Now grab the two bashrc file, one from the base fedora image and one from the image just built.

```
docker run --rm -v `pwd`:/out sdkman-test /bin/cp /root/.bashrc /out/bashrc-sdkman
docker run --rm -v `pwd`:/out fedora /bin/cp /root/.bashrc /out/bashrc-base
```

Diff.

```
localhost$ diff bashrc-base bashrc-sdkman 
12a13,16
> 
> #THIS MUST BE AT THE END OF THE FILE FOR SDKMAN TO WORK!!!
> export SDKMAN_DIR="/root/.sdkman"
> [[ -s "/root/.sdkman/bin/sdkman-init.sh" ]] && source "/root/.sdkman/bin/sdkman-init.sh"
```

Not surprisingly the install has added sourcing an sdkman init script to `~/.bashrc`.

## Conclusion

Obviously there is no limit to the amount of inspection one could do on foreign code, nor the amount of changes an imported bash file could do to an OS. In this case all I've done is find out what files sdkman adds or changes, and review the diffs on one file. But at least it gives me an idea of what is going on with the install.

Using docker to evaluate tools makes a lot of sense to me, mostly because I don't want to mess up my physical OS (which is actually running in a virtual machine, but I digress). Some people take this [much farther](https://github.com/jessfraz/dockerfiles) and use containers to run things like sdkman.

Even though I've worked a lot with Docker, I've never tried to inspect the diffs of image layers. Likely that would be a better option that what I've done above. Something to look into...