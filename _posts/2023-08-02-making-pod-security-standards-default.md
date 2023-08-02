---
layout: post
title:  "Making Pod Security Standards the Default in Kubernetes"
categories:
header_image: "/img/enforce-psa.png"
summary: "Security by default"

---

# {{ page.title }}

Vanilla Kubernetes is pretty insecure. It'll basically allow anything. We used to have something called Pod Security Policies to try to reduce the capability space, like reduce some exposure, but that was deprecated. Now we have Pod Security Standards--these are a set of recommendations for securing pods and are managed by the Pod Security Admission Controller. However, by default, no Pod Security Standards are configured, enforcing, etc...but what if we want them enforcing by default on a newly created namespace?

## Using Kyverno to Accomplish Our Goal

One way to accomplish this is to use Kyverno, which, among its features, is the ability to mutate Kubernetes requests. For example, it can add a label to a namespace, such as specifying a Pod Security Standard. Thus, following this model, we can force every new namespace to require a certain security posture by default.

Another way to do this would be to set up a mutating admission controller. There may be other systems that allow us to do something similar. I haven't explored them all yet.

(In fact, if anyone knows how to do this by default with just vanilla Kubernetes, please let me know.)

### Install Kyverno

* Install Kyverno

```
$ helm install kyverno kyverno/kyverno -n kyverno --create-namespace
NAME: kyverno
LAST DEPLOYED: Wed Aug  2 07:47:18 2023
NAMESPACE: kyverno
STATUS: deployed
REVISION: 1
NOTES:
Chart version: 3.0.4
Kyverno version: v1.10.2

Thank you for installing kyverno! Your release is named kyverno.

The following components have been installed in your cluster:
- CRDs
- Admission controller
- Reports controller
- Cleanup controller
- Background controller


⚠️  WARNING: Setting the admission controller replica count below 3 means Kyverno is not running in high availability mode.

💡 Note: There is a trade-off when deciding which approach to take regarding Namespace exclusions. Please see the d
```

Done.

### Make the Restricted Pod Security Standard the Default

Kyverno can mutate a request to create a ns and add a label to it. In this case we're telling it to add the label `pod-security.kubernetes.io/enforce: restricted` so that the pod security admission controller will enforce the Pod Security Standards on the namespace, in this case it will "enforce" the "restricted" profile.

### Configuring Kyverno to Add the Label to Namespaces

Add the ClusterPolicy to Kyverno:

```
cat <<EOF | kubectl apply -f -
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-ns-label
spec:
  rules:
  - name: add-ns-label
    match:
      resources:
        kinds:
        - Namespace
    mutate:
      patchStrategicMerge:
        metadata:
          labels:
            pod-security.kubernetes.io/enforce: restricted
EOF

```

Verify:

```
$ k get clusterpolicy add-ns-label
NAME       	BACKGROUND   VALIDATE ACTION   READY   AGE	MESSAGE
add-ns-label   true     	Audit         	True	7h1m   Ready
```

Good.

## Create a Namespace and Test

* Create a new namespace (the name doesn't matter)

```
kubectl create ns enforcing
```

* Review that the label was added by Kyverno mutating the request for a namespace

```
$ k get ns enforcing -oyaml
apiVersion: v1
kind: Namespace
metadata:
  annotations:
	policies.kyverno.io/last-applied-patches: |
  	add-ns-label.add-ns-label.kyverno.io: added /metadata/labels/pod-security.kubernetes.io~1enforce
  creationTimestamp: "2023-08-02T11:50:14Z"
  labels:
	kubernetes.io/metadata.name: enforcing
	pod-security.kubernetes.io/enforce: restricted
  name: enforcing
  resourceVersion: "11055"
  uid: ce552efc-172e-4aa6-bc6d-13179f73372c
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```

This label was added by Kyverno. This means the pod security admission controller will enforce the restricted Pod Security Standard in this namespace.

```
pod-security.kubernetes.io/enforce: restricted
```

* Create an "insecure" pod in the namespace, just any old pod will do...

```
kubectl run nginx --image=nginx -n enforcing
```

E.g. output:

```
$ k run nginx --image=nginx -n enforcing
Error from server (Forbidden): pods "nginx" is forbidden: violates PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```

The pod was not allowed to be created because it did not meet the Pod Security Standard.

* Create a pod that meets the Pod Security Standard for the "restricted" profile

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx-meets-pod-security-standards
  namespace: enforcing
spec:
  containers:
  - image: nginxinc/nginx-unprivileged
    name: nginx
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
      runAsNonRoot: true
      seccompProfile:
        type: RuntimeDefault
EOF
```

Ok, that worked.

```
$ k get pods
NAME                             	READY   STATUS	RESTARTS   AGE
nginx-meets-pod-security-standards   1/1 	Running   0      	6h26m
```

* Connect to nginx running in the pod by port forwarding

```
kubectl port-forward pod/nginx-meets-pod-security-standards 8080:8080 -n enforcing
```

* Curl it

```
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Works!

## What is the Pod Security Standard "Restricted" Profile?

The Pod Security Standard "restricted" profile is defined in the [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) and is, er, the most restrictive profile.

* Limit types of volumes
* Don't run as root user
* No privilege escalation
* Seccomp profile set to "RuntimeDefault" or "Localhost"
* Drop all capabilities except perhaps add `NET_BIND_SERVICE`

## Conclusion

No one should run vanilla, default Kubernetes in production. No one should run root user pods. I mean we're mostly running web services here, they can listen on port 8080 and don't really need much in the way of permissions. Definitely our namespaces should have security limitations that are only reduced later on if they need to be.

Using Kyverno to do this is one way, there are others.

Ultimately, the way to secure general purpose CPUs is to limit what the workloads can do with them.

## References

* [https://www.jrcomplex.fi/securing-containers-in-kubernetes-with-seccomp/]([https://www.jrcomplex.fi/securing-containers-in-kubernetes-with-seccomp/)
* [https://kubernetes.io/docs/concepts/security/pod-security-standards/]([https://kubernetes.io/docs/concepts/security/pod-security-standards/)



