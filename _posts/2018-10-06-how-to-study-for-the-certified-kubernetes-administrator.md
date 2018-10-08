---
layout: post
title: How to Study for the Certified Kubernetes Administrator (CKA) Exam
categories:

---

# {{ page.title }}

Last week I successfully passed the CKA exam, and I wanted to give some pointers on how to best study for it.

Obviously I can't discuss any of the exam content. You'll need to know a lot more than what I've listed here in terms of deploying and managing Kubernetes. I've only covered some basics--though I think the information on kubectl generators will really help. Kubernetes the hard way is also useful.

## The CKA Exam

Here are a few basic skills you will need to pass the exam. Well, even to take the exam.

1. Exam time management
1. Linux command line
2. Command line text editor
3. Generating valid YAML with kubectl
4. Using and searching the kubernetes.io/docs pages
6. Editing YAML live

Again, there is quite a bit more you will need to know, especially about deploying and managing Kubernetes, but these are the basics. Checkout the [curriculum](https://github.com/cncf/curriculum) for more ideas on what to study.

**Exam Time Management**

It's a three hour exam and I expect that most people will use the entire time. Questions have differing levels of difficulty and value. You are allowed to open a note pad in the exam browser application, so it's a good idea to take notes on what questions have what value and which one's you have completed. Definitely figure out a strategy for picking which questions to answer first, and last.

**Linux Command Line** 

You need to be comfortable with the Linux command line. The basics would be Ok. If you can change directories, open files with vim, and run kubectl you will probably be fine.

**Command Line Text Editor** 

This is a big one. If you are not capable of opening and closing vim (or nano), typing into it, and search and replace, the exam will likely not be possible to pass. I would suggest being very comfortable with vim. Search and replace is a good skill to have: `:s/new/old/g`. `:x` is the same as `:wq`. :)

**Generating YAML with kubectl**

It's easy to create a deployment from the command line and then export the YAML. Actually there are several [generators](https://kubernetes.io/docs/reference/kubectl/conventions/#generators). Using `kubectl run` with different options can create: pods, deployments, replication controllers, jobs, and cron jobs.

First create a deployment with the run command. Without any options a deployment is created.

```
master $ kubectl run --image nginx nginx
deployment.apps/nginx created
```

Now we can export the YAML that kubectl generated behind the scenes.

Note the `--export` option.

```
master $ kubectl get deploy nginx -o yaml --export
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: null
  generation: 1
  labels:
    run: nginx
  name: nginx
  selfLink: /apis/extensions/v1beta1/namespaces/default/deployments/nginx
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      run: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status: {}
```

The best thing to do is to export that to a file.

```
kubectl get deploy nginx -o yaml --export > nginx-deploy.yml
```

Then delete that deploy if you don't need it.

```
kubectl delete deploy nginx
```

Now we have an example deployment that can be used to create other deployments...a base file effectively.

To generate a YAML for a **pod**, use `--restart=Never`

```
master $ kubectl run nginx --image=nginx --port=80 --restart=Never
pod/nginx created
master $ kubectl get pod nginx -o yaml --export
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
SNIP!
```

Remember that other options can create jobs, cron jobs, recplication controllers, etc.

To practice generating YAML you don't need much more than a [Katacoda playground](https://www.katacoda.com/courses/kubernetes/playground).

**kubernetes.io/docs**

This is a very useful page. Get used to being able to search it and find the documents for key resources.

**Editing YAML Live**

In some cases it might be easiest to edit the YAML live.

```
kubectl edit deploy nginx
```

This will open up the YAML for the nginx deploy in $EDITOR, ie. vim. When you close it, the new YAML will be applied (or if the YAML has errors then it will error and reopen in vim, like visudo if you've ever used it).

## Conclusion

The CKA exam is a tough one, but mostly due to time limitations and the fact that you need to generate a lot of YAML quickly, without cutting and pasting. I think the CKA is a good certification to get at this point in time, and I'm glad that I only had to write it once.