---
layout: post
title: Kubernetes Tips and Tricks
categories:
header_image: "/img/k8s-tips-tricks.jpg"
header_permalink: "https://unsplash.com/photos/Esq0ovRY-Zs"
summary: "Some commands you should never use, some you should always use"

---

# {{ page.title }}

A few tips and tricks I've come across. Starting off small!

## Export Cluster Config from Kubeconfig

Say you have a whole bunch of clusters set in your kubeconfig file and you want to extract one. Just one.

Set your config to that cluster (maybe use kubectx) and do:

```
kubectl config view --minify --raw > cluster.kubeconfig
```

Boom!

## Writing an Operator in Shell!

See the [shell operator](https://github.com/flant/shell-operator). Good times!
