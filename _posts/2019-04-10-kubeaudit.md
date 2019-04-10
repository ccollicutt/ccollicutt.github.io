---
layout: post
title: Kubeaudit
categories:

---

# {{ page.title }}


I wanted to call this article "Gleaming the Kube" but [someone](http://dougbtv.com/nfvpe/2017/05/12/kubernetes-from-source/) already did that.

Kubernetes is a relatively complex system, and I typically deal with OpenStack so that is saying something. With any new and complex (and valuable) system it's easy to use that system to build, shall we say, services with less than desirable security settings. And thus we have people building tools that help to ensure systems and definitions are as secure as is reasonably possible. Enter kubeaudit.

## Kubeaudit

[Kubeaudit](https://github.com/Shopify/kubeaudit) is a helpful tool from the folks at Shopify. A Canadian company by the way!

>kubeaudit is a command line tool to audit Kubernetes clusters for various different security concerns: run the container as a non-root user, use a read only root filesystem, drop scary capabilities, don't add new ones, don't run privileged, ... You get the gist of it and more on that later. Just know: **kubeaudit makes sure you deploy secure containers!**

Because go compiles into a nice clean binary, it's easy to install kubeaudit.

```
wget -P /tmp https://github.com/Shopify/kubeaudit/releases/download/v0.5.3/kubeaudit_0.5.3_linux_amd64.tar.gz
mkdir ~/bin # if you don't have one
export PATH=$PATH:~/bin
# only untar kubeaudit, there is a licence and readme file in there as well which we don't need
tar zxvf /tmp/kubeaudit_0.5.3_linux_amd64.tar.gz -C ~/bin kubeaudit
```

Check version.

```
$ kubeaudit version
INFO[0000] Kubeaudit version                             BuildDate="2019-03-29T15:36:37Z" Commit=b19f6509d92abc22a8cf2789a98a740af20831e6 Version=0.5.3
INFO[0000] Not running inside cluster, using local config 
INFO[0000] Kubernetes server version                     Major=1 Minor=13 Platform=linux/amd64
```

Note that this will check the Kubernetes version as well through reading the `.kube/config`.


## Audit

Let's deploy an nginx container.

*NOTE: I alias kubectl to k*

```
$ k run nginx --image=nginxinc/nginx-unprivileged --port=8080
```

It's up and running.

```
$ k get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6c6d45d55d-ckhzw   1/1     Running   0          51s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           51s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6c6d45d55d   1         1         1       51s
```

Now audit it with `kubeaudit`, but we'll only check for nonroot.

```
$ kubeaudit nonroot -n nginx-unpriv
INFO[0000] Not running inside cluster, using local config 
ERRO[0000] RunAsNonRoot is not set in ContainerSecurityContext, which results in root user being allowed!  Container=nginx KubeType=deployment Name=nginx Namespace=nginx-unpriv
ERRO[0000] RunAsNonRoot is not set in ContainerSecurityContext, which results in root user being allowed!  Container=nginx KubeType=pod Name=nginx-6c6d45d55d-ckhzw Namespace=nginx-unpriv
```

Ok let's fix that by patching the deployment.

Here is the patch:

```
$ cat nginx-unpriv-runasnonroot.yml
spec:
  template:
    spec:
      containers:
      - image: nginxinc/nginx-unprivileged
        name: nginx
        securityContext:
          runAsNonRoot: true
      securityContext:
        runAsNonRoot: true
```

Apply the patch.

```
$ kubectl patch deployment nginx --patch "$(cat nginx-unpriv-runasnonroot.yml)"
deployment.extensions/nginx patched
```

A new pod should be created.

```
~$ k get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-686cc9b75c-jfc8t   1/1     Running   0          24s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           3m50s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-686cc9b75c   1         1         1       24s
replicaset.apps/nginx-6c6d45d55d   0         0         0       3m50s
```

And now to run audit again:

```
$ kubeaudit nonroot -n nginx-unpriv
INFO[0000] Not running inside cluster, using local config 
```

No errors.

That's it. A simple example of using `kubeaudit`.