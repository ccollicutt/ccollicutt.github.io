---
layout: post
title:  "Cert Manager's CA Injector and Validating Webhooks"
categories:
header_image: "/img/ca-bundle-injection.jpg"
summary: "Where can I get the CA? Oh, Cert Manager"

---

# {{ page.title }}

I've been working on a simple validating webhook for Kubernetes. More on that later. However, one of the things that you need to provide when you create the Kubernetes manifest for a validating webhook is the CA bundle that the kube-api can use to validate the webhook. But...where does that come from? How do we get it into the manifest?

Here's the Kubernetes docs example of a validating webhook:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"
webhooks:
- name: "pod-policy.example.com"
  rules:
  - apiGroups:   [""]
	apiVersions: ["v1"]
	operations:  ["CREATE"]
	resources:   ["pods"]
	scope:   	"Namespaced"
  clientConfig:
	service:
  	namespace: "example-namespace"
  	name: "example-service"
	caBundle: <CA_BUNDLE>
  admissionReviewVersions: ["v1"]
  sideEffects: None
  timeoutSeconds: 5
```

Note the `CA_BUNDLE` value.

OK, so I have to provide that. But...I guess I just create that manually? At first I was doing the below.

>NOTE: In my homelab I use the kubeadm generated certs, which live in `/etc/kubernetes/pki/`. You probably aren't doing that--the point is that you need to get the CA bundle from somewhere.

```bash
cat /etc/kubernetes/pki/ca.crt | base64 | tr -d '\n'
```

This is a huge pain. You'd have to do this manually every time you create a validating webhook. Why? I was thinking that there must be a better way...and then magically, I stumbled on it. Perhaps everyone else knows about this, but I didn't, I had to dig a bit.

## Cert Manager CA Injector

I stumbled on the [Cert Manager docs for CA Injection](https://cert-manager.io/docs/concepts/ca-injector/).

The example they give is this:

```
apiVersion: v1
kind: Namespace
metadata:
  name: example1

---

apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: webhook1
  annotations:
	cert-manager.io/inject-ca-from: example1/webhook1-certificate
webhooks:
- name: webhook1.example.com
  admissionReviewVersions:
  - v1
  clientConfig:
	service:
  	name: webhook1
  	namespace: example1
  	path: /validate
  	port: 443
  sideEffects: None

---

apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: webhook1-certificate
  namespace: example1
spec:
  secretName: webhook1-certificate
  dnsNames:
  - webhook1.example1
  issuerRef:
	name: selfsigned

---

apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned
  namespace: example1
spec:
  selfSigned: {}
```

Note the annotation:

```
  annotations:
	cert-manager.io/inject-ca-from: example1/webhook1-certificate
```

Once that annotation is there, and of course cert-manager is deployed and issuers configured, etc, the CA bundle can automatically be injected.

## Conclusion

Here it is in a real deployment (*I've removed most of the actual bundle for brevity*):

```
$ k get validatingwebhookconfigurations.admissionregistration.k8s.io blockfriday -oyaml | grep -i cabundle
	caBundle: LS0tLS1CRU <SNIP!>
```

This really made my life easier.