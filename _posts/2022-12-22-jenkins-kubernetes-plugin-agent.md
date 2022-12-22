---
layout: post
title:  "Jenkins and Kubernetes: Getting the plugin working"
categories:
header_image: "/img/jenkins-k8s.png"
summary: "Getting the Jenkins Kubernetes plugin working"

---

# {{ page.title }}

I wanted to try out using Kubernetes from Jenkins, and that is what this post is about.

I have a Jenkins instance running on a host, specifically it is NOT running in Kubernetes. But I want that instance of Jenkins to talk to a Kubernetes cluster and use it as a "cloud", where I'm using the term "cloud" in Jenkins parlance.

## Caveat

This is all just running in my homelab, where security isn't as big an issue as it would be in a real world situation. Keep that in mind! There's likely a lot that could be improved here from a security perspective.

## Install the Kubernetes Plugin

Given this Jenkins instance is just in my homelab, I just click buttons. If I want a plugin, I just install it from the GUI. It's fun to just click around for once. :)

![Jenkins Kubernetes plugin install](/img/jenkins-k8s-plugin1.jpg)


## Set up Kubernetes for use by Jenkins

>NOTE: Here I assume you have a Kubernetes cluster available.

First, create a namespace for Jenkins to use.

```
$ kubectl create ns jenkins-agent
```

Then create a service account in that namespace with the proper role and rolebinding.

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin
  namespace: jenkins-agent
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: jenkins
  namespace: jenkins-agent
  labels:
  "app.kubernetes.io/name": 'jenkins'
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get","list","watch"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins-role-binding
  namespace: jenkins-agent
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins
subjects:
- kind: ServiceAccount
  name: jenkins-admin
  namespace: jenkins-agent
```

Now create a token.

>NOTE, from the Kubernetes docs: If you want to obtain an API token for a ServiceAccount, you create a new Secret with a special annotation, kubernetes.io/service-account.name.

Apply this YAML.

```
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: jenkins-admin-token
  annotations:
    kubernetes.io/service-account.name: "jenkins-admin"
```

Get the token from the secret that was created and decode it from base64. It will be used to configure the Kubernetes cloud in Jenkins as a "secret text" credential type.

## Add a Kubernetes "cloud"

Go to "Dashboard -> Manage Jenkins -> Configure Clouds" and add a new Kubernetes cloud.

![Jenkins Kubernetes plugin install](/img/jenkins-k8s-plugin2.jpg)

Now configure that cloud instance.

Set the URL of the Kubernetes API endpoint.

Create the credential from here as well. Use the token we set up in Kubernetes and create a "secret text" credential.

![Jenkins Kubernetes plugin install](/img/jenkins-k8s-plugin3.jpg)

## Create a Pipeline

Create a new pipeline of "freestyle" type.

![Jenkins Kubernetes plugin install](/img/jenkins-k8s-plugin4.jpg)

Restrict where it can be run to the name you gave the Kubernetes cloud instance in Jenkins. In this case I called my "c2-kubernetes."

Here's the cloud configuration where I've configured the name "c2-kubernetes."

![Jenkins Kubernetes plugin install](/img/jenkins-k8s-plugin5.jpg)

Here's the job configuration. Note that it says "1 cloud" which refers to the Kubernetes cloud we just added.

![Jenkins Kubernetes plugin install](/img/jenkins-k8s-plugin6.jpg)

I created a simple job that just echos some output.

![Jenkins Kubernetes plugin install](/img/jenkins-k8s-plugin7.jpg)

If you run that job now you'll see a container get built in the Kubernetes cluster. It won't take long to run.

```
$ k get pods
NAME                  READY   STATUS        RESTARTS   AGE
c2-kubernetes-58rlp   1/1     Terminating   0          11s
```

Here's the console output of that Jenkins job.

```
Started by user admin
Running as SYSTEM
Agent c2-kubernetes-58rlp is provisioned from template c2-kubernetes
---
apiVersion: "v1"
kind: "Pod"
metadata:
  labels:
    jenkins: "slave"
    jenkins/label-digest: "396f736cb86bcc043738aedb977de7d31c574611"
    jenkins/label: "c2-kubernetes"
  name: "c2-kubernetes-58rlp"
  namespace: "jenkins-agent"
spec:
  containers:
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_AGENT_NAME"
      value: "c2-kubernetes-58rlp"
    - name: "JENKINS_NAME"
      value: "c2-kubernetes-58rlp"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://jenkins.example.com:8080/"
    image: "jenkins/inbound-agent:4.11-1-jdk11"
    name: "jnlp"
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  hostNetwork: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - emptyDir:
      medium: ""
    name: "workspace-volume"

Building remotely on c2-kubernetes-58rlp (c2-kubernetes) in workspace /home/jenkins/agent/workspace/test-kubernetes-cloud
[test-kubernetes-cloud] $ /bin/sh -xe /tmp/jenkins17424164001143670183.sh
+ echo hi c2-kubernetes
hi c2-kubernetes
Finished: SUCCESS
```

## Conclusion

This took a bit of testing to get right, but not that much work. I kinda like Jenkins in my homelab because I can just poke around at it and not worry too much about how replicable it all is. Jenkins is pretty good from that perspective, just install plugins, configure things manually, update plugins. Sometimes it's nice just to do ClickOps.

I've got a fair bit more to understand about this plugin though. There's a lot more work to be done around Pod Templates...but that's for another day. At least at this point Jenkins can create jobs in the Kubernetes cluster.

## ISSUE - tcpSlaveAgentListener

I had one issue with the container not completing properly because it couldn't connect to Jenkins. Note the "tcpSlaveAgentListener" issue.

```
$ k logs c2-kubernetes-xnb6n
Dec 21, 2022 11:28:49 PM hudson.remoting.jnlp.Main createEngine
INFO: Setting up agent: c2-kubernetes-xnb6n
Dec 21, 2022 11:28:49 PM hudson.remoting.jnlp.Main$CuiListener <init>
INFO: Jenkins agent is running in headless mode.
Dec 21, 2022 11:28:50 PM hudson.remoting.Engine startEngine
INFO: Using Remoting version: 4.11
Dec 21, 2022 11:28:50 PM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /home/jenkins/agent/remoting as a remoting work directory
Dec 21, 2022 11:28:50 PM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
INFO: Both error and output logs will be printed to /home/jenkins/agent/remoting
Dec 21, 2022 11:28:50 PM hudson.remoting.jnlp.Main$CuiListener status
INFO: Locating server among [http://jenkins.example.com:8080/]
Dec 21, 2022 11:28:50 PM hudson.remoting.jnlp.Main$CuiListener error
SEVERE: http://jenkins.example.com:8080/tcpSlaveAgentListener/ is invalid: 404 Not Found
java.io.IOException: http://jenkins.example.com:8080/tcpSlaveAgentListener/ is invalid: 404 Not Found
    at org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver.resolve(JnlpAgentEndpointResolver.java:219)
    at hudson.remoting.Engine.innerRun(Engine.java:724)
    at hudson.remoting.Engine.run(Engine.java:540)
```

I had to go into the Jenkin's configuration and give the agent a port. As soon as I set that the containers were able to connect.

![Jenkins Kubernetes plugin install](/img/jenkins-k8s-plugin8.jpg)
