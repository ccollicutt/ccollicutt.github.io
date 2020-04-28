---
layout: post
title: Install the Harbor Container Image Registry Locally
categories:
header_image: "/img/harbor1.jpg"
header_permalink: "https://unsplash.com/photos/V1NM69d-keg"
summary: "Harbor is an Enterprise featured, open source OCI registry, and in this post we show how to deploy it into Kind with Helm and Contour"

---

# {{ page.title }}

In this post I'll deploy a Harbor image registry on a local OSX workstation/laptop.

Requirements:

* OSX
* Brew
* Docker for Desktop

## Kind

[Kind](https://kind.sigs.k8s.io/) stands for "Kubernetes in Docker" and is an easy way to get a local Kubernetes cluster running.

*Note: Docker for Desktop also provides Kubernetes functionality, but in this example we are using Kind to provide k8s.*

Install kind with brew.

```
brew install kind
```

Create a kind based cluster.

*Note the use of "extraPortMappings".*

```
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
        authorization-mode: "AlwaysAllow"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

Once that kind cluster is created we can see k8s nodes.

```
$ kubectl get nodes
NAME                 STATUS   ROLES    AGE     VERSION
kind-control-plane   Ready    master   2m31s   v1.17.0
```

Nice!

## Contour Ingress

[Contour](https://projectcontour.io/) is an advanced open source ingress system supported in part by VMware.

>Contour is an open source Kubernetes ingress controller providing the control plane for the Envoy edge and service proxy.​ Contour supports dynamic configuration updates and multi-team ingress delegation out of the box while maintaining a lightweight profile.

Deploy Contour.

```
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```

Patch it:

```
kubectl patch daemonsets -n projectcontour envoy -p '{"spec":{"template":{"spec":{"nodeSelector":{"ingress-ready":"true"},"tolerations":[{"key":"node-role.kubernetes.io/master","operator":"Equal","effect":"NoSchedule"}]}}}}'
```

Contour should now be running.

*Note: kubectl is provided by Docker for Desktop.*

```
$ kubectl get pods -n projectcontour
NAME                       READY   STATUS      RESTARTS   AGE
contour-54df6b8854-dlsnr   1/1     Running     0          83s
contour-54df6b8854-m2w8k   1/1     Running     0          83s
contour-certgen-n78dz      0/1     Completed   0          83s
envoy-kwr8x                2/2     Running     0          10s
```

Contour will provide an easy way to get ingress. Of course, Contour offers a lot more than just ingress, but it's all we need for now.

## Install Helm

Getting helm is quite easy with brew.

```
brew install helm
```

Also we get helm 3!

## Install Harbor

Now that we have Kubernetes and Helm, installing Harbor is straight forward, though there are many options available in the Helm chart. We will not be making any changes and use the defaults provided.

Add the Harbor Helm repository.

```
helm repo add harbor https://helm.goharbor.io
```

And install Harbor:

```
helm install local-harbor harbor/harbor
```

After a few minutes, there should be several Harbor k8s objects, such as pods.

```
$ k get pods
NAME                                                 READY   STATUS    RESTARTS   AGE
local-harbor-harbor-chartmuseum-bd9c45cbc-gwkbj      1/1     Running   0          58s
local-harbor-harbor-clair-865c9bc5db-cvbk8           1/2     Running   2          58s
local-harbor-harbor-core-64479f8d85-rkqm2            1/1     Running   0          58s
local-harbor-harbor-database-0                       1/1     Running   0          58s
local-harbor-harbor-jobservice-8448b58df7-pgknp      1/1     Running   0          58s
local-harbor-harbor-notary-server-5bd9f5d966-56kk6   1/1     Running   0          58s
local-harbor-harbor-notary-signer-5fbfb48945-l2x54   1/1     Running   0          58s
local-harbor-harbor-portal-756d5d7d9d-xlv2g          1/1     Running   0          58s
local-harbor-harbor-redis-0                          1/1     Running   0          58s
local-harbor-harbor-registry-57989b6446-w8vd8        2/2     Running   0          58s
```

*NOTE: There are a lot of services running with Harbor. Future work for this post will include trying to make it a bit easier on resources on a workstation.*

## Access Harbor

Add a hostname to your `/etc/hosts` file.

```
127.0.0.1 core.harbor.domain
```

Now open a browser session to http://core.harbor.domain/harbor

```
login: admin
password: Harbor12345
```

At this point Harbor is available for local use.

## Delete Harbor and Kind 

To remove everything added:

```
helm uninstall local-harbor
kind delete cluster
```

Then shutdown Docker for Desktop.

## Conclusion

There are still a few things to be ironed out here. My Mac laptop starts sounding like a 747 pretty quickly. In a future post or update I'll see how small we can make harbor, for now whatever helm deploys by default is what we are getting. The Harbor helm chart has many options that can be applied, but all defaults are being used here.