---
layout: post
title:  "Secure Software Supply Chains and the Tanzu Application Platform"
categories:
header_image: "/img/tap-gui-post.jpg"
summary: ""

---

# {{ page.title }}

If you are a company that makes software, then you have a software supply chain, whether you want one or not. Building software is challenging, even without thinking about where all the underlying dependencies and other software comes from, and what's in it, never mind cataloguing and checksumming it all, and being able to replace it within a few hours. 

This is all hard work, work that many companies spend thousands and thousands of engineering hours on trying to build themselves, often unsuccessfully. Other companies simply don't have the people power--the time, the resources--to even try to implement secure pipelines.

## What's a secure software supply chain?

A "secure software supply chain" (SSSC) is...

>...a fancy way of saying "we know all the components that went into building and deploying this software and trust those components." It also includes the actual CI/CD pipeline that you trust and that's resistant to third parties including malicious code, as we've seen happen in recent years. - [Tanzu Blog](https://tanzu.vmware.com/content/blog/devops-vs-devsecops)

Here are some outcomes organizations are looking for with regards to SSSC:

* "We need to be able to deploy code to our staging and production environments reliably every time"
* "When there is a CVE for one of our applications or dependencies, we need to be able to remediate the problem within 24 hours"
* "We must ensure our software is validated during the build process and that it is built upon known secure images"

If you want to find out more about secure software supply chains, take a look at this [learning path](https://tanzu.vmware.com/developer/learningpaths/secure-software-supply-chain/) provided by VMware Tanzu.

## The Tanzu Application Platform

I've talked about the Tanzu Application Platform (TAP) on this blog before. Suffice it to say that Kubernetes can do more than just containers, we can teach to do anything, and TAP is a way to show Kubernetes how to manage and secure applications; to turn it into more than just a "container orchestration engine". 

With TAP we use Kubernetes as a base platform that we add on to, and then turn into a full blown application platform...a *modular* system that understands how to deploy, manage and secure applications on its own, without having to be told what to do (unless we want to).

## TAP Supply Chains

I've got the Tanzu Application Platform deployed into a single cluster (in this case Minikube running on my [Windows workstation](/2022/04/26/tanzu-application-platform-on-windows-workstation.html)). It's has a couple of software supply chains installed by default.

As you can see, I'm asking the Kubernetes API what it knows about "clustersupplychains", i.e. TAP and its components are NATIVE to Kubernetes--we've taught Kubernetes how to do supply chains (and more).

```
$ k get clustersupplychains.carto.run
NAME                 READY   REASON   AGE
basic-image-to-url   True    Ready    4d23h
source-to-url        True    Ready    4d23h
```

With TAP 1.1 you get a few default supply chains, e.g. basic-image-to-url and source-to-url.

source-to-url is the easiest one to understand. This supply chain does the following:

1. Watch a git repository
2. When there are changes, build a new image using that code (no Dockerfile anywhere)
3. Deploy the new image

This all happens from within Kubernetes, and can be across multiple clusters, each with specific duties. (In this case I just have a single cluster though.)

## ClusterSupplyChains

Let's look at the YAML that defines the ClusterSupplyChain.

>NOTE: I'm trying out the "neat" plugin for kubectl here. It removes some of the extra things you see when pulling the YAML from k8s. A few other things I removed by hand.

```
$ k neat get -- clustersupplychain source-to-url
apiVersion: carto.run/v1alpha1
kind: ClusterSupplyChain
metadata:
  name: source-to-url
spec:
  params:
  - default: main
    name: gitops_branch
  - default: supplychain
    name: gitops_user_name
  - default: supplychain
    name: gitops_user_email
  - default: supplychain@cluster.local
    name: gitops_commit_message
  - default: ""
    name: gitops_ssh_secret
  resources:
  - name: source-provider
    params:
    - name: serviceAccount
      value: default
    - name: gitImplementation
      value: go-git
    templateRef:
      kind: ClusterSourceTemplate
      name: source-template
  - name: deliverable
    params:
    - name: registry
      value:
        repository: tap-inner-loop-1-1-full/supply-chain
        server: somerepo.example.com
    templateRef:
      kind: ClusterTemplate
      name: deliverable-template
  - name: image-builder
    params:
    - name: serviceAccount
      value: default
    - name: clusterBuilder
      value: default
    - name: registry
      value:
        repository: tap-inner-loop-1-1-full/supply-chain
        server: somerepo.example.com
    sources:
    - name: source
      resource: source-provider
    templateRef:
      kind: ClusterImageTemplate
      name: kpack-template
  - images:
    - name: image
      resource: image-builder
    name: config-provider
    params:
    - name: serviceAccount
      value: default
    templateRef:
      kind: ClusterConfigTemplate
      name: convention-template
  - configs:
    - name: config
      resource: config-provider
    name: app-config
    templateRef:
      kind: ClusterConfigTemplate
      name: config-template
  - configs:
    - name: config
      resource: app-config
    name: config-writer
    params:
    - name: serviceAccount
      value: default
    - name: registry
      value:
        repository: tap-inner-loop-1-1-full/supply-chain
        server: somerepo.example.com
    templateRef:
      kind: ClusterTemplate
      name: config-writer-template
  selector:
    apps.tanzu.vmware.com/workload-type: web
```

Now we can take that output and plug it into the [Cartographer live editor](https://cartographer.sh/live-edito), and it will show us a nice diagram which depicts the supply chain flow.

>NOTE: Cartographer is the open source project that underlies some of TAP...it's the k8s native component that ties all the disparate, separate functions together into a pipeline.

![cartographer diagram](/img/carto.jpg)

Here's a larger version. The arrows mean "depends on", as opposed to the directional flow.

![cartographer diagram 2](/img/carto2.jpg)

If you look into the YAML, we can see the first section under resources is `source-provider`.

```
SNIP!
  resources:
  - name: source-provider
    params:
    - name: serviceAccount
      value: default
    - name: gitImplementation
      value: go-git
    templateRef:
      kind: ClusterSourceTemplate
      name: source-template
SNIP!
```

The kind "ClusterSourceTemplate" exists in the cluster.

```
$ k get clustersourcetemplates.carto.run
NAME                       AGE
delivery-source-template   4d23h
source-scanner-template    4d23h
source-template            4d23h
testing-pipeline           4d23h
```

Above we can see there is one called "source-template".

```
$ k neat get -- clustersourcetemplates.carto.run source-template -oyaml
apiVersion: carto.run/v1alpha1
kind: ClusterSourceTemplate
metadata:
  name: source-template
spec:
  params:
  - default: default
    name: serviceAccount
  - default: go-git
    name: gitImplementation
  revisionPath: .status.artifact.revision
  urlPath: .status.artifact.url
  ytt: |
    #@ load("@ytt:data", "data")

    #@ def merge_labels(fixed_values):
    #@   labels = {}
    #@   if hasattr(data.values.workload.metadata, "labels"):
    #@     labels.update(data.values.workload.metadata.labels)
    #@   end
    #@   labels.update(fixed_values)
    #@   return labels
    #@ end

    #@ def param(key):
    #@   if not key in data.values.params:
    #@     return None
    #@   end
    #@   return data.values.params[key]
    #@ end

    ---
    #@ if hasattr(data.values.workload.spec.source, "git"):
    apiVersion: source.toolkit.fluxcd.io/v1beta1
    kind: GitRepository
    metadata:
      name: #@ data.values.workload.metadata.name
      labels: #@ merge_labels({ "app.kubernetes.io/component": "source" })
    spec:
      interval: 1m0s
      url: #@ data.values.workload.spec.source.git.url
      ref: #@ data.values.workload.spec.source.git.ref
      gitImplementation: #@ data.values.params.gitImplementation
      ignore: |
        !.git
      #@ if/end param("gitops_ssh_secret"):
      secretRef:
        name: #@ param("gitops_ssh_secret")
    #@ end

    #@ if hasattr(data.values.workload.spec.source, "image"):
    apiVersion: source.apps.tanzu.vmware.com/v1alpha1
    kind: ImageRepository
    metadata:
      name: #@ data.values.workload.metadata.name
      labels: #@ merge_labels({ "app.kubernetes.io/component": "source" })
    spec:
      serviceAccount: #@ data.values.params.serviceAccount
      interval: 1m0s
      image: #@ data.values.workload.spec.source.image
    #@ end
```

A lot of the above YAML is a "template" built with Carvel's YAML templating tool, ytt, which may look a little unusual to those who haven't seen ytt before.

From this section of the YAML, it's somewhat obvious that this is a "if/then" template, and IF the source comes from GIT, then use "kind: GitRepository". (And if it's an IMAGE then use ImageRepository.)

```
SNIP!
    #@ if hasattr(data.values.workload.spec.source, "git"):
    apiVersion: source.toolkit.fluxcd.io/v1beta1
    kind: GitRepository
SNIP!
```

The demo app, tanzu-java-web-app, is using a git repository, as defined in its YAML manifest / k8s object. Note the "source.git" section.

```
$ k neat get -- workloads.carto.run tanzu-java-web-app  -oyaml
apiVersion: carto.run/v1alpha1
kind: Workload
SNIP!
  source:
    git:
      ref:
        branch: main
      url: https://github.com/sample-accelerators/tanzu-java-web-app
```

So there must be a manifest for that...yep.

```
$ k get gitrepositories.source.toolkit.fluxcd.io
NAME                 URL                                                         READY   STATUS
                                   AGE
tanzu-java-web-app   https://github.com/sample-accelerators/tanzu-java-web-app   True    Fetched revision: main/f5cf96ef23f3fddba94616112dfad882882aabe4   4d23h
```

Note the above is "...fluxcd.io". TAP is using parts of the open source flux project, in this case specifically the flux capability to get code from a git repository.

We can continue this k8s sleuthing to follow the entire software supply chain. So far we've just looked at how source code is retrieved. 

An important point is that this is all programmable, so much so that we can simply use the Cartographer live editor and paste in the YAML definition and it can easily produce an image.

This is also what is visualized in the TAP web interface.

![TAP GUI supply chain](/img/tap-on-lap-gui3.jpg)

## Conclusion

Here we've sleuthed through one stage of one of the supply chains, and looked at how we can visualize the chains with Cartographer's live editor and the TAP GUI. In the next post we'll create a custom supply chain.

