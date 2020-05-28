---
layout: post
title: TUF, Notary, and Harbor Registry
categories:
header_image: "/img/harbor-tuf3.jpg"
header_permalink: "https://unsplash.com/photos/GKVZUKrdkPI"
summary: "Configuring tls secrets and helm values for harbor and custom tls"

---

# {{ page.title }}

If you watch this [video](https://www.youtube.com/watch?v=Hnzc6va4l6k) by Justin Cormack, he starts it off by talking about an attack vector that is used successfully by "evil doers" time and time again...**software updates**. Securing software updates is complicated and can't be solved with simple solutions, and what's worse, when using simple solutions it actually kinda makes things worse. Without some kind of process/framework for managing updates, securing systems is impossible. Sure, securing software updates will require signing things digitally, most technical people get that, but that's just one part of the solution and it's not enough on its own.

That's where [TUF](https://theupdateframework.io/), "The Update Framework," comes in.

>The Update Framework (TUF) helps developers maintain the security of software update systems, providing protection even against attackers that compromise the repository or signing keys. TUF provides a flexible framework and specification that developers can adopt into any software update system.

The [overview](https://github.com/theupdateframework/tuf/blob/develop/docs/OVERVIEW.rst) for TUF is quite good. I don't want to try (and fail) to reproduce that work here. So just go read it. :)

## Harbor

The easiest way to describe Harbor is that it's an image registry. But, with the [recent release of Harbor 2.0](https://goharbor.io/blog/harbor-2.0/), it is much more than just a simple image registry...it's an OCI compliant registry!

>This release makes Harbor the first OCI ( Open Container Initiative)-compliant open source registry capable of storing a multitude of cloud-native artifacts like container images, Helm charts, OPAs, Singularity, and much more.

When Harbor is deployed it can have Notary enabled.

## Notary

Notary is an implementation of TUF, and, like TUF, it's a [CNCF project](https://www.linuxfoundation.org/cloud-containers-virtualization/2017/10/cncf-host-two-security-projects-notary-tuf-specification/).

>Notary is one of the industry’s most mature implementations of the TUF specification and its Go implementation is used today to provide robust security for container image updates, even in the face of a registry compromise. Notary takes care of the operations necessary to create, manage, and distribute the metadata needed to ensure the integrity and freshness of user content. Notary/TUF provides both a client, and a pair of server applications to host signed metadata and perform limited online signing functions.


## Quick Look at Notary and Harbor

I've got a Harbor installation setup to use. I've also installed the notary CLI.

First, I tell Docker to use content trust and point it to the Notary instance that comes with Harbor. In this case Harbor was deployed via the Helm chart, and Notary is exposed on its own hostname via ingress.

```
export DOCKER_CONTENT_TRUST=1
export DOCKER_CONTENT_TRUST_SERVER=https://notary.example.com
```

Next I try to push an image. By doing this, I'll be initializing the keys for use with Notary. I'm glossing over what is happening here, as generating these keys should have a lot of thought put around it, and they should be properly stored and themselves secured. More on this in a future post, I hope.

```
$ docker pull alpine
$ docker tag alpine harbor.example.com/secure/alpine-signed:1.0
$ docker push  harbor.example.com/secure/alpine-signed:1.0
The push refers to repository [harbor.example.com/secure/alpine-signed]
3e207b409db3: Pushed
1.0: digest: sha256:39eda93d15866957feaee28f8fc5adb545276a64147445c64992ef69804dbf01 size: 528
Signing and pushing trust metadata
You are about to create a new root signing key passphrase. This passphrase
will be used to protect the most sensitive key in your signing system. Please
choose a long, complex passphrase and be careful to keep the password and the
key file itself secure and backed up. It is highly recommended that you use a
password manager to generate the passphrase and keep it safe. There will be no
way to recover this key. You can find the key in your config directory.
Enter passphrase for new root key with ID 6b438eb:
Repeat passphrase for new root key with ID 6b438eb:
Enter passphrase for new repository key with ID aafd072:
Repeat passphrase for new repository key with ID aafd072:
Finished initializing "harbor.example.com/secure/alpine-signed"
Successfully signed harbor.example.com/secure/alpine-signed:1.0
```

This first signed image push initializes TUF. Files are created in ~/.docker/trust.

```
$ find ~/.docker/trust/ | head
/home/ubuntu/.docker/trust/
/home/ubuntu/.docker/trust/tuf
/home/ubuntu/.docker/trust/tuf/harbor.example.com
/home/ubuntu/.docker/trust/tuf/harbor.example.com/secure
/home/ubuntu/.docker/trust/tuf/harbor.example.com/secure/alpine-signed
/home/ubuntu/.docker/trust/tuf/harbor.example.com/secure/alpine-signed/changelist
/home/ubuntu/.docker/trust/tuf/harbor.example.com/secure/alpine-signed/metadata
/home/ubuntu/.docker/trust/tuf/harbor.example.com/secure/alpine-signed/metadata/targets.json
/home/ubuntu/.docker/trust/tuf/harbor.example.com/secure/alpine-signed/metadata/root.json
/home/ubuntu/.docker/trust/tuf/docker.io
```

Using the Notary CLI we can see that image is registered.

```
$ alias notary="notary -s https://notary.example.com -d ~/.docker/trust"
$ notary list harbor.example.com/secure/alpine-signed
Enter username: admin
Enter password:
NAME    DIGEST                                                              SIZE (BYTES)    ROLE
----    ------                                                              ------------    ----
1.0     39eda93d15866957feaee28f8fc5adb545276a64147445c64992ef69804dbf01    528             targets
```

In the Harbor GUI the image is show as being signed.

![signed image in harbor](/img/harbor-tuf2.png)

The Harbor project we are pushing the image to has content trust enabled. 

*NOTE: What enabling this means is that Harbor won't allow any images to be pulled from this repo unless they are signed.*

![harbor project content trust enabled](/img/harbor-tuf1.png)

As an example, here I'm trying to pull an **UNSIGNED** image into a k8s cluster, which, when "content trust" is enabled in the project, Harbor will not allow.

```
$ k describe pod alpine-unsigned-1.0  | tail
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                From                                          Message
  ----     ------     ----               ----                                          -------
  Normal   Scheduled  48s                default-scheduler                             Successfully assigned default/alpine-unsigned-1.0 to central-tools-md-0-5bfcdd98d9-2w5kx
  Normal   BackOff    21s (x3 over 47s)  kubelet, central-tools-md-0-5bfcdd98d9-2w5kx  Back-off pulling image "harbor.example.com/secure/alpine-unsigned:1.0"
  Warning  Failed     21s (x3 over 47s)  kubelet, central-tools-md-0-5bfcdd98d9-2w5kx  Error: ImagePullBackOff
  Normal   Pulling    9s (x3 over 47s)   kubelet, central-tools-md-0-5bfcdd98d9-2w5kx  Pulling image "harbor.example.com/secure/alpine-unsigned:1.0"
  Warning  Failed     9s (x3 over 47s)   kubelet, central-tools-md-0-5bfcdd98d9-2w5kx  Failed to pull image "harbor.example.com/secure/alpine-unsigned:1.0": rpc error: code = Unknown desc = failed to pull and unpack image "harbor.example.com/secure/alpine-unsigned:1.0": failed to copy: httpReaderSeeker: failed open: unexpected status code https://harbor.example.com/v2/secure/alpine-unsigned/manifests/sha256:39eda93d15866957feaee28f8fc5adb545276a64147445c64992ef69804dbf01: 412 Precondition Failed - Server message: unknown: The image is not signed in Notary.
  Warning  Failed     9s (x3 over 47s)   kubelet, central-tools-md-0-5bfcdd98d9-2w5kx  Error: ErrImagePull
  ```

  Note the error message Harbor responds with:

  ```
  412 Precondition Failed - Server message: unknown: The image is not signed in Notary.
  ```

  Pulling a signed image is fine, of course. 

## Conclusion

Here's a very basic example of using Notary and content trust in Harbor. To put this process into production would take a fair amount of consideration and should not be taken lightly. But if you want to quickly try out signing an image in combination with using content trust with Harbor, then it's quite simple to do. Deploying Harbor with Helm is also [straight forward](https://www.linuxfoundation.org/cloud-containers-virtualization/2017/10/cncf-host-two-security-projects-notary-tuf-specification/) if you have a k8s cluster to use. :)