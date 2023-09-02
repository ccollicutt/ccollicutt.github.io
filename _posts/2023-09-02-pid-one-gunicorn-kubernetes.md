---
layout: post
title:  "Why My Flask App Refused to Crash: Understanding PID 1 in Containers and Kubernetes"
categories:
header_image: "/img/gunicorn-pid-1.jpg"
summary: "When your app just flat out refuses to crash..."

---

# {{ page.title }}

You've just deployed your Python Flask app on Kubernetes. You're using Gunicorn as your WSGI server, and you're trying to test how the container would behave if the app crashed. But wait! You find out that the container never crashes. Why not? Oh, Gunicorn is being helpful--it keeps restarting the application. 

OK, maybe this isn't "you" it's "me". I was trying to build a demo app that showed crash loop backoff in Kubernetes, and I couldn't get the container to crash. 

Pid 1 is Gunicorn.

```
$ k exec -it crash-only-backend-0 -- cat /proc/1/status | grep "Name\|Uid"
Name:	gunicorn-run.sh
Uid:	10001	10001	10001	10001
```

Let's dive into why this happens and the importance of understanding PID 1 in containers.

## What the Heck is Gunicorn?

>Gunicorn 'Green Unicorn' is a Python WSGI HTTP Server for UNIX. It's a pre-fork worker model. The Gunicorn server is broadly compatible with various web frameworks, simply implemented, light on server resources, and fairly speedy. - [Gunicorn](https://gunicorn.org/)

## What is a WSGI Server?

A Web Server Gateway Interface (WSGI) server is a web server that implements the WSGI specification. The WSGI specification is a Python standard that describes how a web server communicates with web applications.

## Why Does Flask Need a WSGI Server?

Flask is a micro web framework written in Python. It's a WSGI application, which means it needs a WSGI server to run.

>"Production" means "not development", which applies whether you’re serving your application publicly to millions of users or privately / locally to a single user. Do not use the development server when deploying to production. It is intended for use only during local development. It is not designed to be particularly secure, stable, or efficient. - [Flask](https://flask.palletsprojects.com/en/2.3.x/deploying/)

## What is PID 1?

In Unix-based systems, the process ID (PID) is a unique identifier for each running process. The very first process that runs when a system starts is the init system with PID 1. The init process has special responsibilities, like adopting orphaned child processes and handling signals.

## Gunicorn and PID 1

When you run a container, the process you start becomes PID 1 within that container. In the case of my Flask app, Gunicorn becomes PID 1. 

>NOTE: Why not just run Flask directly? Because Gunicorn is a production-ready WSGI server that can handle multiple requests concurrently.

## Why Doesn't the Container Crash?

If your Flask app (running as a Gunicorn worker) crashes, Gunicorn will restart it. Since Gunicorn is PID 1, the container will remain alive as long as Gunicorn does. This is why even if your Flask app encounters an error, the container doesn't crash.

## The Kubernetes Perspective

In a Kubernetes cluster, the kubelet will restart a crashed container based on its `restartPolicy`. However, if Gunicorn (PID 1) doesn't crash, Kubernetes won't know that something is wrong with your Flask app. This could lead to misleading metrics and logs, affecting your debugging and monitoring efforts.

## Killing Pid 1

In the app I still wanted to use gunicorn which means to demonstrate an app crashing and Kubernetes restarting the container, I needed to kill gunicorn. I needed to kill PID 1.

Here's what I ended up with inside the app:

```
def random_crash():
    if random.randint(1, 100) > 94:
        logger.error("<<<< Crashing... >>>>")
        # gunicorn will restart the process, which is pretty cool, but for this
        # app we want to purposely crash the whole container, so we kill the
        # parent process which is gunicorn
        os.kill(os.getppid(), 9)
```

>NOTE: I'm purposely trying to crash the container to demonstrate crashLoopBackoff. As well, I'm not using `sys.exit()` because that would just exit the flask process, not the container. (Which, by the way, is what I originally did and why I couldn't get the container to crash.)

## Conclusion

Understanding PID 1 in containers is crucial for debugging, process management, and robustness, especially when deploying applications on Kubernetes. 

To many, this is elementary...even downright obvious. But I've been doing this for a while and I still learned something when building this little demo app. As well, while researching this I found a fair bit of confusion around using gunicorn in containers.

Let me know if you have any questions or comments! 