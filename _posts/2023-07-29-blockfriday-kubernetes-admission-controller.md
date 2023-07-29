---
layout: post
title:  "blockfriday - Blocking Kubernetes Deployments with an Admission Controller"
categories:
header_image: "/img/blockfriday.png"
summary: "It's Friday somewhere..."

---

*Creating new deployments on a Friday is NOT allowed.*

# {{ page.title }}

Would you create a Kubernetes Admission Controller to block deployments on a Friday and use it in production? No. But you could, creat one, say, as an example admission controller.

So that is precisely what I have done, created a very, very (very) simple admission controller, written in Go, that blocks NEW Kubernetes deployments on a Friday. I call it [blockfriday](https://github.com/ccollicutt/blockfriday).

## What's an Admission Controller?

An admission controller is a piece of software that can intercept requests to the Kubernetes API server and either allow or deny them. You can use them to enforce policies, like "no deployments on a Friday" or "all deployments must have a label of `app: foo`". Using admission controllers you can basically apply "policy as code" to your Kubernetes cluster. (Note that there are a lot of admission controllers out there. This is just one example.)

## How does it work?

The admission controller is a simple Go program, this one is about 200 lines long, that runs in a pod in the cluster. Once a ValidatingWebhookConfiguration is created which points to this service, the kube-api will send requests to the admission controller for validation. The admission controller will then either allow or deny the request.

The code:

```
if isTodayFriday() {
	klog.Infof("Denying the request to create a new Deployment on Friday. Deployment: %s, Namespace: %s", deploymentName, namespace)
	return makeAdmissionResponse(admissionReview.Request.UID, false, "Creating new Deployments on Fridays is not allowed.")
}
```

## Certificates

Honestly the certificate portion of this admission controller was harder than writing the actual code. 

In the case of blockfriday:

* I'm using cert-manager to, uh, manage certificates
* The cluster as deployed with kubeadm, so there is a CA in /etc/kubernetes/pki/ca.crt
* I use that CA as part of a Cluster Issuer
* When deploying the admission controller, I use the Cluster Issuer to create a certificate for the admission controller
* The admission controller mounts that certificate (which has a cert and a key) in /cert and uses it
* The ValidatingWebhookConfiguration has a CA bundle that cert-manager injects (magically) for me based on the certificate that cert-manager created (nice), and then the kube-api can talk to the admission controller (though it would already be able to because I'm using the CA that kubeadm deployed, but you get my drift)

## Conclusion

Once the admission controller has been setup (certs, deployment, validatingwebhookconfiguration) it will block new deployments on a Friday.

```
$ date +%A
Friday
$ k create -f test/deployment.yaml 
Error from server: error when creating "test/deployment.yaml": admission webhook "blockfriday.serverascode.com" denied the request: Creating new Deployments on Fridays is not allowed.
```

Now we're blockin' Fridays!