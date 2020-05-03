---
layout: post
title: Tanzu Kubernetes Grid and Antrea
categories:
header_image: "/img/antrea.jpg"
header_permalink: "https://unsplash.com/photos/TnUelBKIDfM"
summary: "Antrea is a Kubernetes networking solution intended to be Kubernetes native. It leverages Open vSwitch as the networking data plane."

---

# {{ page.title }}

In this post I'll briefly discuss Tanzu Kubernetes Grid, and then get a bit into the Antrea Container Network Interface (CNI) for Kubernetes.

## Tanzu Kubernetes Grid

[Tanzu Kubernetes Grid](https://tanzu.vmware.com/kubernetes-grid), otherwise known as TKG, is the underlying Kuberenetes distribution for all of VMware's Kuberenete based products. It's part of vSphere with Kubernetes. It's part of TKGI (what was once PKS). But it's not just a distribution...it's also a standalone lifecycle manager that heavily utilizes [Cluster API](https://github.com/kubernetes-sigs/cluster-api) to manage virtual machines on which Kubernetes runs.

I'm not going to get into how TKG works, other than to say one of the first things you do with TKG is to deploy a management cluster. That cluster is then used to manage the life cycle of many other k8s "workload" clusters.

Here's all the CRDs that are part of the management cluster.

```
$ k get crds | grep "machine/|vsphere\|cluster"
clusterinformations.crd.projectcalico.org                 2020-04-21T19:03:46Z
clusterissuers.cert-manager.io                            2020-04-21T19:03:48Z
clusters.cluster.x-k8s.io                                 2020-04-21T19:05:35Z
haproxyloadbalancers.infrastructure.cluster.x-k8s.io      2020-04-21T19:05:46Z
kubeadmconfigs.bootstrap.cluster.x-k8s.io                 2020-04-21T19:05:39Z
kubeadmconfigtemplates.bootstrap.cluster.x-k8s.io         2020-04-21T19:05:39Z
kubeadmcontrolplanes.controlplane.cluster.x-k8s.io        2020-04-21T19:05:43Z
machinedeployments.cluster.x-k8s.io                       2020-04-21T19:05:35Z
machinehealthchecks.cluster.x-k8s.io                      2020-04-21T19:05:35Z
machinepools.exp.cluster.x-k8s.io                         2020-04-21T19:05:36Z
machines.cluster.x-k8s.io                                 2020-04-21T19:05:36Z
machinesets.cluster.x-k8s.io                              2020-04-21T19:05:36Z
providers.clusterctl.cluster.x-k8s.io                     2020-04-21T19:03:46Z
vsphereclusters.infrastructure.cluster.x-k8s.io           2020-04-21T19:05:46Z
vspheremachines.infrastructure.cluster.x-k8s.io           2020-04-21T19:05:46Z
vspheremachinetemplates.infrastructure.cluster.x-k8s.io   2020-04-21T19:05:47Z
vspherevms.infrastructure.cluster.x-k8s.io                2020-04-21T19:05:47Zs
```

You can see some fun ones like machines and machinesets, as well as CRDs related to using vSphere.

Using TKG I have deployed three workload clusters.

```
$ tkg get clusters
+--------------+-------------+
| NAME         | STATUS      |
+--------------+-------------+
| dc-cluster   | Provisioned |
| edge-cluster | Provisioned |
| tkg-antrea   | Provisioned |
+--------------+-------------+
```

One of them is using Antrea as the container networking interface (tkg-antrea). The other two clusters are using the current TKG default CNI, Calico.

## Antrea

There are quite a few networking options with k8s. One of the best is [VMware's NSX-t](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/index.html). But another interesting networking option that VMware supports is the open source project [Antrea](https://github.com/vmware-tanzu/antrea/).

>Antrea is a Kubernetes networking solution intended to be Kubernetes native. It operates at Layer3/4 to provide networking and security services for a Kubernetes cluster, leveraging Open vSwitch as the networking data plane.

Basically it orchestrates Open vSwitch. Version 0.6 was [released ](https://github.com/vmware-tanzu/antrea/releases/tag/v0.6.0)only a few days ago.

William Lam has a [blog post](https://www.virtuallyghetto.com/2020/04/how-to-deploy-tanzu-kubernetes-grid-tkg-cluster-with-antrea-cni.html) on how to deploy a TKG workload cluster with Antrea instead of Calico. It's pretty straight forward. TKG uses the concept of "plans" to configure k8s clusters. These plans are basically YAML templates, and by default Calico is set up in the template, but Antrea can be swapped in.

```
$ diff cluster-template-dev-antrea.yaml cluster-template-dev.yaml | head
247,952c247,1036
<      ---
<      apiVersion: apiextensions.k8s.io/v1beta1
<      kind: CustomResourceDefinition
<      metadata:
<        labels:
<          app: antrea
<        name: antreaagentinfos.clusterinformation.antrea.tanzu.vmware.com
<      spec:
<        group: clusterinformation.antrea.tanzu.vmware.com
```

Follow Mr. Lam's blog post if you want to try this out. A few quick commands and you'll be all set.

## Antrea

I configured the tkg-antrea workload cluster to have three worker nodes. Then I applied the below busybox deployment.

```
$ cat deployment.yml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
  labels:
    app: busybox
spec:
  replicas: 6 
  strategy: 
    type: RollingUpdate
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo Container 1 is Running ; sleep 3600']
```

Here are the pods.

*NOTE: I always alias kubectl to k.*

```
$ k get pods
NAME                                  READY   STATUS    RESTARTS   AGE
busybox-deployment-66458f7d4b-4s6vh   1/1     Running   11         11h
busybox-deployment-66458f7d4b-5kv5l   1/1     Running   11         11h
busybox-deployment-66458f7d4b-fgjzx   1/1     Running   11         11h
busybox-deployment-66458f7d4b-gqs8g   1/1     Running   11         11h
busybox-deployment-66458f7d4b-h25sf   1/1     Running   11         11h
busybox-deployment-66458f7d4b-r977v   1/1     Running   11         11h
```

We can also checkout their IP addresses. (Hat tip to this [blog](https://alexbrand.dev/post/first-look-at-antrea-a-cni-plugin-based-on-open-vswitch/).)

```
$ k get pods -o custom-columns='name:.metadata.name,pod ip:.status.podIPs[0].ip,node:.spec.nodeName' --sort-by='.spec.nodeName'
name                                  pod ip       node
busybox-deployment-66458f7d4b-4s6vh   100.96.1.3   tkg-antrea-md-0-548d498b47-2xjf8
busybox-deployment-66458f7d4b-h25sf   100.96.1.4   tkg-antrea-md-0-548d498b47-2xjf8
busybox-deployment-66458f7d4b-fgjzx   100.96.2.3   tkg-antrea-md-0-548d498b47-ckwvd
busybox-deployment-66458f7d4b-r977v   100.96.2.4   tkg-antrea-md-0-548d498b47-ckwvd
busybox-deployment-66458f7d4b-5kv5l   100.96.3.4   tkg-antrea-md-0-548d498b47-f4rmz
busybox-deployment-66458f7d4b-gqs8g   100.96.3.3   tkg-antrea-md-0-548d498b47-f4rmz
```

We have two pods per node. With Antrea, each k8s node gets a /24. In this case node tkg-antrea-md-0-548d498b47-2xjf8 has 100.96.1.0/24m the next node is 100.96.2.0/24, and so on.

```
$ k get nodes -o custom-columns=Name:.metadata.name,PodCIDR:.spec.podCIDR
Name                               PodCIDR
tkg-antrea-control-plane-fzb44     100.96.0.0/24
tkg-antrea-md-0-548d498b47-2xjf8   100.96.1.0/24
tkg-antrea-md-0-548d498b47-ckwvd   100.96.2.0/24
tkg-antrea-md-0-548d498b47-f4rmz   100.96.3.0/24
```

antctl is quite useful.

```
$ k -n kube-system exec -it antrea-agent-2d8xp antrea-agent -- antctl get ovsflows -h
Defaulting container name to antrea-agent.
Use 'kubectl describe pod/antrea-agent-2d8xp -n kube-system' to see all of the containers in this pod.
Dump all the OVS flows or the flows installed for the specified entity.

Usage:
  antctl get ovsflows [flags]

Aliases:
  ovsflows, of

Examples:
  Dump all OVS flows
  $ antctl get ovsflows
  Dump OVS flows of a local Pod
  $ antctl get ovsflows -p pod1 -n ns1
  Dump OVS flows of a NetworkPolicy
  $ antctl get ovsflows --networkpolicy np1 -n ns1
  Dump OVS flows of a flow Table
  $ antctl get ovsflows -t IngressRule

  Antrea OVS Flow Tables:
  0	Classification
  10	SpoofGuard
  20	ARPResponder
  30	ConntrackZone
  31	ContrackState
  40	DNAT
  50	EgressRule
  60	EgressDefaultRule
  70	L3Forwarding
  80	L2Forwarding
  90	IngressRule
  100	IngressDefaultRule
  105	ConntrackCommit
  110	Output
SNIP!
```

We can dump the flows using antctl based on the above flow table names. There are no network polices in place right now, so the returned flows are few.

*NOTE: Use an antrea agent that is on a worker node.*

```
$ k -n kube-system exec -it antrea-agent-hs5kp antrea-agent -- antctl get ovsflows -T IngressRule
Defaulting container name to antrea-agent.
Use 'kubectl describe pod/antrea-agent-hs5kp -n kube-system' to see all of the containers in this pod.
FLOW                                                                                                  
table=90, n_packets=155728, n_bytes=14819958, priority=210,ct_state=-new+est,ip actions=resubmit(,105)
table=90, n_packets=17156, n_bytes=1269544, priority=210,ip,nw_src=100.96.1.1 actions=resubmit(,105)  
table=90, n_packets=15, n_bytes=1110, priority=0 actions=resubmit(,100)   
```

Above we can see the default flows when there are no network polices.

## Network Policies

k8s started out with the idea that every pod could talk to every other pod over the network. Obviously it couldn't stay like this...we need to be able to limit and control connectivity. So Antrea supports [Kubernetes Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

>By default, pods are non-isolated; they accept traffic from any source. Pods become isolated by having a NetworkPolicy that selects them. Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections that are not allowed by any NetworkPolicy. (Other pods in the namespace that are not selected by any NetworkPolicy will continue to accept all traffic.) - [k8s docs](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

Deploy nginx:

```
$ cat nginx-deployment-service.yaml 
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx
  replicas: 3 # tells deployment to run 1 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  externalTrafficPolicy: Local
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
```

ClusterIP:

```
$ k get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   100.64.0.1       <none>        443/TCP        23h
nginx        NodePort    100.70.238.176   <none>        80:31710/TCP   30m
```

With nginx deployed, I can exec into one of the busybox instances I already created and do this:

```
/ # wget -q -O - 100.70.238.176 | grep title
<title>Welcome to nginx!</title>
```

But now lets apply a network policy.

```
$ cat network-policy.yml 
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: nginx-allow
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
      - podSelector:
          matchLabels:
            app: nginx
$ k apply -f network-policy.yml 
networkpolicy.networking.k8s.io/nginx-allow created
$ k get networkpolicy
NAME          POD-SELECTOR         AGE
nginx-allow   app=nginx            3s
```

A busybox pod can no longer connect to the nginx deployment (only pods with 32app=nginx).

```
$ k -n default exec -it busybox-deployment-66458f7d4b-4s6vh -- /bin/sh
/ # wget -q -O - 100.70.238.176 | grep title
wget: can't connect to remote host (100.70.238.176): Connection timed out
```

Let's check the flows now.

```
$ k -n kube-system exec -it antrea-agent-hs5kp antrea-agent -- antctl get ovsflows -T IngressRule
Defaulting container name to antrea-agent.
Use 'kubectl describe pod/antrea-agent-hs5kp -n kube-system' to see all of the containers in this pod.
FLOW                                                                                                  
table=90, n_packets=155929, n_bytes=14839032, priority=210,ct_state=-new+est,ip actions=resubmit(,105)
table=90, n_packets=17178, n_bytes=1271172, priority=210,ip,nw_src=100.96.1.1 actions=resubmit(,105)  
table=90, n_packets=0, n_bytes=0, priority=200,ip,nw_src=100.96.3.5 actions=conjunction(1,1/2)        
table=90, n_packets=0, n_bytes=0, priority=200,ip,nw_src=100.96.2.5 actions=conjunction(1,1/2)        
table=90, n_packets=0, n_bytes=0, priority=200,ip,nw_src=100.96.1.5 actions=conjunction(1,1/2)        
table=90, n_packets=0, n_bytes=0, priority=200,ip,reg1=0x6 actions=conjunction(1,2/2)                 
table=90, n_packets=0, n_bytes=0, priority=190,conj_id=1,ip actions=resubmit(,105)                    
table=90, n_packets=15, n_bytes=1110, priority=0 actions=resubmit(,100)  
```

More flows!

Those IPs are of the nginx pods.

```
$ k get pods -o custom-columns='name:.metadata.name,pod ip:.status.podIPs[0].ip,node:.spec.nodeName' --sort-by='.spec.nodeName' --selector app=nginx
name                     pod ip       node
nginx-85ff79dd56-2ndjq   100.96.1.5   tkg-antrea-md-0-548d498b47-2xjf8
nginx-85ff79dd56-fdrsp   100.96.2.5   tkg-antrea-md-0-548d498b47-ckwvd
nginx-85ff79dd56-cxjvw   100.96.3.5   tkg-antrea-md-0-548d498b47-f4rmz
```

```
$ k -n kube-system exec -it antrea-agent-hs5kp antrea-agent -- antctl get networkpolicies
Defaulting container name to antrea-agent.
Use 'kubectl describe pod/antrea-agent-hs5kp -n kube-system' to see all of the containers in this pod.
NAMESPACE NAME        APPLIED-TO                           RULES
default   nginx-allow 766a9e51-f132-5c2f-b862-9ac68e75d77d 1 
```

Above we can ask Antrea about network polices as well.

## Conclusion

Open vSwitch is the swiss army knife of networking. It's open. It's widely used. It can run on Linux and Windows. It serves as a great basis for a software defined networking system for Kubernetes. 

Checkout Antrea's [roadmap](https://github.com/vmware-tanzu/antrea/blob/master/ROADMAP.md) to see where they are going. Many great features on the horizon!

**PS.** Please note that TKG does not officially support Antrea at the time I wrote this, but they are definitely looking at it.