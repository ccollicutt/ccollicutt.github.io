---
layout: post
title:  "Tanzu Application Platform and knative"
categories:
header_image: "/img/knative.jpg"
summary: "With Tanzu's TAP you get access to knative...and much more!"

---

# {{ page.title }}

## tl;dr

The point of this post is to show that knative is part of The [Tanzu Application Platform](https://tanzu.vmware.com/application-platform), AKA TAP, and one can use knative outside of TAP, directly with the kn CLI, if desired. (Though, in the real world you would probably use the rest of TAP as well, but it's modular so you don't *have* to.)

In this post TAP has been deployed into a GKE cluster (yep, a GKE cluster). TAP includes knative, so in this post I'll deploy a simple demo app into the GKE cluster and that deployment will be done via the knative CLI. Serverless here we come!

## GKE Cluster

First, as I mentioned, I've got a Google Kubernetes Cluster.

```
$ k get nodes
NAME                                       STATUS   ROLES    AGE    VERSION
gke-gke-tap-1-default-pool-4e593ad2-9dk4   Ready    <none>   4d2h   v1.22.3-gke.700
gke-gke-tap-1-default-pool-4e593ad2-m8qs   Ready    <none>   4d2h   v1.22.3-gke.700
gke-gke-tap-1-default-pool-8dc7b056-8g1l   Ready    <none>   4d2h   v1.22.3-gke.700
gke-gke-tap-1-default-pool-8dc7b056-drtd   Ready    <none>   4d2h   v1.22.3-gke.700
gke-gke-tap-1-default-pool-bb825ba8-3pgz   Ready    <none>   4d2h   v1.22.3-gke.700
gke-gke-tap-1-default-pool-bb825ba8-6v55   Ready    <none>   4d2h   v1.22.3-gke.700
```

It has a bunch of nodes and is running the full TAP "profile".

## What is the Tanzu Application Platform?

>VMware Tanzu Application Platform is a modular, application-aware platform that provides a rich set of developer tooling and a prepaved path to production to build and deploy software quickly and securely on any compliant public cloud or on-premises Kubernetes cluster.

TAP is a set of modular components which extend Kubernetes, making it easier and more secure to use. It's important to understand that underlying TAP is still Kubernetes--all the pieces of TAP are "native" Kubernetes which means they become part of the Kubernetes API. The way we interact with TAP is through the Kubernetes API, though that will often be hidden away by pipelines and developer inner loop tooling. (No one should have to use kubectl, but you can of course.)

For an example of extending Kubernetes, once you've deployed TAP you have an image resource, which, in plain old vanilla Kubernetes, doesn't exist. (NOTE: I believe the power of Kubernetes is not so much in orchestrating containers, instead the fact that it is a platform to *build other platforms on top of*, which is precisely what TAP is...it's even in the name!).

Below I have a couple of container images represented in Kubernetes for a demo application for Spring called Pet Clinic, which can be deployed via knative through TAP. knative is part of what VMware Tanzu calls [Cloud Native Runtimes](https://docs.vmware.com/en/Cloud-Native-Runtimes-for-VMware-Tanzu/1.0/tanzu-cloud-native-runtimes-1-0/GUID-cnr-overview.html)

Here I ask what Kubernetes knows about container images.

```
$ k get images
NAME                              IMAGE
pet-clinic-00001-cache-workload   <registry>/tap-beta-4/supply-chain/pet-clinic-default@sha256:3b4ef38a43d464750d63ca0226c67ae59fdf990efe01c37ae88e8e10d2f574e8
pet-clinic-00002-cache-workload   <registry>/tap-beta-4/supply-chain/pet-clinic-default@sha256:edfeabd87ee782f06510a0f1bc984a6134ae923121a6437cd5f401c59ff815de
```

>NOTE: The images resource is provided by the Tanzu Build Service (also part of TAP), which itself is built on the open source projects [kpack](https://github.com/pivotal/kpack) and [Paketo](https://paketo.io/) and years of history and experience with Cloud Foundry and Buildpacks.

But that's just an example of how we build a platform on Kubernetes, in this post I'll try to stay focussed on knative.

## Cloud Native Runtimes and knative

[Cloud Native Runtimes](https://docs.vmware.com/en/Cloud-Native-Runtimes-for-VMware-Tanzu/1.0/tanzu-cloud-native-runtimes-1-0/GUID-cnr-overview.html) is VMware's product to provide various ways of running applications in Kubernetes. Currently there is only the single runtime, knative, but more will be added over time.

So what's knative? It's an open source project that makes developers more productive by abstracting away some of the complexity of Kubernetes.

> Knative components build on top of Kubernetes...by codifying the best practices shared by successful real-world implementations, Knative solves the "boring but difficult" parts of deploying and managing cloud native services so you don't have to. - [knative website](https://knative.dev/docs/)

knative provides:

1. Serverless - "Run serverless containers on Kubernetes with ease, Knative takes care of the details of networking, autoscaling (even to zero), and revision tracking. You just have to focus on your core logic."
2. Eventing - "Universal subscription, delivery, and management of events. Build modern apps by attaching compute to a data stream with declarative event connectivity and developer-friendly object model."

I'd like to look specifically at just running a serverless app in knative, which in this case is easily provided/installed into Kubernetes by TAP. (That said, eventing is clearly important to modern applications, so it's important to keep in mind that knative satisfies that need as well.)

## Deploying an App into knative

In our GKE cluster we have many packages from TAP installed, including knative, aka Cloud Native Runtimes. That means we can run serverless workloads!

```
$ tanzu package installed list --namespace tap-install
\ Retrieving installed packages...
  NAME                      PACKAGE-NAME                                        PACKAGE-VERSION  STATUS               
  accelerator               accelerator.apps.tanzu.vmware.com                   1.0.0            Reconcile succeeded  
  api-portal                api-portal.tanzu.vmware.com                         1.0.8            Reconcile succeeded  
  appliveview               run.appliveview.tanzu.vmware.com                    1.0.1            Reconcile succeeded  
  appliveview-conventions   build.appliveview.tanzu.vmware.com                  1.0.1            Reconcile succeeded  
  buildservice              buildservice.tanzu.vmware.com                       1.4.2            Reconcile succeeded  
  cartographer              cartographer.tanzu.vmware.com                       0.1.0            Reconcile succeeded  
  cert-manager              cert-manager.tanzu.vmware.com                       1.5.3+tap.1      Reconcile succeeded  
  cnrs                      cnrs.tanzu.vmware.com                               1.1.0            Reconcile succeeded  
  contour                   contour.tanzu.vmware.com                            1.18.2+tap.1     Reconcile succeeded  
  conventions-controller    controller.conventions.apps.tanzu.vmware.com        0.5.0            Reconcile succeeded  
  developer-conventions     developer-conventions.tanzu.vmware.com              0.5.0-build.1    Reconcile succeeded  
  fluxcd-source-controller  fluxcd.source.controller.tanzu.vmware.com           0.16.0           Reconcile succeeded  
  grype                     grype.scanning.apps.tanzu.vmware.com                1.0.0            Reconcile succeeded  
  image-policy-webhook      image-policy-webhook.signing.apps.tanzu.vmware.com  1.0.0            Reconcile succeeded  
  learningcenter            learningcenter.tanzu.vmware.com                     0.1.0            Reconcile succeeded  
  learningcenter-workshops  workshops.learningcenter.tanzu.vmware.com           0.1.0            Reconcile succeeded  
  metadata-store            metadata-store.apps.tanzu.vmware.com                1.0.1            Reconcile succeeded  
  ootb-delivery-basic       ootb-delivery-basic.tanzu.vmware.com                0.5.1            Reconcile succeeded  
  ootb-supply-chain-basic   ootb-supply-chain-basic.tanzu.vmware.com            0.5.1            Reconcile succeeded  
  ootb-templates            ootb-templates.tanzu.vmware.com                     0.5.1            Reconcile succeeded  
  scanning                  scanning.apps.tanzu.vmware.com                      1.0.0            Reconcile succeeded  
  service-bindings          service-bindings.labs.vmware.com                    0.6.0            Reconcile succeeded  
  services-toolkit          services-toolkit.tanzu.vmware.com                   0.5.0            Reconcile succeeded  
  source-controller         controller.source.apps.tanzu.vmware.com             0.2.0            Reconcile succeeded  
  spring-boot-conventions   spring-boot-conventions.tanzu.vmware.com            0.3.0            Reconcile succeeded  
  tap                       tap.tanzu.vmware.com                                1.0.0            Reconcile succeeded  
  tap-gui                   tap-gui.tanzu.vmware.com                            1.0.1            Reconcile succeeded  
  tap-telemetry             tap-telemetry.tanzu.vmware.com                      0.1.2            Reconcile succeeded  
  tekton-pipelines          tekton.tanzu.vmware.com                             0.30.0           Reconcile succeeded
  ```

That's a lot of stuff. Those pieces are currently part of the Tanzu Application Platform. But lets just use a single piece: knative.

First I've got the `kn` CLI.

```
$ which kn
/usr/local/bin/kn
```

Next I'll create a namespace (and switch to it).

```
$ kubectl create namespace cnr-demo
```

Then I'll setup a reg secret because my registry requires authentication.

```
$ kubectl create secret docker-registry registry-credentials \
  --docker-server=<redacted> \
  --docker-email=<redacted> \
  --docker-username=<redacted> \
  --docker-password=<redacted>
```

Allow the namespace's default SA to use it...

```
$ kubectl patch serviceaccount default -p "{\"imagePullSecrets\": [{\"name\": \"registry-credentials\"}]}"
```

Now deploy a workload, pointing to an image that resides in the registry I previously setup secret credentials for.

```
$ kn service create hello-yeti -n cnr-demo \
  --image <registry>/hello-yeti --env TARGET='hello-yeti'
```

Output:

```
$ kn service create hello-yeti -n cnr-demo --image <registry>/hello-yeti --env TARGET='hello-yeti'
Creating service 'hello-yeti' in namespace 'cnr-demo':

  0.043s The Route is still working to reflect the latest desired specification.
  0.091s ...
  0.148s Configuration "hello-yeti" is waiting for a Revision to become ready.
  5.639s ...
  5.712s Ingress has not yet been reconciled.
  5.756s Waiting for Envoys to receive Endpoints data.
  6.108s Waiting for load balancer to be ready
  6.329s Ready to serve.

Service 'hello-yeti' created to latest revision 'hello-yeti-00001' is available at URL:
http://hello-yeti.cnr-demo.example.com
```

Once that deploys there will be a pod running.

>NOTE: The deployment will scale to zero if it's not being used, something to remember.

```
$ k get pods
NAME                                          READY   STATUS    RESTARTS   AGE
hello-yeti-00001-deployment-6f49f84f5-z6lgq   2/2     Running   0          13s
```

List the knative deployments/services.

```
$ kn service list
NAME         URL                                      LATEST             AGE     CONDITIONS   READY   REASON
hello-yeti   http://hello-yeti.cnr-demo.example.com   hello-yeti-00001   3m48s   3 OK / 3     True   
```

Curl the app.

>NOTE: TAP also deploys an ingress controller for you. To connect to the knative service we'll need the IP of the Kubernetes loadbalancer for the ingress service.

```
$ export LB=`kubectl get services -n tanzu-system-ingress envoy -o jsonpath="{.status.loadBalancer.ingress[0].ip}"`
$ curl -H "Host: hello-yeti.cnr-demo.example.com" $LB
```

Output:

```
$ curl -H "Host: hello-yeti.cnr-demo.example.com" $LB
              ______________
            /               \
           |   hello from    |
           |  cloud native   |
           |    runtimes     |     .xMWxw.
            \______________\ |   wY     Ym.
                            \|  C  ,  ,   O
                                 \  ww   /.
                               ..x       x..
                              .x   wwwww    x.
                             .x               x.
                             x   \         /   x
                             Y   Y         Y   Y
                              wwv    x      vww
                                \    /\    /
                                :www:  :www:

```

If the application isn't accessed for a while it'll scale to zero.

```
$ k get pods
No resources found in cnr-demo namespace.
```

If we try to hit the URL again, the pods will be restarted automatically by knative.

So, that was pretty easy. Of course it's a simple demo application which lives in an existing container image, but the point of this post was to illustrate that by deploying TAP we have access to knative, and we can even use knative outside of the TAP workflow if we want, or deploy it outside of TAP as well. knative is a major part of VMware Tanzu.

## Conclusion

The Tanzu Application Platform gives you cutting edge tools like serverless via knative. Now, the point of TAP isn't necessarily to just directly use knative like this, but I wanted to show that knative is indeed part, an integral part, of TAP, that you can use if you want to (by default most TAP demos will use knative) but you don't have to use it.

Another thing to keep in mind in the context of this post, is exactly where did that hello-yeti container image come from? What if we want to deploy our own code? Not surprisingly, I'll look into that in the next post in this series. 

Thanks for reading!