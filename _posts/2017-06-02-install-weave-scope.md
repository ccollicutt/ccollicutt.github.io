---
layout: post
title: Installing Weave Scope into Kubernetes
categories:
header_image: /img/weave-scope-1.jpg
---

# {{ page.title }}

In a [previous post](/2017/06/02/kubeadm-openstack.html) I installed a k8s cluster using kubeadm.

Now I would like to add the [Weave Scope](https://www.weave.works/oss/scope/) application to the cluster so that I can visualize what is going on. I should note that I first saw Scope used in a presentation regarding [OpenStack Helm](https://www.youtube.com/watch?v=9-EgvlJ0dvY). (Yeah, that's right, using k8s to manage the OpenStack control plane.)

Weave Scope allows you to:

>See your Docker hosts, containers and services in real time. Easily identify and correct issues to ensure the stability and performance of your containerized applications.

## Installation

Basically it's just a matter of getting the command correct. There are [official docs](https://www.weave.works/docs/scope/latest/installing/#k8s) on how to install Weave Scope.

```
$ kubectl --kubeconfig ./admin.conf apply --namespace kube-system -f "https://cloud.weave.works/k8s/scope.yaml?k8s-service-type=NodePort&k8s-version=$(kubectl --kubeconfig ./admin.conf version | base64 | tr -d '\n')"
```

Note that I added the `k8s-service-type=NodePort`.

After a few seconds we can validate the deployment:

```
root@k8s-1:/etc/kubernetes# kubectl --kubeconfig ./admin.conf -n kube-system get svc
NAME              CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
kube-dns          10.96.0.10      <none>        53/UDP,53/TCP   2h
weave-scope-app   10.111.34.232   <nodes>       80:31863/TCP    1h
```

Above we can see what port it's on, in this case `31863`. Note that I'm not exporting this service in any fashion right now, to use Scope I'm connecting my browser directly to node:31863.

## sock-shop

In the previous post I deployed `sock-shop`.

Here's a view from Weave Scope of the `sock-shop` namespace.

![sock shop weave](/img/weave-scope-2.jpg)

## More Images

There are all kinds of handy tools.

![sock shop weave](/img/weave-scope-3.jpg)

The Internet.

![sock shop weave](/img/weave-scope-4.jpg)

Processes.

![sock shop weave](/img/weave-scope-5.jpg)

I'm leaving a lot out.

## Conclusion

I usually don't reach for visualization tools, but with k8s it was helpful to get a picture of what's going on. Plus, it's so simple to install. I really like how services like this are installed into the kube-system namespace, a form of self-hosting. It's a great model.
