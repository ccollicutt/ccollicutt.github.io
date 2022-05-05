---
layout: post
title:  "Deploy Tanzu Kubernetes Grid in an Offline/Airgapped Environment"
categories:
header_image: "/img/tkg-offline.jpg"
summary: "Deploy TKG without Internet connectivity"

---

# {{ page.title }}

[Tanzu Kubernetes Grid (TKG)](https://tanzu.vmware.com/kubernetes-grid/) is VMware's distribution of Kubernetes that can be deployed into vSphere and public clouds like Azure and AWS. Sometimes customers prefer that the deployment and management of TKG is done "offline" without needing to obtain any artifacts of the deployment from Internet hosted resources, that the deployment is self-contained.

This is a fairly common requirement, especially in organizations that consider segmentation of resources important, and it's usually done at the network level, i.e. "network segmentation", which has become even more popular as of late, around terms like microsegmentation and zero-trust networking.

## Documentation and Links

* [VMware Tanzu TKG - Internet Restricted Environments](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.5/vmware-tanzu-kubernetes-grid-15/GUID-mgmt-clusters-airgapped-environments.html)
* [Downloading TKG 1.5.3 CLIs](https://customerconnect.vmware.com/en/downloads/details?downloadGroup=TKG-153&productId=988&rPId=88185)

## Offline Environment

Firewall rules:

* tkg-offline-* networks: no internet access, all packets dropped at the edge firewall
* Office network can connect to anything, i.e. the host that copies the container images from VMware to the internal Harbor instance
* Anything on lab switch is available to tkg-offline-*, e.g. vCenter, DNS, Harbor, no firewalling, i.e. the TKG management clusters can talk to vCenter, etc

Hardware, software:

* TKG >= 1.5.3
* 3 ESXI hosts
* Running vSphere 7U2
* Enough resources for TKG
* A Linux + Docker instance to download the container images used in deployment, and to run the tanzu CLI from, as well as certain [Carvel](https://carvel.dev) CLIs

## A Word on Container Images

When we think of container images we think of running an application, like, oh I don't know, nginx. But container images aren't only used for software, they can also be used to store configuration information. That is something that TKG does a lot of. As the industry gets more and more mature around the use of container images, we will do more with them, and that will cause us to simply have more of them. Smaller images, but more of them. My point here is that the number of container images that TKG could use can seem considerable, around 500 or so, but most of them are quite small, on the order of a few megabytes or even kilobytes, and often contain configuration information like BOMs.

In an online environment, one that is connected to the Internet, we probably wouldn't even notice how many images are used by TKG when it's being deployed. However, in an offline environment, one in which we need to copy these images, these artifacts, we have an opportunity to see first hand exactly how many images there are, and it can be surprising to some. I'd say there are about 500 total container images needed for ALL of TKG, and this is to support not only TKG itself, but also EVERY Kuberentes version that we support, which is many. 

## A Word on TLS Certificates

In offline environments organizations almost always also use self-signed certificates, or certificates that are not part of the typical bundle found in operating systems, mostly for browsers. When deploying TKG many container images are pulled from the internal container image registry, in this case Harbor, and that Harbor instance will have a custom TLS certificate.

This means we need to ensure that TLS certificate, or certificate authority, is deployed into the TKG nodes, the virtual machines that underlie the Kubernetes clusters. Along with the various image artifacts, this is a big part of managing the offline deployment.

## Relocating Container Images

One of the things we have to do is relocate the necessary container images from VMware's Internet available registry to the organizations offline registry. This requires some sort of intermediary system, a jumpbox/bastion host or similar.

Currently our official docs provide a couple of scripts to do perform the actual relocation.

First we generate a list of images to relocate.  To run this script we need to set some variables.

```
export TKG_CUSTOM_IMAGE_REPOSITORY="<harbor>/<project>"
export TKG_IMAGE_REPO="projects.registry.vmware.com/tkg"
export TKG_CUSTOM_IMAGE_REPOSITORY_CA_CERTIFICATE=<base64 certificate>
```

I created a project in my Harbor called "tkg-1-5-3".

```
export TKG_CUSTOM_IMAGE_REPOSITORY="<harbor-ip>/tkg-1-5-3"
```

I use mkcert to manage my certificates internally, so for the CA certificate I used that. You might take a different approach, but it's the same idea.

```
base64 -w 0 < /home/curtis/.local/share/mkcert/rootCA.pem
```

The result of that command I put into TKG_CUSTOM_IMAGE_REPOSITORY_CA_CERTIFICATE.

Now I can run the script to generate the list of images to copy to the internal Harbor.

```
$ ./gen-publish-images.sh > image-copy-list
```

That will create this list of images, most of the lines will be an imgpkg command. imgpkg is from the Carvel set of tools.

imgpkg is used to:

>Package, distribute, and relocate your Kubernetes configuration and dependent OCI images as one OCI artifact: a bundle. Consume bundles with confidence that their contents are unchanged after relocation.

```
$ wc -l image-copy-list 
4275 image-copy-list
```

While that seems like a lot of lines, some of the image copy commands are duplicates. So if we sort and uniq them, there are many fewer lines.

```
$ grep imgpkg image-copy-list | sort | uniq | wc -l
568
```

The download image script does filter out the duplicate lines, so don't worry about doing that yourself.

Then we use that list of images via another script to download each image and copy it to the Harbor instance.

>NOTE: I'm using Harbor, but it could be any OCI compliant registry.


```
$ ./download-images.sh image-copy-list
```

Depending on the speed of your internet connection and your environment, this can take a few minutes or a couple hours.

Here's what it looks like in Harbor.

![harbor](/img/tkg-offline-1.jpg)

## Deploying TKG

Now that all the images are copied to the internal container image registry, we can deploy TKG.

First we need to set some configuration variables though. These are the same as we set before for the image copy scripts, but now we're going to set them up for TKG.

```
tanzu config set env.TKG_CUSTOM_IMAGE_REPOSITORY <your harbor>/<tkg project>
tanzu config set env.TKG_CUSTOM_IMAGE_REPOSITORY_SKIP_TLS_VERIFY false
tanzu config set env.TKG_CUSTOM_IMAGE_REPOSITORY_CA_CERTIFICATE <your base64 CA cert>
```

Now validate those are set.

```
$ tanzu config get | grep TKG
```

At this point we can now go through the standard deployment.

Usually I use the install GUI to setup the initial configuration file for the management cluster.

```
$ tanzu mc create --ui --bind 0.0.0.0:8080 --browser none

Validating the pre-requisites...
Serving kickstart UI at http://[::]:8080
```

I connect to this server and fill out the install GUI, and that will generate a randomly named file in "~/.config/tanzu/tkg/clusterconfigs/" and the GUI will give you a command to run from the CLI.

## Conclusion

There are quite a few images to copy, but after that work has been done, and the CA certificate has been properly setup for use, the deployment is straightforward. So, for an offline deployment, it's really two big steps: 

1. Relocate all the images locally, and,
2. Determine what certificate is used in the Harbor instance and make sure that TKG knows about it.

I'm skipping quite a few other steps here, but those steps will be the same in any TKG deployment, offline or not, such as uploading the OVA file, or deploying/obtaining a container image registry like Harbor.