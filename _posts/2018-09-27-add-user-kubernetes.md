---
layout: post
title: Add a User to Kubernetes
categories:

---

# {{ page.title }}

Ok, so the title is a bit misleading. k8s doesn't manage users, and instead expects them to be managed externally.

>Normal users are assumed to be managed by an outside, independent service. An admin distributing private keys, a user store like Keystone or Google Accounts, even a file with a list of usernames and passwords. In this regard, Kubernetes does not have objects which represent normal user accounts. Normal users cannot be added to a cluster through an API call.

Basically if a user can authenticate, it's a user. That's how I look at it anyways. Note that service accounts are a different story.

So, in this example, I'm assuming you've deployed using kubeadm and aren't managing users externally. kubeadm has a nice command to generate a kubeconfig for a "new user." It has to read files /etc/kubernetes/pki.

```
sudo kubeadm alpha phase kubeconfig user --client-name="curtis" > ~/curtis.kubeconfig
```

Now that I've created a kubeconfig for the user "curtis" I can use it to access the k8s API.

Or, well I could if RBAC was setup for the user. But at least I'm authenticating, I'm just not going to be authorized (if RBAC is enabled).

You can also create the certs using openssl and [bitnami](https://docs.bitnami.com/kubernetes/how-to/configure-rbac-in-your-kubernetes-cluster/) has some good docs on doing that, so I won't copy them here.