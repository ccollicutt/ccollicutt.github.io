---
layout: post
title:  "Tanzu Application Platform on a Windows Workstation"
categories:
header_image: "/img/tap-on-lap.jpg"
header_permalink: https://unsplash.com/@roadahead_2223
summary: "yep, run TAP on your laptop"

---

# {{ page.title }}

My current favorite VMware document is this:

* [Running Tanzu Application Platform Locally on Your Laptop](https://tanzu.vmware.com/developer/guides/tanzu-application-platform-local-devloper-install/)

If you run through the above how-to document, which is affectionately called TAPonLAP, at the end you will have a functioning [Tanzu Application Platform](https://tanzu.vmware.com/application-platform) (TAP) profile-defined environment to use, and it's all running locally on your personal workstation.

## What is TAP?

TAP is:

>...a modular, application-aware platform that provides a rich set of developer tooling and a prepaved path to production to build and deploy software quickly and securely on any compliant public cloud or on-premises Kubernetes cluster. - [VMware Tanzu](https://tanzu.vmware.com/application-platform)

Using TAP we get all kinds of interesting and useful modular, Kubernetes native components which can work together, in concert with other tools and systems, to abstract way the complexity, and technical debt, of things like Dockerfiles and Kubernetes manifests, while also providing the ability to secure and understand workloads, even across multiple Kubernetes clusters.

It's important to understand that TAP isn't a single binary--it's not a monolithic thing--instead it's a collection of tools that can work together, even across multiple Kubernetes clusters. Some TAP components will be deployed in production clusters that run the apps, other pieces will only be deployed into clusters that build images and compile code, and still more parts only need to be deployed locally for a developers inner loop (if desired). We use the concept of profiles to determine what tools get deployed where. Honestly--it's a new paradigm for Kubernetes based application platforms.

Another thing to keep in mind is that VMware Tanzu is extremely focused on Developer Experience (DX). Because of that focus we care very much about the developers "inner loop"...the things developers do with code before they commit it. With that in mind, the TAPonLAP document is focussed on building a local environment, specifically to meet the needs of that inner loop. I take it a little farther here and deploy most of the TAP components, but this won't be commonplace, unless someone is learning about all of the components.

## My TAPonLAP Environment

My main operating system is Linux (how do you know someone runs Linux on their desktop...just wait and they'll tell you) but I use a Windows workstation for talking to customers with (Zoom, Teams, etc). Because I use Windows when I demonstrate software...well, that software has to run there too. So I've spent a fair amount time running through the TAPonLAP document, using the powershell commands, etc, overall making sure I've got the best local Windows development environment I can.

What I have:

* Windows 10
* Hyper-V enabled
* Minikube
* 32GB main memory, but using 12GB for the minikube instance
* Enough disk (I use a Minikube instance with 80GB or more)
* AMD Ryzen 5 3600 6-Core Processor 

## TAPonLAP

Please note: I'm not going to go through the [entire document](https://tanzu.vmware.com/developer/guides/tanzu-application-platform-local-devloper-install/) and copy and paste the same commands here. However, I will detail some changes I made for my particular situation. So don't expect to be able to follow this blog post and get TAP deployed--instead read the TAPonLAP doc, and this post, and then make your own decisions about how best to deploy TAP locally. Keep in mind it's easy to delete or update TAP and redo the minikube install if needed. This combination of minikube and TAP is a great way to experiment.

## Minikube Start

I use the below. Note that I am adding more disk, I think the default is ~20GB, which should be fine in most situations, but if you run the "full" TAP profile you'll need more resources. 

```
minikube start `
  --kubernetes-version='1.22.8' `
  --cpus='8' --memory='12g' `
  --driver='hyperv' `
  --disk-size='80g'
```

## A Note About Profiles

TAP is modular. It has many components, and not all of them need to be deployed in every situation. The TAPonLAP document shows the "iterate" profile, but in this blog post I use the "full" profile. 

Most people using TAP locally, as part of their inner loop, would not run the full profile. Instead they are iterating on their own code before committing it, and want to make sure it mostly works before the commit, and when the CI system takes over. That is why the "iterate" profile exists.

## TAP Full Profile

The TAPonLAP document shows using the iterative profile. Here's an example full profile. 

>NOTE: This, of course, this will consume more resources that the "iterative" profile.

>NOTE: I'm using "example.com" as the domain, which you may or may not want to do.

>NOTE: You'll have to fill in all the image registry information, users, passwords, etc.

```
profile: full
ceip_policy_disclosed: true # Installation fails if this is set to 'false'

buildservice:
  kp_default_repository: "<some repository>/tap-inner-loop-1-1-full/build-service"
  kp_default_repository_username: ""
  kp_default_repository_password: ""
  tanzunet_username: ""
  tanzunet_password: ""
  enable_automatic_dependency_updates: true

cnrs:
  domain_name: apps.example.com
  domain_template: "{{.Name}}-{{.Namespace}}.{{.Domain}}"
  provider: local

supply_chain: basic

ootb_supply_chain_basic:
  registry:
    server: "<some repository>"
    repository: "tap-inner-loop-1-1-full/supply-chain"
  gitops:
    ssh_secret: ""

learningcenter:
  ingressDomain: "lc.example.com"

tap_gui:
  service_type: ClusterIP
  ingressEnabled: "true"
  ingressDomain: "example.com"
  app_config:
    app:
      baseUrl: http://tap-gui.example.com
    catalog:
      locations:
        - type: url
          target: https://github.com/sample-accelerators/tanzu-java-web-app/blob/main/catalog/catalog-info.yaml
        - type: url
          target: https://github.com/benwilcock/tap-gui-blank-catalog/blob/main/catalog-info.yaml
    backend:
      baseUrl: http://tap-gui.example.com
      cors:
        origin: http://tap-gui.example.com

metadata_store:
  app_service_type: LoadBalancer

grype:
  namespace: "default"
  targetImagePullSecret: "registry-credentials"

# e.g. App Accelerator specific values go under its name
accelerator:
  server:
    service_type: ClusterIP

contour:
  envoy:
    service:
      type: LoadBalancer
```

## Once TAP deploys

We should see the below.

```
PS C:\Windows\system32> tanzu package installed get tap -n tap-install
NAME:                    tap
PACKAGE-NAME:            tap.tanzu.vmware.com
PACKAGE-VERSION:         1.1.0
STATUS:                  Reconcile succeeded
CONDITIONS:              [{ReconcileSucceeded True  }]
USEFUL-ERROR-MESSAGE:
```

## Hostnames

Depending on the profile in use, there may be many hostnames needed to be added to the Windows hosts file. Here's a list of hostnames I use. 

>NOTE: I'm a big fan of our [Learning Center](https://docs.vmware.com/en/Tanzu-Application-Platform/1.1/tap/GUID-learning-center-about.html) tool, more about that in future posts I'm sure, so many of these hostnames are related to that project, which is part of TAP. If you don't deploy it, then you won't need these hostnames. It is part of the "full" profile though. That said, you can exclude packages from deployment.

```
192.168.0.10 tap-gui.example.com tanzu-java-web-app.default.apps.example.com learning-center-guided.lc.example.com learning-center-guided-w01-s001.lc.example.com tanzu-java-web-app-default.apps.example.com learning-center-guided-w01-s001-editor.lc.example.com learning-center-guided-w01-s001-console.lc.example.com learning-center-guided-w01-s001-nginx.lc.example.com learning-center-guided-w01-s001-nginx-via-proxy.lc.example.com learning-center-guided-w01-s001-registry.lc.example.com
```

## Deploying an Application (AKA WOrkload)

Using the Tanzu CLI, which itself is using Kubernetes under the hood (you could simply use the YAML that it shows as well) we can deploy a Java application.

>NOTE: We are not building or managing the container image. A TAP component called the Tanzu Build Service is doing that for us. So no Dockerfiles to manage.

>NOTE: Ensure you "prepare" the dev namespace as per the TAPonLAP document.

```
PS C:\Windows\system32> tanzu apps workload create tanzu-java-web-app `
>>   --git-repo https://github.com/sample-accelerators/tanzu-java-web-app `
>>   --git-branch main `
>>   --type web `
>>   --label app.kubernetes.io/part-of=tanzu-java-web-app `
>>   --label tanzu.app.live.view=true `
>>   --label tanzu.app.live.view.application.name=tanzu-java-web-app `
>>   --annotation autoscaling.knative.dev/minScale=1 `
>>   --namespace $env:TAP_DEV_NAMESPACE `
>>   --yes
Create workload:
←[32m      1 + |---
←[0m←[32m      2 + |apiVersion: carto.run/v1alpha1
←[0m←[32m      3 + |kind: Workload
←[0m←[32m      4 + |metadata:
←[0m←[32m      5 + |  labels:
←[0m←[32m      6 + |    app.kubernetes.io/part-of: tanzu-java-web-app
←[0m←[32m      7 + |    apps.tanzu.vmware.com/workload-type: web
←[0m←[32m      8 + |    tanzu.app.live.view: "true"
←[0m←[32m      9 + |    tanzu.app.live.view.application.name: tanzu-java-web-app
←[0m←[32m     10 + |  name: tanzu-java-web-app
←[0m←[32m     11 + |  namespace: default
←[0m←[32m     12 + |spec:
←[0m←[32m     13 + |  params:
←[0m←[32m     14 + |  - name: annotations
←[0m←[32m     15 + |    value:
←[0m←[32m     16 + |      autoscaling.knative.dev/minScale: "1"
←[0m←[32m     17 + |  source:
←[0m←[32m     18 + |    git:
←[0m←[32m     19 + |      ref:
←[0m←[32m     20 + |        branch: main
←[0m←[32m     21 + |      url: https://github.com/sample-accelerators/tanzu-java-web-app
←[0m
←[32;1mCreated workload "tanzu-java-web-app"
←[0m
```

Above you can see, the command outputs the YAML that is actually deployed. Note the kind.

```
kind: Workload
```

With this command or a few lines of YAML, you can completely manage an application in Kubernetes, from source to running application.

## The Application is Deployed!

The example Java application has now been deployed.

>NOTE: By default the application is running in [knative](https://knative.dev/docs/), which can scale to zero. But, in the above command we told knative that the minimum scale is 1, ie. not to scale to zero, so there will always be at least one pod running.

```
PS C:\Windows\system32> kubectl get pods
NAME                                                   READY   STATUS      RESTARTS   AGE
tanzu-java-web-app-00001-deployment-7fffdb9fcb-2s47s   2/2     Running     0          17m
tanzu-java-web-app-build-1-build-pod                   0/1     Completed   0          19m
tanzu-java-web-app-config-writer-69xhb-pod             0/1     Completed   0          18m
```

If we get the routes we can see the URL to connect to.

```
PS C:\Windows\system32> kubectl get routes
NAME                 URL                                                  READY   REASON
tanzu-java-web-app   http://tanzu-java-web-app-default.apps.example.com   True
```

Curl it:

```
PS C:\Windows\system32> curl.exe http://tanzu-java-web-app-default.apps.example.com
Greetings from Spring Boot + Tanzu!
```

Done (for now)!

## Observe in the TAP GUI

TAP comes with a web GUI, which can be found at http://tap-gui.example.com if using the above "full" profile.

![TAP GUI](/img/tap-on-lap-gui.jpg)

We can start to "drill down" into the running application as well.

![TAP GUI 2](/img/tap-on-lap-gui2.jpg)

We can also visualizae our secure software flow, the secure supply chains that were created by default.

![TAP GUI 3](/img/tap-on-lap-gui3.jpg)

Above we are using the default "source-to-url" supply chain.

```
PS C:\Windows\system32> kubectl get clustersupplychains
NAME                 READY   REASON   AGE
basic-image-to-url   True    Ready    29m
source-to-url        True    Ready    29m
```

## Conclusion

TAP, and it's modular components, are meant to run in many places in many ways, from large clusters, to minikube instances. If TAP is running locally, then the developer can use it to "iterate" on code: to write code, try deploying it, test it out, understand a bit about TAP, and then commit their code at which point TAP can again take over and build the image and run it in production, but of course, not locally, instead on a production Kubernetes cluster (or, more likely, cluster**s**).

This post just brings us through a bit of TAP, gives a few hints, and provides a starting point to understand more about the TAP paradigm.

## Extra Notes

### Don't Forget to Run the Minikube Tunnel

If you forget the below...

```
minikube tunnel
```

or didn't leave it running while installing TAP, then the Ingress load balancer will stay pending and TAP won't deploy completely.

### Kubeconfig into WSL

This might not be the right motion, but I prefer to use Linux to work with Kubernetes as opposed to a powershell...er shell. So here I send the kubeconfig for minikube into my WSL users kube config.

>NOTE: This would, of course, destroy anything existing in your kubeconfig.

>NOTE: This assumes your WSL instance is named "Ubuntu" and that your user's name is "curtis" which is unlikely. :)

```
PS C:\Users\ccollicutt> kubectl config view --flatten > \\wsl$\Ubuntu\home\curtis\.kube\config
```

### Minikube IP

One thing I've noticed is that the minikube IP will changed, say on a Windows OS reboot. So if that happens you'll need to change the IP in the hosts file.

### Minikube Purge

I've had to purge the minikube instance once or twice. For some reason, I had one instance of minikube that was acting very slow. I didn't take the time to try to figure out why it was acting slow. I don't believe it was because of TAP, more likely because of some storage/disk issue.

```
PS C:\Windows\system32> minikube delete --purge
```

### Memory Usage

So far it seems reasonable to run this on my desktop, though again, most of the time one would run a lighter TAP profile than the "full" profile.

![Performance](/img/tap-on-lap-4.jpg)