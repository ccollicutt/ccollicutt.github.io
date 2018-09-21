---
layout: post
title: Local Persistent Volumes with Kubernetes
categories:
header_image: /img/buffalo.jpg
header_permalink: https://unsplash.com/photos/ZOUSOJFzQHg
---

# {{ page.title }}

Let's quickly discuss using persistent local volumes with Kubernetes.

First, get yourself a k8s. I have one here running on a packet.net instance. It's only a single node. Deployed with kubeadm and is running calico as the network plugin.

```
# k get nodes
NAME      STATUS    ROLES     AGE       VERSION
cka       Ready     master    6h        v1.11.1
```

Let's look at everything that's running.

```
# k get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   172.17.0.1   <none>        443/TCP   6h

# k get all -n kube-system
NAME                                           READY     STATUS    RESTARTS   AGE
pod/calico-etcd-2hcdc                          1/1       Running   0          6h
pod/calico-kube-controllers-74b888b647-qr86d   1/1       Running   0          6h
pod/calico-node-5jmrc                          2/2       Running   17         6h
pod/coredns-78fcdf6894-4ngmq                   1/1       Running   0          6h
pod/coredns-78fcdf6894-gzqcw                   1/1       Running   0          6h
pod/etcd-cka                                   1/1       Running   0          6h
pod/kube-apiserver-cka                         1/1       Running   0          6h
pod/kube-controller-manager-cka                1/1       Running   0          6h
pod/kube-proxy-62hp2                           1/1       Running   0          6h
pod/kube-scheduler-cka                         1/1       Running   0          6h

NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
service/calico-etcd   ClusterIP   172.17.0.136   <none>        6666/TCP        5h
service/kube-dns      ClusterIP   172.17.0.10    <none>        53/UDP,53/TCP   6h

NAME                         DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                     AGE
daemonset.apps/calico-etcd   1         1         1         1            1           node-role.kubernetes.io/master=   6h
daemonset.apps/calico-node   1         1         1         1            1           <none>                            6h
daemonset.apps/kube-proxy    1         1         1         1            1           beta.kubernetes.io/arch=amd64     6h

NAME                                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/calico-kube-controllers    1         1         1            1           6h
deployment.apps/calico-policy-controller   0         0         0            0           6h
deployment.apps/coredns                    2         2         2            2           6h

NAME                                                  DESIRED   CURRENT   READY     AGE
replicaset.apps/calico-kube-controllers-74b888b647    1         1         1         6h
replicaset.apps/calico-policy-controller-55b469c8fd   0         0         0         6h
```

The c2.medium.x86 (AMD EPYC!) has two extra SSDs.

```
mkfs.ext4 /dev/sdc
mkfs.ext4 /dev/sdd
mkdir -p /mnt/disks/sdc
mkdir -p /mnt/disks/sdd
mount /dev/sdc /mnt/disks/sdc
mount /dev/sdd /mnt/disks/sdd
```

Now both are formated and mounted.

```
# lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 111.8G  0 disk
sdb      8:16   0 111.8G  0 disk
├─sdb1   8:17   0   512M  0 part /boot/efi
├─sdb2   8:18   0   1.9G  0 part
└─sdb3   8:19   0 109.4G  0 part /
sdc      8:32   0 447.1G  0 disk /mnt/disks/sdc
sdd      8:48   0 447.1G  0 disk /mnt/disks/sdd
```

I usually abbreviate kubectl to k.

```
alias k=kubectl
```

Setup a storage class.

```
cat > storage-class.yml <<EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF

kubectl create -f storage-class.yml
```

List the available storage classes.

```
# k get sc
NAME            PROVISIONER                    AGE
local-storage   kubernetes.io/no-provisioner   4s
```

Create a persistent volume.

```
cat > pv-sdc.yml <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-sdc
spec:
  capacity:
    storage: 440Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/disks/sdc
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - cka
EOF

kubectl create -f pv-sdc.yml
```

Same for sdd.

```
cat > pv-sdd.yml <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-sdd
spec:
  capacity:
    storage: 440Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/disks/sdd
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - cka
EOF

kubectl create -f pv-sdd.yml
```

Now I have two PVs.

```
# k get pv
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM     STORAGECLASS    REASON    AGE
local-pv-sdc   440Gi      RWO            Retain           Available             local-storage             53s
local-pv-sdd   440Gi      RWO            Retain           Available             local-storage             4s
```

Create a persistent volume claim.

```
cat > pvc1.yml <<EOF
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc1
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 100Gi
EOF

kubectl create -f pvc1.yml
```

PVC will be pending until we create a node that uses it.

```
# k get pvc
NAME      STATUS    VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS    AGE
pvc1      Pending                                       local-storage   3s
```

Now a deployment that uses the pvc.

```
cat > deploy-nginx.yml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: storage
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: pvc1
EOF

kubectl create -f deploy-nginx.yml
```

pvc is now bound.

```
# k get pvc
NAME      STATUS    VOLUME         CAPACITY   ACCESS MODES   STORAGECLASS    AGE
pvc1      Bound     local-pv-sdc   440Gi      RWO            local-storage   21m
```

Expose nginx.

```
# kubectl expose deployment nginx-deployment --type=NodePort
service/nginx-deployment exposed
# kubectl get svc nginx-deployment
NAME               TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-deployment   NodePort   172.17.0.157   <none>        80:31813/TCP   7m
```

Now create an index.html page. I'll use a uuid just to show it's not already setup.

```
# uuid=`uuidgen`
# echo $uuid
b5d3c8cd-a56f-4252-a026-7107790fcd44
# echo $uuid > /mnt/disks/sdc/index.html
# curl 172.17.0.157
b5d3c8cd-a56f-4252-a026-7107790fcd44
```

Let's delete the pod.

```
# k delete pod nginx-deployment-7d869874bc-dlz4s
pod "nginx-deployment-7d869874bc-dlz4s" deleted
```

A new pod will automatically be created by Kubernetes.

```
# k get pods
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-7d869874bc-fkt4m   1/1       Running   0          28s
```

Run the same curl.

```
# curl 172.17.0.157
b5d3c8cd-a56f-4252-a026-7107790fcd44
```

Same UUID.

Lets check from the pod.

```
# k exec -it nginx-deployment-7d869874bc-fkt4m -- cat /usr/share/nginx/html/index.html
b5d3c8cd-a56f-4252-a026-7107790fcd44
```

As expected.

## Conclusion

Kubernetes.io has a [blog post](https://kubernetes.io/blog/2018/04/13/local-persistent-volumes-beta/) announcing local persistent volumes. It's worth a read. Anything I say here of value would just be copying from that. :) There are several reasons not to use this model for storage. Seems like it would be best for data heavy applications...data gravity. There is also a [simple volume manager](https://github.com/kubernetes-incubator/external-storage/tree/master/local-volume) available as well, but I didn't explore that in this blog post.