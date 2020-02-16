---
layout: post
title: 20 Tools to Manage Kubernetes Manifests
categories:
header_image: "/img/tools.jpg"
summary: "20 plus ways to generate kubernetes manifests"

---

# {{ page.title }}
 
> Of all the problems we have confronted, the ones over which the most brainpower, ink, and code have been spilled are related to managing _configurations._ - [Brendan Burns, Brian Grant, David Oppenheimer, Eric Brewer, and John Wilkes, Google Inc](https://queue.acm.org/detail.cfm?id=2898444).

I spent some time tracking down tools that seem like they manage K8S manifests. I found...many. This list of ~20 only touches the surface of a bubbling ecosystem.

Some of these tools are large and some are small. Some are simple templating systems, others can manage the entire life cycle of applications across many kinds of infrastructure. Some are libraries for programming languages, others complete languages themselves. A few are mature projects, some brand new, and others side projects of a single developer. The breadth and variance is quite amazing!

Please note that these are in no specific order, and neither does that order represent any kind of preference. Further, some of these tools, eg. Ksonnet, are no longer maintained. _This is not an exhaustive list!_

But let's get to it.

### _**20 Tools (and 2 Bonus Tools)**_

1.  [**Jsonnet**](https://jsonnet.org/) - A data templating language. Used by many tools in this list behind the scenes. Powerful, but sometimes with great power comes with great, uh, cognitive load?
    
2.  [**jk**](https://jkcfg.github.io/#/) - _"jk generates all your JSON, YAML and arbitrary text configuration files. With a little luck, you will not have to touch a YAML file again. Ever."_ Bold claim!
    
3.  [**Cue**](https://cuelang.org/) - Apparently created by the person (or persons) behind the Borg Configuration Language (BCL) used at Google. _"CUE is an open source language, with a rich set APIs and tooling, for defining, generating, and validating all kinds of data: configuration, APIs, database schemas, code, … you name it."_
    
4.  [**Isopod**](https://github.com/cruise-automation/isopod)  \- _"Isopod is an expressive DSL framework for Kubernetes configuration. Without intermediate YAML artifacts, Isopod renders Kubernetes objects as [](https://github.com/protocolbuffers/protobuf)_ [_Protocol Buffers_](https://github.com/protocolbuffers/protobuf)_, so they are strongly typed and consumed directly by the Kubernetes API."_ Interesting that they skip the YAML. Based on [](https://github.com/stripe/skycfg) [Skycfg](https://github.com/stripe/skycfg). A good intro [](https://medium.com/cruise/isopod-5ad7c565d350) [here](https://medium.com/cruise/isopod-5ad7c565d350).
    
5.  [**Configula**](https://github.com/brendandburns/configula) - "_Configula is inspired by the [](https://reactjs.org/docs/introducing-jsx.html)_ [_JSX language_](https://reactjs.org/docs/introducing-jsx.html) _in [](https://reactjs.org/)_ [_React_](https://reactjs.org/) _that combines Javascript and HTML tags. Configula defines a similar pattern for Python and YAML (for now)."_ Joe Beda went over it in [](https://www.youtube.com/watch?v=efUAuOxR-ro) [TKIG](https://www.youtube.com/watch?v=efUAuOxR-ro). It's a Go utility that calls out to Python!
    
6.  [**Ko**](https://github.com/google/ko) - Point your K8S container image to a Go repo. It will build a container image using the binary generated from the repo. Does not use Docker to actually build the image. Wraps kubectl in some places.
    
7.  [**Kustomize**](https://kustomize.io/) - _"kustomize targets kubernetes; it understands and can patch kubernetes style API objects. It's like make, in that what it does is declared in a file, and it's like sed, in that it emits edited text."_ I've seen Kustomize used a few times; seems popular. Also it is conveniently part of Kubectl.
    
8.  [**Helm**](https://helm.sh/) **-** Helm is likely the most commonly used tool on this list. Recently Helm 3 was released removing its main criticism regarding the use of the tiller server.
    
9.  [**Kubecfg**](https://github.com/bitnami/kubecfg) - _"Kubecfg relies heavily on [](http://jsonnet.org/)_ [_jsonnet_](http://jsonnet.org/) _to describe Kubernetes resources, and is really just a thin Kubernetes-specific wrapper around jsonnet evaluation."_ 'nuff said!
    
10.  [**Ksonnet**](https://ksonnet.io/) - Ksonnet is no longer worked on, but I think it's a good thing to have in this list as a reminder ensure that not every tool will stick around. Experimentation will continue in this space.
    
11.  [**k14s**](https://k14s.io/) - _"We believe that working with [](https://content.pivotal.io/blog/introducing-k14s-kubernetes-tools-simple-and-composable-tools-for-application-deployment)_ _[simple, single-purpose tools](https://content.pivotal.io/blog/introducing-k14s-kubernetes-tools-simple-and-composable-tools-for-application-deployment)_ _that easily interoperate with one another results in a better, workflow compared to the all-in-one approach chosen by Helm. We have found this approach to be easier to understand and debug."_ k14s is developed by people at Pivotal (where I work, now part of VMware). There's a [](https://www.youtube.com/watch?v=CSglwNTQiYg) [TGIK](https://www.youtube.com/watch?v=CSglwNTQiYg) episode on it. The Unix philosophy has done us all well over time, perhaps this is the right approach to managing K8S manifests.
    
12.  [**Pulumi**](https://www.pulumi.com/)  \- Allows you to use "real" programming languages to manipulate cloud environments, including K8S. Interestingly supports multiple programming languages. _"Define infrastructure in JavaScript, TypeScript, Python, Go, or any .NET language, including C#, F#, and VB."_
    
13.  [**Ballerina**](https://ballerina.io/)  \- _    "Ballerina is an open source programming language and platform for cloud-era application programmers to easily write software that just works."_ Write code in the ballerina language that can build K8S objects for you.
    
14.  [**Kapitan**](https://github.com/deepmind/kapitan) - _"...a tool to manage complex deployments using jsonnet, kadet (alpha) and jinja2. Use Kapitan to manage your Kubernetes manifests, your documentation, your Terraform configuration or even simplify your scripts."_ Another tool wrapping jsonnet.
    
15.  [**Ansible**](https://docs.ansible.com/ansible/latest/modules/k8s_module.html)  \- Ansible can manage K8S objects and is also fairly good a templating, allowing you to easily write custom inventories and pull in metadata about your systems. Simple and powerful.
    
16.  [**Terraform**](https://www.terraform.io/docs/providers/kubernetes/index.html)  \- Terraform can manage K8S objects. Read about why one would use Terraform and K8S [](https://www.hashicorp.com/blog/managing-kubernetes-applications-with-hashicorp-terraform/) [here](https://www.hashicorp.com/blog/managing-kubernetes-applications-with-hashicorp-terraform/). The fact that Terraform is declarative would likely be a good match for K8S.
    
17.  [**Habitat**](https://www.habitat.sh/)  \- _"Chef Habitat is open source software that creates platform-independent build artifacts and provides built-in deployment and management capabilities."_ I can't do Habitat any justice here, other than to say my impression is that it is a can manage applications on several different types of platforms, of which K8S is one.
    
18.  [**Skaffold**](https://github.com/GoogleContainerTools/skaffold)  \- _"Skaffold is a command line tool that facilitates continuous development for Kubernetes applications. You can iterate on your application source code locally then deploy to local or remote Kubernetes clusters."_ Aimed at the inner dev loop, but can deploy to K8S as well.
    
19.  [**Kuby**](https://github.com/smartive/kuby/) - _"_[_Kuby_](https://blog.smartive.ch/how-we-simplified-our-kubernetes-deployments-with-an-alternative-to-helm-aafedcfd4cce) _wraps kubectl and provides commands like kuby prepare to replace variables in YAML or kuby deploy to prepare and apply deployments to Kubernetes."_
    
20.  [**Dekorate**](https://github.com/dekorateio/dekorate) - _"The are tons of tools out there for scaffolding / generating kubernetes manifests. Sooner or later these manifests will require customization...Using external tools, is often too generic. Using build tool extensions and adding configuration via xml, groovy etc is a step forward, but still not optimal."_ I think that introduction is apropos given this list. Keep manifest generation as part of your development/language environment, makes sense to me especially if your org is focussed on one language, i.e. Java.
    
21.  **BONUS: [](https://github.com/pivotal/kpack)** [**Kpack**](https://github.com/pivotal/kpack) - A system for using Cloud Native [](https://buildpacks.io/) [buildpacks.](https://buildpacks.io/) Buildpacks are a more structured, opinionated way to build container images. Joe Beda ran a [](https://www.youtube.com/watch?v=4zkRX9PSJ5k&feature=youtu.be) [TKIG](https://www.youtube.com/watch?v=4zkRX9PSJ5k&feature=youtu.be) on it. The pack command line can be used to build local images, or kpack can be deployed into K8S and will automatically rebuild images when code is changed in repositories. Kpack is focussed in image builds, not so much on generating manifests. (NOTE: Built by Pivotal, now VMware.)
    
22.  **BONUS: Any programming language** \- I wanted to make sure that I put this on the list, as ultimately any language that can print to standard out could be used to generate K8S manifests. Of course, we would likely not want to do something that simplistic, but I don't think it's far fetched or unreasonable to use templates and a generic programming language to build manifests.
    

### **Conclusion**

As I mentioned at the start this list is random--the order it comes in was the order I discovered them on the Internet. (Certainly there are some cool tools I have missed, let me know in the comments.)

Ultimately, these tools provide some kind of templating, wrapper, abstraction, or other convenience around K8S, though, of course, they all do it in different ways and chose different approaches. The differences and value of these individual tools is typically under the surface layer and thus require some investigation to truly understand when, or when not, to use them.

With so many options it does seem like the market (made up of us technical people--the people who want and use K8S) hasn't decided how exactly to use Kubernetes. A "silver bullet" approach probably won't work. Will one of these tools win out? A handful? Will we get something new? Or does Platform as a Service still make the most sense? Time will tell. Until then, we all have to perform some due diligence and keep innovating.

----

PS. Once I found ~20 I stopped. A few minutes later I found [this spreadsheet](https://docs.google.com/spreadsheets/d/1FCgqz1Ci7_VCz_wdh8vBitZ3giBtac_H8SBw4uxnrsE/edit#gid=0) with 120 similar tools!