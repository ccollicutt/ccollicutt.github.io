---
layout: post
title:  "Restarting Kubernetes Pods When There Are New Secrets With Reloader"
categories:
header_image: "/img/reloader-post.png"
summary: "New secrets, who dis?"

---

# {{ page.title }}

I will tell you a secret—no, a story. Say, at some point, I had a Kubernetes webhook admission controller that I wrote and deployed, and then the TLS certificate was automatically (nice!) renewed by cert-manager, but the pod wasn't restarted, so it still had the old certificate, and now all Kubernetes deployments failed. That is indeed a story, perhaps a sad one. I had this shiny new cert, but no one was using it. Say I wanted to fix that. One way would be with [Reloader](https://github.com/stakater/Reloader).

## Reloader

>*Reloader can watch changes in ConfigMap and Secret and do rolling upgrades on Pods with their associated DeploymentConfigs, Deployments, Daemonsets Statefulsets and Rollouts.* - [Reloader](https://github.com/stakater/Reloader)

## Install Reloader

First add the repo.

```
$ helm repo add stakater https://stakater.github.io/stakater-charts
$ helm repo update
```

Create a namespace.

```
$ k create ns reloader
namespace/reloader created
$ kn reloader 
✔ Active namespace is "reloader"
```

Install reloader.

```
$ helm install reloader stakater/reloader
NAME: reloader
LAST DEPLOYED: Thu Nov 23 09:36:22 2023
NAMESPACE: reloader
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
- For a `Deployment` called `foo` have a `ConfigMap` called `foo-configmap`. Then add this annotation to main metadata of your `Deployment`
  configmap.reloader.stakater.com/reload: "foo-configmap"

- For a `Deployment` called `foo` have a `Secret` called `foo-secret`. Then add this annotation to main metadata of your `Deployment`
  secret.reloader.stakater.com/reload: "foo-secret"

- After successful installation, your pods will get rolling updates when a change in data of configmap or secret will happen.
```

Now we've got pods.

```
$ k get pods
NAME                                 READY   STATUS    RESTARTS   AGE
reloader-reloader-64df699b8d-tm5rn   1/1     Running   0          3m4s
```

Nice and easy. Thanks Helm!

## Simple Test

Create a secret.

```
kubectl create secret generic foo-secret --from-literal=key1=bar
```

Create a cert-manager certificate. (Of course you need cert-manager installed.)

```
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: foo-certs
  namespace: foo
spec:
  secretName: foo-certs
  issuerRef:
    name: kubeadm-ca
    kind: ClusterIssuer
  duration: 24h  # Validity period of the certificate
  renewBefore: 12h 
  commonName: foo.foo.svc.cluster.local
  dnsNames:
    - foo.foo.svc.cluster.local
    - foo.foo.svc
EOF
``` 

Use that secret in a deployment. Note the annotation for Reloader. We're mounting the secret in `/etc/foo` and certificates `/etc/certs`.

```
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo
  labels:
    app: foo
  annotations:
    secret.reloader.stakater.com/reload: "foo-secret,foo-certs"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: foo
  template:
    metadata:
      labels:
        app: foo
    spec:
      containers:
      - name: my-container
        image: nginx
        volumeMounts:
        - name: secret-volume
          mountPath: "/etc/foo"
          readOnly: true
        - name: certs
          mountPath: "/etc/certs"
          readOnly: true
      volumes:
      - name: secret-volume
        secret:
          secretName: foo-secret
      - name: certs
        secret:
          secretName: foo-certs
EOF
```

Recreate the secret and check the logs of reloader.

```
$ kubectl create secret generic foo-secret --from-literal=key1=foo --dry-run=client -o yaml | kubectl apply -f -
Warning: resource secrets/foo-secret is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
secret/foo-secret configured
```

Reloader logs. It has noticed the secret update and restarted the pod.

```
$ k logs -n reloader reloader-reloader-64df699b8d-tm5rn 
time="2023-11-23T14:36:25Z" level=info msg="Environment: Kubernetes"
time="2023-11-23T14:36:25Z" level=info msg="Starting Reloader"
time="2023-11-23T14:36:25Z" level=warning msg="KUBERNETES_NAMESPACE is unset, will detect changes in all namespaces."
time="2023-11-23T14:36:25Z" level=info msg="created controller for: configMaps"
time="2023-11-23T14:36:25Z" level=info msg="Starting Controller to watch resource type: configMaps"
time="2023-11-23T14:36:25Z" level=info msg="created controller for: secrets"
time="2023-11-23T14:36:25Z" level=info msg="Starting Controller to watch resource type: secrets"
time="2023-11-23T15:18:53Z" level=info msg="Changes detected in 'foo-secret' of type 'SECRET' in namespace 'foo', Updated 'foo' of type 'Deployment' in namespace 'foo'"
```

New pod should be starting.

```
$ k get pods
NAME                   READY   STATUS        RESTARTS   AGE
foo-5c67d96557-s6cj2   1/1     Running       0          18s
foo-75cb458f7d-xcszx   1/1     Terminating   0          2m30s
```

Now it's got the new secret.

```
$ k exec -it foo-5c67d96557-s6cj2 -- cat /etc/foo/key1
foo
```

Boom.

## Certificates

Above we crated a certificate with only 24 hours of validity that should renew after 12 hours. So when it's renewed, there will be a new version of the secret, and reloader will restart the pod. Let's see.

```
$ k logs -n reloader reloader-reloader-7f4859f649-6cvqt 
time="2023-11-23T16:03:57Z" level=info msg="Environment: Kubernetes"
time="2023-11-23T16:03:57Z" level=info msg="Starting Reloader"
time="2023-11-23T16:03:57Z" level=warning msg="KUBERNETES_NAMESPACE is unset, will detect changes in all namespaces."
time="2023-11-23T16:03:57Z" level=info msg="created controller for: configMaps"
time="2023-11-23T16:03:57Z" level=info msg="Starting Controller to watch resource type: configMaps"
time="2023-11-23T16:03:57Z" level=info msg="created controller for: secrets"
time="2023-11-23T16:03:57Z" level=info msg="Starting Controller to watch resource type: secrets"
time="2023-11-23T16:06:18Z" level=info msg="Changes detected in 'foo-secret' of type 'SECRET' in namespace 'foo', Updated 'foo' of type 'Deployment' in namespace 'foo'"
time="2023-11-24T04:44:56Z" level=info msg="Changes detected in 'foo-certs' of type 'SECRET' in namespace 'foo', Updated 'foo' of type 'Deployment' in namespace 'foo'"
```

Looking at cert-manager logs we see:
  
```
I1124 04:44:56.006536       1 trigger_controller.go:194] "cert-manager/certificates-trigger: Certificate must be re-issued" key="foo/foo-certs" reason="Renewing" message="Renewing certificate as renewal was scheduled at 2023-11-24 04:44:56 +0000 UTC"
SNIP!
I1124 04:44:56.636293       1 conditions.go:263] Setting lastTransitionTime for CertificateRequest "foo-certs-jk5sq" condition "Ready" to 2023-11-24 04:44:56.636261134 +0000 UTC m=+4380108.430326366
```

Right, so the secret was updated. Let's see if the pod was restarted.

```
$ k get pods
NAME                  READY   STATUS    RESTARTS   AGE
foo-746699dd7-kr99d   1/1     Running   0          6h43m
$ k describe pod foo-746699dd7-kr99d | grep -i started
      Started:      Thu, 23 Nov 2023 23:44:59 -0500
```

That time converts to 04:44:59 UTC, which is when the secret was updated. So it was restarted. This is great, so when a new certificate is issued, the pod will be restarted and mount the new secret and have access to the new certificate and key.

There's a reloader annototioant as well.

```
$ k get pods -oyaml | grep reloader
      reloader.stakater.com/last-reloaded-from: '{"type":"SECRET","name":"foo-certs","namespace":"foo","hash":"94af434fda756e922affdd1c43d723b26f196f3e","containerRefs":["my-container"],"observedAt":1700801096}'
```

## Conclusion

Personally, I would think that this kind of thing would be automatic, but it's not. So this is a good way to make sure that your pods are restarted when there are new secrets.

Kubernetes is a framework, and you have to pull in a lot of "libraries," such as Reloader.