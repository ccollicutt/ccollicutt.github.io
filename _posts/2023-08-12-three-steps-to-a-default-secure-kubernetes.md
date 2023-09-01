---
layout: post
title:  "Three Steps to a Default Secure Kubernetes"
categories:
header_image: "/img/three-steps-default-security.jpg"
summary: "Exploring secure by default"

---

# {{ page.title }}

Kubernetes is a framework. We don't usually describe it as a framework, but it is. IMHO, it's a library we can use to deploy applications and imprint our organization's policies and requirements on top of. That's what makes it valuable, not the fact that it can create a container.

Because it's a basic framework, a set of lego blocks, it's not designed to be secure "out of the box." We've got to make it as secure as we need it to be.

## NOTE: Not a Panacea

This post is an exploration of some things we could do to make Kubernetes more secure by default. Like what are a couple minimal steps we could take that have a large return on investment. It's not meant to meet every organization's requirements or be the end-all-be-all of security. It's meant as an exploration of a secure starting point that could potentially work for everyone and every Kuberenetes.

In fact I should say here that I've already had people give me diffrent opinions on these settings. For example for network policies here I'm thinking more of lateral movement, but many organizations would prefer to stop outbound access. It really depends on your organization's requirements. 

## 1 - Pod Security Standards

First, if you're not familiar with Pod Security Standards, it's not a bad idea to [go read up on them](https://serverascode.com/2023/08/02/making-pod-security-standards-default.html), but suffice it to say let's make sure every namespace has the following label.

```
kubectl label namespace <NAMESPACE> pod-security.kubernetes.io/enforce=restricted
```

Which means:

* Limit types of volumes
* Don’t run as root user
* No privilege escalation
* Seccomp profile set to “RuntimeDefault” or “Localhost”
* Drop all capabilities except perhaps add NET_BIND_SERVICE

Here's an example of running the nginx unprivileged container.

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

That's a pretty good start!

## 2 - Network Policies

Historically, in Kubernetes, the connectivity was based on a giant, flat layer 3 network, which means every pod had an IP address and could talk to every other pod in the cluster. Obviously this doesn't really work in enterprise environments, so Kubernetes added the ability to create [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

Here let's create a default policy to ensure that pods in a namespace can talk to one another, but cannot talk to pods OUTSIDE of their namespace. This is super basic, but I like it as a starting point. Note that services would still be accessible, just not pods directly.

Here's an example:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: namespace-only
spec:
  podSelector: {} # Selects all pods in the namespace
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector: {} # Allow pods in the same namespace to talk to one another
  egress:
  - to:
    - ipBlock:
   	 cidr: 0.0.0.0/0 # Allow egress to any destination
```

Obviously this can be taken a lot further, especially with a CNI that provides extra capabilities.

## 3: Runtime Threat Detection

The last step is to make sure that we're monitoring our Kubernetes clusters for runtime threats. While we have a good baseline of security, something will always get through, no matter what we do. Any adversary worth their salt will find a way in, or mistakes will be made, etc. So we need to be able to detect them when they do. The only way to do that is to monitor the runtime environment.

We can do that in a couple of ways, one is to use Sysdig Secure, which is a commercial [CNAPP (Cloud Native Application Protection Platform)](https://sysdig.com/learn-cloud-native/cloud-security/cloud-native-application-protection-platform-cnapp-fundamentals/) that has a decade of history in runtime protection. The other is to use Falco, which is an open source project that is part of the CNCF. Sysdig as a company supports the Falco project.

### 3a - Sysdig Secure

Sysdig Secure is a security platform which has a decade of history in runtime protection.

It's easy to sign up for a [free trial](https://sysdig.com/start-free/) at Sysdig.

Then, install the agent into Kubernetes clusters with Helm:

```
helm repo add sysdig https://charts.sysdig.com
helm repo update
helm install sysdig-agent --namespace sysdig-agent --create-namespace \
--set global.sysdig.accessKey=<ACCESS_KEY> \
--set global.sysdig.region=<SAAS_REGION> \
--set nodeAnalyzer.secure.vulnerabilityManagement.newEngineOnly=true \
--set global.kspm.deploy=true \
--set nodeAnalyzer.nodeAnalyzer.benchmarkRunner.deploy=false \
--set nodeAnalyzer.nodeAnalyzer.hostScanner.deploy=true
--set global.clusterConfig.name=<CLUSTER_NAME> \
sysdig/sysdig-deploy
```

Done!

### 3b - Falco

[Falco](https://falco.org/) is an open source project that is part of the CNCF. It's a runtime threat detection engine that can be used to detect threats in Kubernetes clusters. Sysdig as a company supports the Falco project.

>Falco is a cloud native runtime security tool for Linux operating systems. It is designed to detect and alert on abnormal behavior and potential security threats in real-time.
>At its core, Falco is a kernel monitoring and detection agent that observes events, such as syscalls, based on custom rules. Falco can enhance these events by integrating metadata from the container runtime and Kubernetes. The collected events can be analyzed off-host in SIEM or data lake systems.

Installing Falco into Kubernetes is easy, just use Helm:

```
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
helm install falco falcosecurity/falco
```

Done!

## Automation

I would prefer that this all be done automatically. Because Kubernetes is a framework there are ways to make these kinds of security settings default, including the concepts of building operators and admission controllers. That would be my next step, to set up some tooling that would automatically apply these settings to every cluster, to every namespace, and to every pod.

So, look forward to a future blog post on that!

## Conclusion

I want to be clear that the point here is to create something that is simple and at the same time really improves the default security of Kubernetes--like what's the best bang for the buck we can get in terms of security.

In this blog post we've seen how to create a higher level of default security for Kubernetes, and we looked at how to use Sysdig Secure and Falco to monitor the runtime environment for threats.

Ultimately, this post is an exploration of how to configure a Kubernetes cluster so that it is much more secure "by default." There's no need to have Kubernetes be so wide open.

PS. I've included an optional section discussing Buildpacks and how they can be used to create more secure container images.

## OPTIONAL: Buildpacks and Paketo

### Background

Often people are surprised to find out that there is more than one way to build a container image. I mean, what's a container image: it's just a fancy tar file. There are many ways one can make a fancy tar file.

One way is [buildpacks](https://buildpacks.io/). I've written about them [before](https://serverascode.com/2019/12/16/buidpack-pack.html). [Paketo](https://paketo.io/) is a set of buildpacks that are designed to be used with Kubernetes.

For the purposes of this blog post, the point of Vuildpacks is that they are a way to build a container image that is more secure by default. For example, buildpacks don't run as root. If we just get rid of that one thing, we've made our container images more secure.

The value of Buildpacks:

* Security - Buildpacks run as non-root by default.
* Advanced Caching - Robust caching is used to improve performance.
* Auto-detection -Images can be built directly from application source without additional instructions.
* Bill-of-Materials - Insights into the contents of the app image through standard build-time SBOMs in CycloneDX, SPDX and Syft JSON formats.
* Modular / Pluggable- Multiple buildpacks can be used to create an app image.
* Multi-language - Supports more than one programming language family.
* Multi-process - Image can have multiple entrypoints for each operational mode.
* Minimal app image - Image contains only what is necessary.
* Rebasing - Instant updates of base images without re-building.
* Reproducibility - Reproduces the same app image digest by re-running the build.
* Reusability - Leverage production-ready buildpacks maintained by the community.

### Paketo

[Paketo](https://paketo.io/) is a set of buildpacks that are designed to be used with Kubernetes.

First off this is a Python app and I'm using gunicorn, so we have a Procfile. This is really the only difference I have between a Dockerfile based image and a buildpack based image. Instead of a Dockerfile I have a Procfile, and the Procfile only describes the command to run the app, nothing else.

>Procfiles define processes from your application’s code and contains one process per line.

Here's the example Procfile:  

```
$ cat Procfile
web: gunicorn app:app
```

Now we can build the image with a straight forward command.

>NOTE: This assumes, of course, the pack CLI is installed.

```
pack build somepythonflaskapp \
  --buildpack paketo-buildpacks/python \
  --builder paketobuildpacks/builder:base
```

And deploy it into Kubernetes, port-forward to the service, and finally curl the app. 

>NOTE: This app is setup to report the user it's running as.

```
$ curl localhost:8000
Application is running as user: cnb
```

Above we can see the app, which is configured to return the user it's running as, is reporting that it is running as user "cnb" aka not root, aka Cloud Native Buildpacks. Done by default. Nice.