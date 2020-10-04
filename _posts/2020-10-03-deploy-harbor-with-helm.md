---
layout: post
title:  Deploy Harbor with Helm
categories:
header_image: "/img/harbor-helm2.png"
summary: "Using vSphere with Tanzu and the Tanzu Application Catalog"

---

# {{ page.title }}

I've put together a demo/workshop/lab/what-have-you for deploying Harbor to Kubernetes with Helm. There's also a video below that goes through all the steps, showing deploying a Tanzu Kubernetes Grid workload cluster via vSphere with Tanzu, and using the Tanzu Application Catalog to consume a Harbor Helm Chart.

The idea with this demo is to provide a starting place to build up a production deployment of Harbor via Helm.

That said, any Kubernetes cluster should be fine, as long as it has load balancer and persistent volume capability.

* Git repo with documentation [deploying Harbor with Helm](https://github.com/ccollicutt/harbor-demos)

Here's a couple links to vSphere with Tanzu and the Tanzu Application Catalog which were used as the Kubernetes and Helm chart provider respectively.

* [vSphere with Tanzu](https://tanzu.vmware.com/kubernetes-grid)
* [Tanzu Application Catalog](https://tanzu.vmware.com/application-catalog)

