---
layout: post
title: Kubernetes the Hard Way in AWS with Ansible
categories:
header_image: /img/angus_mckie.jpg
header_permalink: http://70sscifiart.tumblr.com/post/53902582514/a-beautiful-angus-mckie 
---

# {{ page.title }}

Even though I mostly work with OpenStack, I also quite like Amazon Web Services (AWS). Further I am doing a lot of work with containers, and have been doing so for a while--I was messing around with [Mesos](https://github.com/ccollicutt/ansible-mesos-playbook) two years ago.

Kubernetes has a lot of attention right now. Docker has slightly cooled off, but is moving rapidly forwards [technology-wise](https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/). Kubernetes has recently reached some kind of front-runner status in terms of container management systems, though I should note that Docker is being used to manage containers in all the examples discussed in this blog post. Kubernetes does not provide containers directly, and instead uses an underlying system such as Docker, or more recently, [rkt](https://coreos.com/rkt/) for that layer. Kubernetes focuses on managing the deployment of containerized applications.

## Kubernetes the Hard Way

A couple months ago [Kelsey Hightower](https://twitter.com/kelseyhightower) released a set of documents entitled [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) (KtHW) which details how to manually deploy k8s. k8s comes with a *kube-up.sh* script that will deploy it to several IaaS providers, but it hides much of the deployment complexity, which is considerable. I don't see how you could run k8s in production with just the *kube-up.sh* script.

I took the example that Mr. Hightower had provided and altered it to use AWS (instead of Google Compute Engine) and also use Ansible instead of running individual commands. But I still kept the "one step at a time" mentality, which combines perfectly with Ansible when using multiple playbooks for each infrastructure layer.

If you'd like to try out *Kubernetes the Hard Way in AWS with Ansible* (KtHWAA) then you can take a look at [this Github repository](https://github.com/ccollicutt/kubernetes-the-hard-way-with-aws-and-ansible). If you run into any errors, please don't hesitate to submit an issue via Github and I will work on getting it fixed right away.

## k8s in AWS

k8s integrates well with AWS. For example, using the networking methodology that is shown in KtHW, route tables need to be setup in the VPC to point the node networks to particular nodes. k8s can do this automatically. 

Further, when you create a service in k8s, it can automatically configure an AWS EC2 load balancer to expose the service externally. This includes creating appropriate security groups.

There are a few other integrations that I have yet to configure in my KtHWAA repository, but the two important ones, 1) pod routes and 2) ELB are completed and work greate.

The ability of k8s to configure the IaaS around it is quite amazing. I'm sure most people understand that AWS IAM roles and policies can be created, but to me it means something special because the infrastructure I create in AWS, such as k8s, can also configure AWS features. It seems obvious, but that is extremely powerful. This is something that is lacking in OpenStack--the ability to create roles and policies and provide components, such as a virtual machine, the ability to alter the tenant infrastructure around it.

I'm also impressed that the k8s project has put so much work in AWS integration. Given k8s is a Google led project, I'm happy to see them supporting other IaaS providers. So kudos to Google and the other members of the k8s project team.

## Using AWS with Ansible

I ran into a few funny issues with Ansible modules and AWS, but overall I'm happy with how things turned out. I could do everything I needed to with Ansible, though certainly there are some improvements to be made. Even though Ansible has been around for a while the modules are constantly improving and as AWS continues to grow at an unprecedented pace I'm sure the Ansible will get better and better in terms of managing AWS, as will my ability to properly use both.

## Forward Looking Statement

I plan on doing a lot more work with k8s in AWS. There are at least of couple of areas, 1) volumes and 2) auto scaling, that I would like to investigate. k8s is powerful but complicated, and the real goal isn't to be someone running k8s infrastructure, but rather to use k8s to run *actual* applications. 

AWS is extremely powerful and granular. Through creating KtHWAA I learned a lot about AWS Virtual Private Cloud (ie. networking) as well as the load balancer service. I've always been a big fan of AWS spot instances, so I use those as well (though I wouldn't use them as much in a production instance). As mentioned, IAM roles and policies are a huge feature and differentiator, though I haven't used other IaaS systems as extensively.

Regarding networking in k8s, I don't completely understand the [Container Networking Interface](https://github.com/containernetworking/cni) (CNI) setup, so there is some reading and experimentation to do in that area as well.

## Thanks!

Thanks to [Kelsey Hightower](https://twitter.com/kelseyhightower) for releasing KtHW, as well as the team that put together [KtHW with AWs](https://github.com/ivx/kubernetes-the-hard-way-aws), which replaces GCE commands with similar AWS ones.
