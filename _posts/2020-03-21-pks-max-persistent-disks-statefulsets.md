---
layout: post
title: PKS and Persistent Volumes
categories:
header_image: "/img/3d.jpg"
summary: "How many persistent volumes can you get in k8s? Where are the constraints?"

---

# {{ page.title }}

I work at [VMware in the Tanzu group](https://tanzu.vmware.com/) as a Solutions Engineer. Occasionally customers ask me numbers questions, by that I mean they ask about so called "speeds and feeds" like "How many of this can you do?" and "How fast is that?". The answers to these change all the time, and overall exact numbers aren't typically important to the business outcomes. That said, sometimes it's good to explore what the limitation are, and perhaps more importantly *where* the limitations exist. 

Actually, as I write this, I don't think *limitations* is the right word, as it's really about how systems and products work together to create constraints. Yes, I like that better. Constraints.

## StatefulSet

In this lab I have:

* vSphere 6.7u3 - Single physical host
* NSX-t 2.5
* Harbor 1.10
* PKS 1.6 and 1.7 

PKS has kindly created a small cluster, called small-pks. Actually this cluster is 1.7, but that doesn't have any bearing on this quick test.

```
$ pks cluster small-pks

PKS Version:              1.7.0-build.19
Name:                     small-pks
K8s Version:              1.16.7
Plan Name:                small
UUID:                     f5879fcf-46ec-4a7e-b2cb-7a74004aed87
Last Action:              UPGRADE
Last Action State:        succeeded
Last Action Description:  Instance update completed
Kubernetes Master Host:   small.example.com
Kubernetes Master Port:   8443
Worker Nodes:             3
Kubernetes Master IP(s):  10.197.123.128
Network Profile Name:
```

The cluster has 3 worker nodes, so we should be able to get a max of stateful sets that is 3x...something.  But what is that something? I'm making a guess the limitation is not more than 64, so let's try 200. (This example is [from here](https://github.com/kubernetes/examples/blob/master/staging/volumes/vsphere/simple-statefulset.yaml).)

```
$ cat simple-statefulset.yaml 
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1 #  for k8s versions before 1.9.0 use apps/v1beta2  and before 1.8.0 use extensions/v1beta1
kind: StatefulSet
metadata:
  name: web
  labels:
    app: nginx
spec:
  serviceName: "nginx"
  selector:
    matchLabels:
      app: nginx
  replicas: 200
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
      storageClassName: ci-storage
```

After that runs for a while, I end up stuck with 132 replicas.

```
$ k get statefulsets
NAME   READY     AGE
web    132/200   5h12m
```

Why is that?

Well, it turns out, and [this post](https://orchestration.io/2018/06/13/storage-classes-in-pivotal-container-service-pks/) does a better job of discussing it, that vSphere allows for 4 SCSI adapters per virtual machine, and PKS will use 3 of them. Each of those 3 adapters can have 15 disks. Thus we end up with a constraint of 45 disks per vm for use with persistent volumes.

In the image below you can see the vm has 48 disks. 3 disks are for the OS and managing the vm itself via bosh, and are attached to the first SCSI adapter, the other 45 are persistent volumes attached to the remaining 3 adapters. ([Here are the maximums for vSphere 6.7](https://configmax.vmware.com/guest?vmwareproduct=vSphere&release=vSphere%206.7&categories=1-0.).)

![vm disks](/img/vm-disks.jpg)


In the end the constraint is around the number of SCSI adapters possible, how PKS uses them, and how many virtual drives can be attached. With those all in play, we get a constraint of 45 persistent volumes per virtual machine in PKS. Is 45 a lot, a little, or just right? Hard to say. Sounds about right.

---

*PS. Note that numbers like this change all the time. By the time you read this, maybe it will be higher than 45 PVs per node.*