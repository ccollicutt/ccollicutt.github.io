---
layout: post
title:  "Understanding Kubernetes Pod Security: runAsNonRoot and runAsUser"
categories:
header_image: "/img/runasuser.jpg"
summary: "Who can run what and as whom within a pod?"

---

# {{ page.title }}

Security is a prime concern when deploying applications in a Kubernetes cluster. One of the security aspects in Kubernetes is controlling who can run what and as whom within a Pod. Kubernetes provides two important fields in the Security Context to achieve this: runAsNonRoot and runAsUser. While they might seem similar at first glance, they serve different purposes. This blog post aims to demystify these settings and help you make the right choice for your applications.

## nginx images

I have to wonder what percentage of containers are just nginx instances that are there to test something out. Nginx is an easy image to deploy because you can just do:

```
kubectl run nginx --image=nginx
```

And you have a running nginx instance. 

However, that default nginx image will run as root (if your cluster allows that).

```
$ k exec -it nginx -- cat /proc/1/status | grep "Name\|Uid"
Name:	nginx
Uid:	0	0	0	0
```

There is an nginx unprivileged image that will run as a non-root user; it runs as user 101. I would definitely recommend using this image if you are just testing something out. It's a few more letters to type, but it's worth it.

```
kubectl run nginx-unprivileged --image=nginxinc/nginx-unprivileged
```

Inspecting the images...

```
$ docker image inspect nginx | jq '.[0].Config.User'
""
$ docker image inspect nginxinc/nginx-unprivileged | jq '.[0].Config.User'
"101"
```

Here's the nginx user in the unprivileged image:

```
$ k exec -it runasnonroot-and-runasuser -- grep nginx /etc/passwd
nginx:x:101:101:nginx user:/nonexistent:/bin/false
```

<br>

## Differences Between runAsNonRoot and runAsUser

|                  | runAsNonRoot  | runAsUser     |
|------------------|--------------|--------------|
| **Purpose**      | Ensure container doesn't run as root | Specify the exact UID for container |
| **Settings**     | true/false   | Numeric UID  |
| **Flexibility**  | Less flexible | More flexible |
| **Specificity**  | General: just not root | Very specific: exact UID |
| **Root Allowed** | No           | Yes, if specified |

<br>

## runAsNonRoot

The runAsNonRoot field specifies that the container must not run as the root user. Setting this to true enforces that the container should be executed as a non-root user. If the container image specifies a user as root or numerically as 0, the container won't start. It's a way to ensure that your application doesn't unintentionally run with more permissions than it needs.

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  securityContext:
    runAsNonRoot: true
  containers:
  - name: my-container
    image: nginxinc/nginx-unprivileged
EOF
```

Check the id of the user running the container:

```
$ kubectl exec -it non-root-pod -- id
uid=101(nginx) gid=101(nginx) groups=101(nginx)
```

But if we run the plain nginx image, it will fail:

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod-plain
spec:
  securityContext:
    runAsNonRoot: true
  containers:
  - name: my-container
    image: nginx
EOF
```

Note that the container is not running:

```
$ k get pods
NAME                 READY   STATUS                       RESTARTS   AGE
non-root-pod         1/1     Running                      0          112s
non-root-pod-plain   0/1     CreateContainerConfigError   0          5s
```

And the reason is:

```
$ k describe pod non-root-pod-plain  | grep Error
      Reason:       CreateContainerConfigError
  Warning  Failed     1s (x5 over 42s)  kubelet            Error: container has runAsNonRoot and image will run as root (pod: "non-root-pod-plain_runasnonroot(c5764bbb-c1cf-47b1-9606-3a3a49ebf666)", container: my-container)
```

<br>

## runAsUser

On the other hand, runAsUser specifies which UID (User ID) the container process should run as. Unlike runAsNonRoot, this allows you to specify the exact UID of the user, including root if you explicitly set it to 0.

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: specific-user-pod-nginx-priviliged
spec:
  securityContext:
    runAsUser: 1001
  containers:
  - name: my-container
    image: nginxinc/nginx-unprivileged
EOF
```

Check the id of the user running the container:

```
$ kubectl exec -it specific-user-pod-nginx-priviliged -- id
uid=1001 gid=0(root) groups=0(root)
```

But if we run the plain nginx image, it will fail:

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: specific-user-pod-nginx-plain
spec:
  securityContext:
    runAsUser: 1001
  containers:
  - name: my-container
    image: nginx
EOF
```

Note that the container is not running:

```
$ k get pod specific-user-pod-nginx-plain 
NAME                            READY   STATUS             RESTARTS      AGE
specific-user-pod-nginx-plain   0/1     CrashLoopBackOff   1 (10s ago)   16s
```

<br>

## Both runAsNonRoot and runAsUser

You can also use both runAsNonRoot and runAsUser together. In this case, runAsUser specifies the UID to use, and runAsNonRoot ensures that UID is not root.

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: runasnonroot-and-runasuser
spec:
  securityContext:
    runAsUser: 1001
    runAsNonRoot: true
  containers:
    - name: my-container
      image: nginxinc/nginx-unprivileged
EOF
```

Check the id of the user running the container:

```
$ k exec -it runasnonroot-and-runasuser -- id
uid=1001 gid=0(root) groups=0(root)
$ k get pods runasnonroot-and-runasuser 
NAME                         READY   STATUS    RESTARTS   AGE
runasnonroot-and-runasuser   1/1     Running   0          22s
```

And nginx is indeed running as user 1001:

```
$ k exec -it runasnonroot-and-runasuser -- cat /proc/1/status | grep "Name\|Uid"
Name:	nginx
Uid:	1001	1001	1001	1001
```

>NOTE: PID 1 is the first process that runs in any operating system or containerized environment. When a container starts, it launches a single process with a PID (Process ID) of 1 within the isolated namespace of that container. 

<br>

## nginx plain as non-root with runAsUser

Will it blend?

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx-plain-runasnonroot-and-runasuser
spec:
  securityContext:
    runAsUser: 1001
    runAsNonRoot: true
  containers:
    - name: my-container
      image: nginx
EOF
```

No.

```
$ k get pod nginx-plain-runasnonroot-and-runasuser 
NAME                                     READY   STATUS             RESTARTS     AGE
nginx-plain-runasnonroot-and-runasuser   0/1     CrashLoopBackOff   1 (7s ago)   12s
```

<br>

## Use Cases

* runAsNonRoot: Use this setting when you want a general assurance that none of the containers in the Pod are running as root, without caring which user they run as.

* runAsUser: Use this when you need more control over the exact user that runs the container process, such as for compliance with internal security policies that require specific UIDs for different types of applications.

## Conclusion

While runAsNonRoot and runAsUser both provide ways to control the user running a container, they serve different needs. runAsNonRoot is a more generalized setting to prevent root access, while runAsUser gives you fine-grained control over the user ID. Knowing when to use each can improve the security posture of your Kubernetes applications.

That said, both runAsUser and runAsNonRoot can co-exist. When they do, runAsUser specifies which UID to use, and runAsNonRoot ensures that UID is not root.

In normal, production systems one would never run a container as root so the image would have a user setup and the Kubernetes manifest would have runAsNonRoot set to true.