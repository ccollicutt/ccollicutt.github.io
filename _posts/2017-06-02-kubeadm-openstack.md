---
layout: post
title: Installing Kubernetes with Kubeadm
categories:
header_image: /img/boat.jpg
header_permalink: http://www.metmuseum.org/art/collection/search/11122
---

# {{ page.title }}

I have done a good amount of work with Kubernetes in the last year or so. I created a fairly substantial set of Ansible playbooks and workflows, called [Sk8ts](https://github.com/sk8ts), which would deploy Kubernetes to AWS. It would create networks, gateways, instances, setup clusters, etc. But to be honest I only went about 85% as far as I should have and ran out of time to spend on it. Further, perhaps creating my own distribution is not a great idea, though I certainly learned a lot about Ansible, AWS, and k8s.

I mention my work in Sk8ts because it was essentially a 3rd party installer or distribution. I also need to add to the context of this post the fact that I have spent years working on OpenStack, which does not have a specific, project led installer, and some people consider this to be a problem. Whether or not large, complicated infrastructure systems like OpenStack and Kubernetes have official installers is a bit of a conundrum.

While OpenStack does not have an official installer, Kubernetes does: [Kubedadm](https://kubernetes.io/docs/admin/kubeadm/). So in this post I will look at deploying Kubernetes 1.6 with Kubeadm. Please note that Kubeadm is not production ready yet. But someday...

## Create hosts

K8s needs somewhere to run. I have an OpenStack cloud that I can create networks and instances in.

I've created four nodes to deploy k8s to. I've done this a few times so I kept the command around. For reference it's below. I'll use the first node as the master and the other three as the workers. The `m1.medium` flavor just has 4GB of memory, so they are not that large resource-wise.

```
#!/bin/bash

NET=cee24724-e062-4370-ba9f-57bed80f32cd

openstack server create \
--image xenial \
--key-name curtis \
--flavor m1.medium \
--min 4 \
--max 4 \
--nic net-id=$NET \
k8s
```

Just note that that will boot four instances. :)

## Setup Docker

Once the instances have been created, we can install the k8s and docker packages.

```
$ os server list
+--------------------------------------+-------+--------+---------------------------------------------------+------------+
| ID                                   | Name  | Status | Networks                                          | Image Name |
+--------------------------------------+-------+--------+---------------------------------------------------+------------+
| 5da0a8b9-9635-47ba-b381-f3f10b569523 | k8s-4 | ACTIVE | k8s-vxlan=10.50.0.16                             | xenial     |
| b033b2f6-b7b1-4f62-81c6-cc486320880a | k8s-3 | ACTIVE | k8s-vxlan=10.50.0.13                             | xenial     |
| 9a4f75d9-20ba-4be0-8daf-7b9a5b6ae289 | k8s-2 | ACTIVE | k8s-vxlan=10.50.0.17                             | xenial     |
| edfccc19-98da-463a-b0d4-a779ff19e12a | k8s-1 | ACTIVE | k8s-vxlan=10.50.0.11                             | xenial     |
+--------------------------------------+-------+--------+---------------------------------------------------+------------+
```

Above are the four k8s-x instances. Now I'll ssh into k8s-1 and install the k8s and docker packages. To do the install I'll just a script.

```
ubuntu@k8s-1:~$ cat kube-install.sh
#!/bin/bash
apt-get update
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

apt-add-repository \
   "deb http://apt.kubernetes.io/ kubernetes-xenial main"
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
apt-get update
apt-get install docker-ce -y --allow-unauthenticated
apt-get install -y kubelet kubeadm kubectl kubernetes-cni --allow-unauthenticated
```

That will *insecurely* install various packages. I'm not getting any GPG keys, etc.

```
ubuntu@k8s-1:~$ sudo bash kube-install.sh
sudo: unable to resolve host k8s-1
Get:1 http://security.ubuntu.com/ubuntu xenial-security InRelease [102 kB]
SNIP!
Setting up docker-ce (17.03.1~ce-0~ubuntu-xenial) ...
Processing triggers for libc-bin (2.23-0ubuntu7) ...
Processing triggers for systemd (229-4ubuntu17) ...
Processing triggers for ureadahead (0.100.0-19) ...
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  ebtables socat
The following NEW packages will be installed:
  ebtables kubeadm kubectl kubelet kubernetes-cni socat
0 upgraded, 6 newly installed, 0 to remove and 10 not upgraded.
Need to get 43.2 MB of archives.
After this operation, 323 MB of additional disk space will be used.
WARNING: The following packages cannot be authenticated!
  kubernetes-cni kubelet kubectl kubeadm
Authentication warning overridden.
Get:5 http://nova.clouds.archive.ubuntu.com/ubuntu xenial/main amd64 ebtables amd64 2.0.10.4-3.4ubuntu1 [79.6 kB]
Get:6 http://nova.clouds.archive.ubuntu.com/ubuntu xenial/universe amd64 socat amd64 1.7.3.1-1 [321 kB]
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubernetes-cni amd64 0.5.1-00 [5,560 kB]
Get:2 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.6.4-00 [18.3 MB]
Get:3 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.6.4-00 [9,659 kB]
Get:4 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.6.4-00 [9,234 kB]
Fetched 43.2 MB in 4s (10.4 MB/s)    
Selecting previously unselected package ebtables.
(Reading database ... 54160 files and directories currently installed.)
Preparing to unpack .../ebtables_2.0.10.4-3.4ubuntu1_amd64.deb ...
Unpacking ebtables (2.0.10.4-3.4ubuntu1) ...
Selecting previously unselected package kubernetes-cni.
Preparing to unpack .../kubernetes-cni_0.5.1-00_amd64.deb ...
Unpacking kubernetes-cni (0.5.1-00) ...
Selecting previously unselected package socat.
Preparing to unpack .../socat_1.7.3.1-1_amd64.deb ...
Unpacking socat (1.7.3.1-1) ...
Selecting previously unselected package kubelet.
Preparing to unpack .../kubelet_1.6.4-00_amd64.deb ...
Unpacking kubelet (1.6.4-00) ...
Selecting previously unselected package kubectl.
Preparing to unpack .../kubectl_1.6.4-00_amd64.deb ...
Unpacking kubectl (1.6.4-00) ...
Selecting previously unselected package kubeadm.
Preparing to unpack .../kubeadm_1.6.4-00_amd64.deb ...
Unpacking kubeadm (1.6.4-00) ...
Processing triggers for systemd (229-4ubuntu17) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for man-db (2.7.5-1) ...
Setting up ebtables (2.0.10.4-3.4ubuntu1) ...
update-rc.d: warning: start and stop actions are no longer supported; falling back to defaults
Setting up kubernetes-cni (0.5.1-00) ...
Setting up socat (1.7.3.1-1) ...
Setting up kubelet (1.6.4-00) ...
Setting up kubectl (1.6.4-00) ...
Setting up kubeadm (1.6.4-00) ...
Processing triggers for systemd (229-4ubuntu17) ...
Processing triggers for ureadahead (0.100.0-19) ...
```

Nice. Now we have all the k8s packages and Docker installed. I should note that the Docker version we are getting is perhaps not supported by k8s. I believe k8s is only validated on Docker 1.11 or 1.12. Frankly I'm not sure how to get that version any more, as Docker has split out into a community and enterprise versions. The k8s install does seem to work with this version though.

```
ubuntu@k8s-1:~$ dpkg --list docker-ce
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                  Version                 Architecture            Description
+++-=====================================-=======================-=======================-===============================================================================
ii  docker-ce                             17.03.1~ce-0~ubuntu-xen amd64                   Docker: the open-source application container engine
```

So I'm getting 17.02-1-ce...??? Honestly, I don't know what that version means.

## Installing the k8s master

Now I can use kubeadm.

```
ubuntu@k8s-1:~$ sudo kubeadm init
sudo: unable to resolve host k8s-1
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[init] Using Kubernetes version: v1.6.4
[init] Using Authorization mode: RBAC
[preflight] Running pre-flight checks
[preflight] WARNING: docker version is greater than the most recently validated version. Docker version: 17.03.1-ce. Max validated version: 1.12
[preflight] WARNING: hostname "k8s-1" could not be reached
[preflight] WARNING: hostname "k8s-1" lookup k8s-1 on 10.50.0.1:53: no such host
[certificates] Generated CA certificate and key.
[certificates] Generated API server certificate and key.
[certificates] API Server serving cert is signed for DNS names [k8s-1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.50.0.11]
[certificates] Generated API server kubelet client certificate and key.
[certificates] Generated service account token signing key and public key.
[certificates] Generated front-proxy CA certificate and key.
[certificates] Generated front-proxy client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
[apiclient] Created API client, waiting for the control plane to become ready
[apiclient] All control plane components are healthy after 23.025086 seconds
[apiclient] Waiting for at least one node to register
[apiclient] First node has registered after 4.505916 seconds
[token] Using token: bdc910.dac015f93ad5a064
[apiconfig] Created RBAC rules
[addons] Created essential addon: kube-proxy
[addons] Created essential addon: kube-dns

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  sudo cp /etc/kubernetes/admin.conf $HOME/
  sudo chown $(id -u):$(id -g) $HOME/admin.conf
  export KUBECONFIG=$HOME/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token bdc910.dac015f93ad5a064 10.50.0.11:6443
```

There are a bunch of containers running.

```
ubuntu@k8s-1:~$ sudo docker ps
sudo: unable to resolve host k8s-1
CONTAINER ID        IMAGE                                                                                                                            COMMAND                  CREATED             STATUS              PORTS               NAMES
bf36a19d1d61        gcr.io/google_containers/kube-proxy-amd64@sha256:44cc08e7e8a2089eb8dfad6b692e9ece5994d6e6cff07fc9e9b1273cab0f6c6a                "/usr/local/bin/ku..."   2 minutes ago       Up 2 minutes                            k8s_kube-proxy_kube-proxy-jvdkl_kube-system_fbc037b7-4864-11e7-acb2-fa163ef42293_0
9bda7bb1a3f2        gcr.io/google_containers/pause-amd64:3.0                                                                                         "/pause"                 2 minutes ago       Up 2 minutes                            k8s_POD_kube-proxy-jvdkl_kube-system_fbc037b7-4864-11e7-acb2-fa163ef42293_0
d5a926f598ef        gcr.io/google_containers/kube-scheduler-amd64@sha256:57661c79890b01ef2ff183ed4b467ca470efc4fb8d0517cd29abe49e72f6d904            "kube-scheduler --..."   2 minutes ago       Up 2 minutes                            k8s_kube-scheduler_kube-scheduler-k8s-1_kube-system_3145edd89dab0492bdacc0dd589d0e90_0
95faeb5d116b        gcr.io/google_containers/kube-controller-manager-amd64@sha256:a93d4c26d71de94861f78cf5ea62600e4952685d580e2774c630ea206b7c18ee   "kube-controller-m..."   2 minutes ago       Up 2 minutes                            k8s_kube-controller-manager_kube-controller-manager-k8s-1_kube-system_8d185204c4cf91dd9e76230d0642391b_0
fc4c977e5061        gcr.io/google_containers/etcd-amd64@sha256:d83d3545e06fb035db8512e33bd44afb55dea007a3abd7b17742d3ac6d235940                      "etcd --listen-cli..."   2 minutes ago       Up 2 minutes                            k8s_etcd_etcd-k8s-1_kube-system_7075157cfd4524dbe0951e00a8e3129e_0
c3d248897b53        gcr.io/google_containers/kube-apiserver-amd64@sha256:6d5aa429c2b0806e4b6d1d179054d6deee46eec0aabe7bd7bd6abff97be36ae7            "kube-apiserver --..."   2 minutes ago       Up 2 minutes                            k8s_kube-apiserver_kube-apiserver-k8s-1_kube-system_76f5cdc7dab34e6c8b32d96a42cc51e8_0
8482b6284833        gcr.io/google_containers/pause-amd64:3.0                                                                                         "/pause"                 2 minutes ago       Up 2 minutes                            k8s_POD_kube-scheduler-k8s-1_kube-system_3145edd89dab0492bdacc0dd589d0e90_0
4016d11d968d        gcr.io/google_containers/pause-amd64:3.0                                                                                         "/pause"                 2 minutes ago       Up 2 minutes                            k8s_POD_kube-controller-manager-k8s-1_kube-system_8d185204c4cf91dd9e76230d0642391b_0
ebc0ef82e638        gcr.io/google_containers/pause-amd64:3.0                                                                                         "/pause"                 2 minutes ago       Up 2 minutes                            k8s_POD_kube-apiserver-k8s-1_kube-system_76f5cdc7dab34e6c8b32d96a42cc51e8_0
045d7c8d75ba        gcr.io/google_containers/pause-amd64:3.0                                                                                         "/pause"                 2 minutes ago       Up 2 minutes                            k8s_POD_etcd-k8s-1_kube-system_7075157cfd4524dbe0951e00a8e3129e_0
```

## Install Networking Plugin

Now we need a networking plugin. By default kubeadm is ready to use weave. This is amazingly simple.

```
root@k8s-1:/etc/kubernetes# kubectl --kubeconfig ./admin.conf apply -f https://git.io/weave-kube-1.6
clusterrole "weave-net" created
serviceaccount "weave-net" created
clusterrolebinding "weave-net" created
daemonset "weave-net" created
```

This will modify the networking on the host.

```
root@k8s-1:/etc/kubernetes# ip ad sh
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:f4:22:93 brd ff:ff:ff:ff:ff:ff
    inet 10.50.0.11/24 brd 10.50.0.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fef4:2293/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:59:52:32:1d brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
4: datapath: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UNKNOWN group default qlen 1
    link/ether a2:29:39:a0:df:49 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::a029:39ff:fea0:df49/64 scope link
       valid_lft forever preferred_lft forever
6: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP group default qlen 1000
    link/ether 9a:80:36:0d:7c:64 brd ff:ff:ff:ff:ff:ff
    inet 10.32.0.1/12 scope global weave
       valid_lft forever preferred_lft forever
    inet6 fe80::9880:36ff:fe0d:7c64/64 scope link
       valid_lft forever preferred_lft forever
7: dummy0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether ee:ec:e0:cc:a1:9e brd ff:ff:ff:ff:ff:ff
9: vethwe-datapath@vethwe-bridge: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master datapath state UP group default qlen 1000
    link/ether 2e:9b:d3:2f:66:21 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::2c9b:d3ff:fe2f:6621/64 scope link
       valid_lft forever preferred_lft forever
10: vethwe-bridge@vethwe-datapath: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP group default qlen 1000
    link/ether 9e:1d:61:4f:c1:71 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::9c1d:61ff:fe4f:c171/64 scope link
       valid_lft forever preferred_lft forever
11: vxlan-6784: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65485 qdisc noqueue master datapath state UNKNOWN group default qlen 1000
    link/ether 8e:12:6f:d6:0c:1d brd ff:ff:ff:ff:ff:ff
    inet6 fe80::8c12:6fff:fed6:c1d/64 scope link
       valid_lft forever preferred_lft forever
13: vethweplc205ec0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue master weave state UP group default
    link/ether 62:6f:d0:66:4a:2b brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::606f:d0ff:fe66:4a2b/64 scope link
       valid_lft forever preferred_lft forever
```

Note the weave components.

There are also weave containers created.

```
root@k8s-1:/etc/kubernetes# docker ps | grep weave
fa0eaddf9b6e        weaveworks/weave-npc@sha256:d4b37edd345b42fdc4cd4fdc9398233db035916c7ad04f2a99fb8230b1d2f6e9                                     "/usr/bin/weave-npc"     About a minute ago   Up About a minute                       k8s_weave-npc_weave-net-8n654_kube-system_889073fd-4865-11e7-acb2-fa163ef42293_0
f3e22468fc86        weaveworks/weave-kube@sha256:0445da5b752a50133133e2d4d6383e622f4a06a3c744268740238c23ae05c594                                    "/home/weave/launc..."   About a minute ago   Up About a minute                       k8s_weave_weave-net-8n654_kube-system_889073fd-4865-11e7-acb2-fa163ef42293_0
3953f0b070dd        gcr.io/google_containers/pause-amd64:3.0                                                                                         "/pause"                 About a minute ago   Up About a minute                       k8s_POD_weave-net-8n654_kube-system_889073fd-4865-11e7-acb2-fa163ef42293_0
```

## Add K8s workers

I'll ssh into the other nodes and install the k8s and docker packages.

```
ubuntu@k8s-2:~$ sudo bash kube-install.sh
SNIP!
```

Now they can join via kubeadm.

```
ubuntu@k8s-2:~$ sudo   kubeadm join --token bdc910.dac015f93ad5a064 10.50.0.11:6443
sudo: unable to resolve host k8s-2
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[preflight] Running pre-flight checks
[preflight] WARNING: docker version is greater than the most recently validated version. Docker version: 17.03.1-ce. Max validated version: 1.12
[preflight] WARNING: hostname "k8s-2" could not be reached
[preflight] WARNING: hostname "k8s-2" lookup k8s-2 on 10.50.0.1:53: no such host
[discovery] Trying to connect to API Server "10.50.0.11:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://10.50.0.11:6443"
[discovery] Cluster info signature and contents are valid, will use API Server "https://10.50.0.11:6443"
[discovery] Successfully established connection with API Server "10.50.0.11:6443"
[bootstrap] Detected server version: v1.6.4
[bootstrap] The server supports the Certificates API (certificates.k8s.io/v1beta1)
[csr] Created API client to obtain unique certificate for this node, generating keys and certificate signing request
[csr] Received signed certificate from the API server, generating KubeConfig...
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"

Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on the master to see this machine join.
```

We can see there are two nodes now.

```
root@k8s-1:/etc/kubernetes# kubectl --kubeconfig ./admin.conf get nodes
NAME      STATUS    AGE       VERSION
k8s-1     Ready     16m       v1.6.4
k8s-2     Ready     5m        v1.6.4
```

Now I'll add the other nodes.

```
root@k8s-1:/etc/kubernetes# kubectl --kubeconfig ./admin.conf get nodes
NAME      STATUS     AGE       VERSION
k8s-1     Ready      21m       v1.6.4
k8s-2     Ready      10m       v1.6.4
k8s-3     Ready      1m        v1.6.4
k8s-4     NotReady   7s        v1.6.4
```

Great, now we have a k8s cluster of four nodes that was deployed by kubeadm.

## Deploy sock-shop

So how do we know this is even working? Lets deploy the socks shop app.

```
root@k8s-1:/etc/kubernetes# kubectl --kubeconfig ./admin.conf create namespace sock-shop
namespace "sock-shop" created
root@k8s-1:/etc/kubernetes# kubectl --kubeconfig ./admin.conf apply -n sock-shop -f "https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true"
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
namespace "sock-shop" configured
namespace "zipkin" created
deployment "carts-db" created
service "carts-db" created
deployment "carts" created
service "carts" created
deployment "catalogue-db" created
service "catalogue-db" created
deployment "catalogue" created
service "catalogue" created
deployment "front-end" created
service "front-end" created
deployment "orders-db" created
service "orders-db" created
deployment "orders" created
service "orders" created
deployment "payment" created
service "payment" created
deployment "queue-master" created
service "queue-master" created
deployment "rabbitmq" created
service "rabbitmq" created
deployment "shipping" created
service "shipping" created
deployment "user-db" created
service "user-db" created
deployment "user" created
service "user" created
the namespace from the provided object "zipkin" does not match the namespace "sock-shop". You must pass '--namespace=zipkin' to perform this operation.
the namespace from the provided object "zipkin" does not match the namespace "sock-shop". You must pass '--namespace=zipkin' to perform this operation.
the namespace from the provided object "zipkin" does not match the namespace "sock-shop". You must pass '--namespace=zipkin' to perform this operation.
the namespace from the provided object "zipkin" does not match the namespace "sock-shop". You must pass '--namespace=zipkin' to perform this operation.
the namespace from the provided object "zipkin" does not match the namespace "sock-shop". You must pass '--namespace=zipkin' to perform this operation.
```

This might take a while to complete in terms of downloading docker images and such.

We can ask for the port information.

```
root@k8s-1:/etc/kubernetes# kubectl --kubeconfig ./admin.conf -n sock-shop get svc front-end
NAME        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
front-end   10.96.97.28   <nodes>       80:30001/TCP   55s
```

We can access the socks shop page...

```
root@k8s-1:/etc/kubernetes# curl localhost:30001 | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html lang="en">

<head>

    <meta charset="utf-8">
    <meta name="robots" content="all,follow">
    <meta name="googlebot" content="index,follow,snippet,archive">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="WeaveSocks Demo App">
100  8688  100  8688    0     0   314k      0 --:--:-- --:--:-- --:--:--  326k
curl: (23) Failed writing body (248 != 744)
```

## Issues

* Initially I tried installing using kubeadm from behind an http proxy, but that brought all kinds of issues, so I gave up.
* As mentioned, perhaps should be installing docker 1.12.
* Not clear on the zipkin issue with socks-shop
* I am confused with regards to how to setup access to deployed applications. With AWS it was straightfoward, configured K8s to create AWS loadbalancers. But in this situation, I'm not sure...yet. :)

## Conclusion

k8s has changed a lot since I was using it in version 1.4. I'm quite behind. :)

I'm curious to see if kubeadm will catch on and actually be the best way to deploy and manage k8s. There are many other (competing?) projects.

I was inspired to try kubeadm by [this heptio blog post](https://blog.heptio.com/why-heptio-is-a-kubernetes-company-not-a-kubernetes-distribution-df35bb0a559f) in which they discuss how they don't want to be k8s distribution.

>...we need to be careful: distributions can be a dangerous path for a community. Each distributor has strong incentives to deliver differentiated experiences, and differentiated capabilities. As they develop a customer following their customers clamor for features. The community cannot move as fast as they could and so they deliver a patch. And somewhere a fairy dies. The community gets fragmented one really great customer request at a time. You end up with semantic divergence, and a community ‘dark ages’ period happens until a conquering empire emerges to pull it all together.

I don't know if kubeadm can deploy k8s in a way that every single organization will be happy with. But we shall see.

At the very least, it's an easy way to get a test/dev k8s install.
