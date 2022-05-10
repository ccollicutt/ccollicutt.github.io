---
layout: post
title:  "Creating CUSTOM Secure Software Supply Chains with Tanzu"
categories:
header_image: "/img/tap-image-scan.jpg"
summary: "test"

---

# {{ page.title }}

In the [last post](/2022/05/10/creating-secure-software-supply-chains-with-tanzu.html) I looked at creating secure software supply chains with the Tanzu Application Platform. In that post I used a default supply chain. But what if we wanted to create our own, custom supply chain?

## Creating a Custom Supply Chain

Ok, so we have two supply chains by default.

```
$ k get clustersupplychains.carto.run
NAME                 READY   REASON   AGE
basic-image-to-url   True    Ready    5d2h
source-to-url        True    Ready    5d2h
```

The reason we have these two is that when TAP was installed the TAP values file was configured with the below.

```
supply_chain: basic
```

There are other options to provide "out of the box templates", such as the below, which will provide other OOTB chains. That said, we expect that most organizations will build their own supply chains using our platform and its various building blocks.

```
supply_chain: testing_scanning
```

For now, for the purposes of this blog post, I start with the two basic chains, and I'd like to add another *custom* chain, an extension of source-to-url.

## Creating a Custom Supply Chain

Let's say my goal is:

* Take the "source-to-url" chain, create a new chain, and add "image scanning" to it, so that the image that is created is also scanned to see if there are any CVEs

I'm going to grab the "source-to-url" chain and edit it.

```
$ k neat get  -- clustersupplychains.carto.run source-to-url
apiVersion: carto.run/v1alpha1
kind: ClusterSupplyChain
metadata:
  annotations:
    kapp.k14s.io/identity: v1;/carto.run/ClusterSupplyChain/source-to-url;carto.run/v1alpha1
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

Next, let's change it so that it looks like the below.

```
apiVersion: carto.run/v1alpha1
kind: ClusterSupplyChain
metadata:
  name: source-to-url-with-image-scan
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
  #scan-image
  - name: scan-image
    images:
    - name: image
      resource: image-builder
    templateRef:
      kind: ClusterImageTemplate
      name: image-scanner-template      
  - images:
    - name: image
      resource: scan-image
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
    apps.tanzu.vmware.com/workload-type: web-image-scan
```

>NOTE: The selector is now "web-image-scan".

At this point we have a diff something like this. All that's happened is the insertion of the "scan-image" block into the chain, and changed the name to make it unique.

```bash
$ git diff source-to-url-original.yml curtis-source-to-url-with-image-scan.yml
diff --git a/source-to-url-original.yml b/curtis-source-to-url-with-image-scan.yml
index 06e8dff..3c22bac 100644
--- a/source-to-url-original.yml
+++ b/curtis-source-to-url-with-image-scan.yml
@@ -1,12 +1,7 @@
 apiVersion: carto.run/v1alpha1
 kind: ClusterSupplyChain
 metadata:
-  annotations:
-    kapp.k14s.io/identity: v1;/carto.run/ClusterSupplyChain/source-to-url;carto.run/v1alpha1
-  labels:
-    kapp.k14s.io/app: "1651760734110088811"
-    kapp.k14s.io/association: v1.4e1a1027543b1d663294132ebfdd4f33
-  name: source-to-url
+  name: source-to-url-with-image-scan
 spec:
   params:
   - default: main
@@ -54,9 +49,17 @@ spec:
     templateRef:
       kind: ClusterImageTemplate
       name: kpack-template
-  - images:
+  #scan-image
+  - name: scan-image
+    images:
     - name: image
       resource: image-builder
+    templateRef:
+      kind: ClusterImageTemplate
+      name: image-scanner-template      
+  - images:
+    - name: image
+      resource: scan-image
     name: config-provider
     params:
     - name: serviceAccount
@@ -86,4 +89,4 @@ spec:
       kind: ClusterTemplate
       name: config-writer-template
   selector:
-    apps.tanzu.vmware.com/workload-type: web
\ No newline at end of file
+    apps.tanzu.vmware.com/workload-type: web-image-scan
\ No newline at end of file
```

The new chain looks like this in Cartographer's live editor. As you can see, there is now "scan-image" in the chain.

![image scan pipeline](/img/carto3.jpg)

Which, of course, is different from the non-image scan version. Note how there is no "scan image" box.

![cartographer diagram 2](/img/carto2.jpg)


Checkout the image scanner template.

```
$ k neat get -- clusterimagetemplates.carto.run image-scanner-template -oyaml
apiVersion: carto.run/v1alpha1
kind: ClusterImageTemplate
metadata:
  annotations:
    kapp.k14s.io/identity: v1;/carto.run/ClusterImageTemplate/image-scanner-template;carto.run/v1alpha1
    kapp.k14s.io/original-diff-md5: c6e94dc94aed3401b5d0f26ed6c0bff3
  labels:
    kapp.k14s.io/app: "1651760721125747499"
    kapp.k14s.io/association: v1.7d6419553fe4d29522bcc6dc11d61feb
  name: image-scanner-template
spec:
  imagePath: .status.compliantArtifact.registry.image
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

    apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
    kind: ImageScan
    metadata:
      name: #@ data.values.workload.metadata.name
      labels: #@ merge_labels({ "app.kubernetes.io/component": "image-scan" })
    spec:
      registry:
        image: #@ data.values.image
      scanTemplate: private-image-scan-template
      scanPolicy: scan-policy
```

If we look at the above definition, we see that it's using "private-image-scan-template" of kind "ImageScan".

Let's look at those.

```
$ k get scantemplates.scanning.apps.tanzu.vmware.com
NAME                          AGE
blob-source-scan-template     5d2h
private-image-scan-template   5d2h
public-image-scan-template    5d2h
public-source-scan-template   5d2h
```

Let's look at the private scan template.

```
$ k neat get -- scantemplates.scanning.apps.tanzu.vmware.com private-image-scan-template -oyaml
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ScanTemplate
metadata:
  name: private-image-scan-template
  namespace: default
spec:
  template:
    containers:
    - args:
      - -c
      - ./image/copy-docker-config.sh /secret-data && ./image/scan-image.sh /workspace
        scan.xml true
      command:
      - /bin/bash
      image: registry.tanzu.vmware.com/tanzu-application-platform/tap-packages@sha256:d3a8f3cae0db15e416e805dc598223f93059c3a295cbf33f1409bc6cb9a9709c
      imagePullPolicy: IfNotPresent
      name: scanner
      resources:
        limits:
          cpu: 1000m
        requests:
          cpu: 250m
          memory: 128Mi
      volumeMounts:
      - mountPath: /.docker
        name: docker
        readOnly: false
      - mountPath: /workspace
        name: workspace
        readOnly: false
      - mountPath: /secret-data
        name: registry-cred
        readOnly: true
    imagePullSecrets:
    - name: scanner-secret-ref
    restartPolicy: Never
    securityContext:
      runAsNonRoot: true
    volumes:
    - name: docker
    - name: workspace
    - name: registry-cred
      secret:
        secretName: registry-credentials
```

To use this we'll need a scan policy.

## Scan Policy

Next we need a scan policy. 

>NOTE: We are only looking for "Critical" vulnerabilities. Those will fail, everything else will pass.

```
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ScanPolicy
metadata:
  name: scan-policy
spec:
  regoFile: |
    package policies

    default isCompliant = false

    # Accepted Values: "Critical", "High", "Medium", "Low", "Negligible", "UnknownSeverity"
    violatingSeverities := ["Critical"]
    ignoreCVEs := []

    contains(array, elem) = true {
      array[_] = elem
    } else = false { true }

    isSafe(match) {
      fails := contains(violatingSeverities, match.Ratings.Rating[_].Severity)
      not fails
    }

    isSafe(match) {
      ignore := contains(ignoreCVEs, match.Id)
      ignore
    }

    isCompliant = isSafe(input.currentVulnerability)
```

That needs to be installed.

```
$ k create -f image-scan-policy.yml
```

Now it's available to use.

## Install and Use

Let's load that new, custom supply chain into TAP/k8s.

```
$ k create -f curtis-source-to-url-with-image-scan.yml
clustersupplychain.carto.run/source-to-url-with-image-scan created
```

Voila:

```
$ k get clustersupplychains.carto.run
NAME                            READY   REASON   AGE
basic-image-to-url              True    Ready    5d2h
source-to-url                   True    Ready    5d2h
source-to-url-with-image-scan   True    Ready    19s
```

Now to deploy the app. 

>NOTE: I'm flipping between an Ubuntu WSL terminal and a Powershell terminal. Here I'm using Powershell to run the tanzu CLI. Note the type is "web-image-scan".

```
$Env:TAP_DEV_NAMESPACE = "default"
tanzu apps workload create tanzu-java-web-app-with-image-scan `
  --git-repo https://github.com/sample-accelerators/tanzu-java-web-app `
  --git-branch main `
  --type web-image-scan `
  --label app.kubernetes.io/part-of=tanzu-java-web-app-with-image-scan `
  --label tanzu.app.live.view=true `
  --label tanzu.app.live.view.application.name=tanzu-java-web-app-with-image-scan `
  --annotation autoscaling.knative.dev/minScale=1 `
  --namespace $env:TAP_DEV_NAMESPACE `
  --yes
```

I can check the results of the scan.

```
$ k logs scan-tanzu-java-web-app-with-image-scan-qr9q4--1-9b4w9 | grep severity | sort | uniq -c
     27               <v:severity>Low</v:severity>
      8               <v:severity>Medium</v:severity>
```

Many low, a few medium.

Now we have imagescans:

```
$ k get imagescans.scanning.apps.tanzu.vmware.com
NAME                                 PHASE       SCANNEDIMAGE
                                                                                                            AGE   CRITICAL   HIGH   MEDIUM   LOW   UNKNOWN   CVETOTAL
tanzu-java-web-app-with-image-scan   Completed   somerepo.example.com/tap-inner-loop-1-1-full/supply-chain/tanzu-java-web-app-with-image-scan-default@sha256:bb0da26d42537abaa7a7f02afac8eb77387c42524fbd413a265d716934ec2f4c   20m   0          0      3        12    0         15
```

The app is up and running.

```
PS C:\Users\curtis> curl.exe http://tanzu-java-web-app-with-image-scan-default.apps.example.com
Greetings from Spring Boot + Tanzu!
```
## Conclusion

At this point we've created a custom supply chain by adding image scanning to the default source-to-url chain.

This is a simple example, but you can see how powerful, and modular, TAP is.

## Additional Links and Resources

* [Tanzu Application Platform’s OOTB Supply Chain with Testing and Scanning Events](https://dodd-pfeffer.medium.com/tanzu-application-platforms-ootb-supply-chain-with-testing-and-scanning-events-cfc0d50506f7)
* [Deep-dive on Tanzu Application Platform’s OOTB Supply Chain](https://dodd-pfeffer.medium.com/deep-dive-on-tanzu-application-platforms-ootb-supply-chain-ac8a929d2e43)
* [SSSC learning path](https://tanzu.vmware.com/developer/learningpaths/secure-software-supply-chain/) provided by VMware Tanzu.