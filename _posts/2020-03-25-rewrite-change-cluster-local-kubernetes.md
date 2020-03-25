---
layout: post
title: Change or Rewrite cluster.local in Kubernetes
categories:
header_image: "/img/k8s-boat.jpg"
header_permalink: "https://unsplash.com/photos/eUMEWE-7Ewg"
summary: "Maybe you just want to use another domain for services. Who knows."

---

# {{ page.title }}

Some organizations may want to use a domain other than cluster.local as the service domain in Kubernetes.

One way to do that is to add a rewrite substring rule in CoreDNS. (Of course this assumes the use of CoreDNS.)

I have a VMware Enterprise PKS cluster to test with.

The CoreDNS configmap starts out looking like this:

```
 $ kubectl get -n kube-system cm/coredns -o yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        proxy . /etc/resolv.conf {
          policy sequential # needed for workloads to be able to use BOSH-DNS
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"Corefile":".:53 {\n    errors\n    health\n    kubernetes cluster.local in-addr.arpa ip6.arpa {\n      pods insecure\n      upstream\n      fallthrough in-addr.arpa ip6.arpa\n    }\n    prometheus :9153\n    proxy . /etc/resolv.conf {\n      policy sequential # needed for workloads to be able to use BOSH-DNS\n    }\n    cache 30\n    loop\n    reload\n    loadbalance\n}\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"coredns","namespace":"kube-system"}}
  creationTimestamp: "2020-03-25T14:21:39Z"
  name: coredns
  namespace: kube-system
  resourceVersion: "1341"
  selfLink: /api/v1/namespaces/kube-system/configmaps/coredns
  uid: 0dd853af-a4d7-498d-9745-752cbf8fbffb
```

All I want to do is add this line:

```
rewrite name substring svc.example.com svc.cluster.local
```

So the full file looks like:


```
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        rewrite name substring svc.example.com svc.cluster.local
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        proxy . /etc/resolv.conf {
          policy sequential # needed for workloads to be able to use BOSH-DNS
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
```

Let's replace the existing configmap with the new one.

```
$ k replace -f coredns-configmap-rewrite.yml 
configmap/coredns replaced
```

After a few seconds, CoreDNS will restart.

I've deployed two nginx based servcies.

```
$ k get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.100.200.1     <none>        443/TCP        3h30m
nginx-1      LoadBalancer   10.100.200.226   10.1.4.18     80:31668/TCP   138m
nginx-2      LoadBalancer   10.100.200.160   10.1.4.19     80:31692/TCP   116m
```

I'll exec into one and run dig.

```
root@nginx-1-56958fdfdd-d26tj:/# dig +short nginx-1.default.svc.cluster.local
10.100.200.226
root@nginx-1-56958fdfdd-d26tj:/# dig +short nginx-2.default.svc.cluster.local
10.100.200.160
root@nginx-1-56958fdfdd-d26tj:/# dig +short nginx-2.default.svc.example.com  
10.100.200.160
root@nginx-1-56958fdfdd-d26tj:/# dig +short nginx-1.default.svc.example.com
10.100.200.226
```

Note how both domains work. Nice!

That's it.