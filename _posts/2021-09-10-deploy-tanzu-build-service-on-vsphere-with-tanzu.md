---
layout: post
title:  "Deploy Tanzu Build Service into a vSphere with Tanzu Workload Cluster"
categories:
header_image: "/img/tbs-on-vsphere.jpg"
summary: "Don't tell me you're still using dockerfiles?"

---

# {{ page.title }}

In this post we'll deploy the [Tanzu Build Service](https://tanzu.vmware.com/build-service) (TBS) onto a [vSphere with Tanzu](https://www.vmware.com/ca/products/vsphere/vsphere-with-tanzu.html) Kubernetes workload cluster.

## Requirements

1. vSphere with Tanzu deployed and enough resources for the TBS workload cluster
2. A container image repository, such as Harbor, or Azure CR, etc, any compliant registry should do
3. A place to run commands (a linux host is best IMHO)

## About vSphere with Tanzu

VMware believes that you will have many Kubernetes clusters. Not just one or two or three. So, the main feature of vSphere with Tanzu is to manage the lifecycle of **many** Kubernetes clusters. The way we do this is by extending Kubernetes with something called Cluster API.

This means, and this can be confusing, that when we want to create a Kubernetes cluster, we actually ask a specialized Kubernetes cluster--the Supervisor Cluster--to do this for us. So we use Kubernetes to deploy Kubernetes. Make sense?

Once the Supervisor Cluster has created our "workload" cluster (and there will be many of these), we can then talk directly to that new workload cluster via its own, completely separate, Kubernetes API.

So, to create a workload cluster we ask the supervisor cluster. Once the workload cluster is created, we talk to it to deploy applications into it. Simple enough once you get the hang of it.

## Deploy a vSphere with Tanzu Workload Cluster for TBS

We need a cluster to install TBS into. That cluster needs a couple of things:

1. Enough room on the nodes to build images - the default 16Gi disk size is not enough, we need a cluster with at least 50Gi on each node for the image builds that TBS does
2. The right RBAC configuration and permissions

First, let's deploy the cluster with larger disks.

### Login to the Supervisor Cluster

Ensure you are logged into your vSphere with Tanzu supervisor Kubernetes cluster.

>NOTE: I alias `kubectl` to `k`.

>NOTE: I'm logging into my vSphere with Tanzu supervisor cluster found at 10.0.14.128...yours will of course be different. I'm also using the admin account.

```
$ k vsphere login --server 10.0.14.128 --insecure-skip-tls-verify -u administrator@vsphere.local

Password:
Logged in successfully.

You have access to the following contexts:
   10.0.14.128
   dev-team-purple
   dev-team-tundra
   test-ns

If the context you wish to use is not in this list, you may need to try
logging in again later, or contact your cluster administrator.

To change context, use `kubectl config use-context <workload name>`
```

Now that I'm logged in, I have access to the supervisor cluster as well as the supervisor namespaces. So there will be several kube contexts set up.

>NOTE: I'm using kubectx aliased to kc instead of `kubectl config use-context` just because it's what I always use and I find it easier, IMHO.

```
$ kc
10.0.14.128
dev-team-purple
dev-team-tundra
test-ns
```

<br />

### Switch the Desired Supervisor Namespace

I'm going to deploy the TBS workload cluster into the `dev-team-tundra` supervisor namespace. So I'll switch to that config.

```
$ kc dev-team-tundra
✔ Switched to context "dev-team-tundra".
```

<br />

### Deploy a Workload Cluster

Set up a few variables that will be dependent on how you have set up the supervisor cluster, storage, etc.

>NOTE: Change these to fit your environment.

```
export NS=dev-team-tundra # the supervisor namespaces to use
export SC=k8s-storage-policy # the storage policy configured when enabling workload management
export CLUSTER_NAME="tanzu-build-service-cluster2" # the name of the workload cluster
export K8S_VERSION=v1.20.7 # version of k8s to deploy
```

Now I'll deploy a cluster to that supervisor namespace.

>NOTE: This will deploy the cluster! Note the pipe at the top of the command to `kubectl`.

>NOTE: Notice that in the YAML here we define a separate disk for /var/lib/containerd that is 50Gi in size. We need this for TBS.

```
cat << EOF | kubectl create -f-
apiVersion: run.tanzu.vmware.com/v1alpha1
kind: TanzuKubernetesCluster
metadata:
  name: $CLUSTER_NAME
  namespace: $NS
spec:
  distribution:
    version: $K8S_VERSION
  topology:
    controlPlane:
      count: 1
      class: best-effort-medium
      storageClass: $SC
      volumes:
        - name: etcd
          mountPath: /var/lib/etcd
          capacity:
            storage: 4Gi
    workers:
      count: 3
      class: best-effort-medium
      storageClass: $SC
      volumes:
        - name: containerd
          mountPath: /var/lib/containerd
          capacity:
            storage: 50Gi
  settings:
    network:
      services:
        cidrBlocks: ["10.96.0.0/16"]
      pods:
        cidrBlocks: ["172.20.0.0/16"]
    storage:
      classes: ["$SC"]
      defaultClass: $SC
EOF
```

After a few minutes the cluster will be deployed. (How long depends on the speed of your infrastructure, but say 15-20 minutes.)

### Login to the Workload Cluster

Once the new, completely separate k8s cluster is created, we use the `kubectl vsphere` plugin to login to the workload cluster, switch to that kube context, and from this point on we'll talk to that cluster's Kubernetes API, not the supervisor cluster.

```
k vsphere login --server 10.0.14.128 --insecure-skip-tls-verify -u administrator@vsphere.local\
  --tanzu-kubernetes-cluster-name $CLUSTER_NAME \
  --tanzu-kubernetes-cluster-namespace $NS
```

Once that command completes you'll have a new context for the workload cluster.

```
$ kc
10.0.14.128
dev-team-purple
dev-team-tundra
tanzu-build-service-cluster2
test-ns
```

Use that context to deploy TBS into that cluster.

```
$ kc tanzu-build-service-cluster2
✔ Switched to context "tanzu-build-service-cluster2".
```

Once switched to taht config, we can talk to that cluster's API, and for example, get the nodes that make up the cluster. There should be one control plane and three worker nodes, unless you adjusted the cluster YAML.

```
$ k get nodes
NAME                                                         STATUS   ROLES                  AGE   VERSION
tanzu-build-service-cluster2-control-plane-86zx9             Ready    control-plane,master   39m   v1.20.7+vmware.1
tanzu-build-service-cluster2-workers-268cf-9686cf46d-4ccdh   Ready    <none>                 33m   v1.20.7+vmware.1
tanzu-build-service-cluster2-workers-268cf-9686cf46d-6sznb   Ready    <none>                 33m   v1.20.7+vmware.1
tanzu-build-service-cluster2-workers-268cf-9686cf46d-d7nld   Ready    <none>                 33m   v1.20.7+vmware.1
```

<br />

### Configure Pod Security Policy

The supervisor cluster configures some default security which we will need to further configure to allow TBS to deploy into this cluster.

>NOTE: Ensure your kubeconfig is set to the workload cluster, not the supervisor cluster!

>NOTE: This RBAC is good for a PoC, it's likely that we would want to customize this for production.

```
cat << EOF | kubectl create -f-
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: psp:privileged
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - vmware-system-privileged
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: all:psp:privileged
roleRef:
  kind: ClusterRole
  name: psp:privileged
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: system:serviceaccounts
  apiGroup: rbac.authorization.k8s.io
EOF
```

Now we can deploy TBS!

## Install TBS

>NOTE: Best to read through the [official docs](https://docs.pivotal.io/build-service/1-2/installing.html) before proceeding.

I'm going to skip through the requirements section and assume that you have read through it and downloaded all the correct pieces which are laid out in the above docs. There are quite a few pieces so please do read carefully. There are some activities you have to do, like downloading CLIs and accepting EULAs and the like.

At this point we can start the deployment.

### Copy TBS Images to Your Container Image Repository

We use `imgpkg` to copy the TBS images to your repo.

First set up a variable.

```
export IMAGE_REPO="<your.repo/some-repo>"
```

In my example I'm using the Azure container registry. (Usually I would use Harbor, but I thought I'd try something different today.)

>NOTE: This will take a few minutes to complete as we are copying several images from one repo to another.

```
imgpkg copy -b "registry.pivotal.io/build-service/bundle:1.2.2" --to-repo $IMAGE_REPO
```

Now pull this image locally and unpack in /tmp.

```
imgpkg pull -b $IMAGE_REPO:1.2.2 -o /tmp/bundle
```

There should be files in `/tmp/bundle`.

```
$ find /tmp/bundle/ | head
/tmp/bundle/
/tmp/bundle/values.yaml
/tmp/bundle/.imgpkg
/tmp/bundle/.imgpkg/images.yml
/tmp/bundle/config
/tmp/bundle/config/values.star
/tmp/bundle/config/ca-cert.yaml
/tmp/bundle/config/pod-webhook
/tmp/bundle/config/pod-webhook/rbac.yaml
/tmp/bundle/config/pod-webhook/deployment.yaml
```

<br />

## Deploy TBS

And we can now deploy.

First configure some variables.

>NOTE: Please configure all of these variables. They should not be empty.

```
export IMAGE_REPO="" # where we we copied the TBS images to
export REGISTRY_USER=""
export REGISTRY_PASS=""
export TANZUNET_USER=""
export TANZUNET_PASS=""
```

Next, perform the deployment.

>NOTE: We're using various Carvel tools to perform the deployment. `ytt`, `kbld`, and `kapp` to name a few.

>NOTE: If you supply the tanzunet user/password TBS will be configured to automatically talk to tanzunet and download the latest buildpacks so that you will always be automatically up to date. If they aren't supplied, that's fine, you'll just be in charge of updating the underlying buildpacks. When supplying this information the last step in the deployment can take a while because it's downloading and uploading images into your registry.

>NOTE: Of course this assumes that you've followed the TBS docs and downloaded all the Carvel CLIs.

```
ytt -f /tmp/bundle/values.yaml \
    -f /tmp/bundle/config/ \
    -v docker_repository="$IMAGE_REPO" \
    -v docker_username="$REGISTRY_USER" \
    -v docker_password="$REGISTRY_PASS" \
    -v tanzunet_username="$TANZUNET_USER" \
    -v tanzunet_password="$TANZUNET_PASS" \
    | kbld -f /tmp/bundle/.imgpkg/images.yml -f- \
    | kapp deploy -a tanzu-build-service -f- -y
```

This will take a few minutes to deploy.

Once it completes we can run `kapp list`.

```
$ kapp list
Target cluster 'https://10.0.14.140:6443' (nodes: tanzu-build-service-cluster2-control-plane-86zx9, 3+)

Apps in namespace 'default'

Name                 Namespaces                      Lcs   Lca  
tanzu-build-service  (cluster),build-service,kpack,  true  2m  
                     stacks-operator-system                  

Lcs: Last Change Successful
Lca: Last Change Age

1 apps

Succeeded
```

And would want to see `succeeded`.

## Build an Image

With TBS installed we have extended the Kubernetes API so that it knows how to build container images. So to build images we'll talk to the Kubernetes API using YAML, just like any other Kubernetes object such as pods.

### Configure Container Image Repository Secret

Decide what namespace you want to have the images in. I'll use the default namespace.

>NOTE: I use [kubectx](https://github.com/ahmetb/kubectx) to manage my clusters and namespaces.

```
$ kn default
✔ Active namespace is "default"
```

Create a repository secret. TBS needs to have write access to the container image repo to push the resulting image to.

Use `kp` to do that.

set up some vars.

```
export SECRET_NAME=""
export REGISTRY=""
export REGISTRY_USER=""
export REGISTRY_PASS=""
```

Now create the secret.

```
kp secret create $SECRET_NAME --registry $REGISTRY --registry-user $REGISTRY_USER --namespace default
```

You will have to enter the registry password on the command line.

Now that we have TBS installed and a repo secret configured we can build an image.

### Build Spring Petclinic

We need to ensure we're going to upload the newly built image to the right container image repository. This repository is where you want the resulting image to end up!

```
export REPOSITORY="your.container.image.repo/some-repo"
```

Now ask TBS to build the image.

>NOTE: This will take a while on the first build as all the maven dependencies will get downloaded...Spring Petclinic is written in Java.

```
[ -z "$REGISTRY" ] && echo "ERROR: Please set REGISTRY variable" || \
  kp image create spring-petclinic-image \
  --tag $REGISTRY/spring-petclinic-image \
  --git https://github.com/ccollicutt-tanzu/spring-petclinic \
  --git-revision main
```

We can watch logs of the build with:

```
kp build logs spring-petclinic-image
```

Once the build is completed, the image will be pushed to the `$REPOSITORY`.

>NOTE: I've pushed to Azure's container image registry, yours would be different. [Harbor](https://goharbor.io/) is a great choice as well. I just wanted to try out Azure's registry.

```
$ az acr repository list --name $MY_REPO --output table | grep spring-petclinic-image
build-service/spring-petclinic-image
```

From here on we can magically build almost any application just from the artifacts or source, without having to write and manage a dockerfile. Amazing!

## Conclusion

Ultimately this post was about setting up a vSphere with Tanzu cluster that can accept a TBS deployment to it. We didn't spend much time on **why** you'd want to use TBS. For that I'd suggest watching a [video](https://www.youtube.com/watch?v=IMmUjUjBzes).

## Hat Tip

* This [repo](https://github.com/papivot/deploy-TBS-on-vSphere7)