---
layout: post
title:  "Chain-Link - A Chain of Services in Kubernetes"
categories:
header_image: "/img/chain-link-header.jpg"
summary: "Demo tracing a set of applications"

---

# {{ page.title }}

## tl;dr

I built an app called [chain-link](https://github.com/ccollicutt/chain-link) that will create a "chain" of apps in Kubernetes of an arbitrary length. It's written in Python. 

While the point of this all is the app, what I learned most about what writing the CLI portion: the `chain-link-cli`. There's actually way more code there (for better or worse) to deploy and manage the app than there is in the app itself.

## What is it?

I wanted to do some work with a simple Python application that could allow creating a set of services that would form a chain that could be visualized in some kind of program (in this case it ended up being Zipkin, but it could be anything that can show traces).

I wanted it to do a few things:

* Support traces
* Create a "chain" of services of an arbitrary length 
* Randomly insert some sleep time into the chain
* Write a CLI that could be used to create and manage the chain
* Deploy a loadgenerator to activate the chain
* Deploy a tool that can visualize the chain through the traces it generates

## How does it work?

The app is written in Python and uses Flask. The app itself is pretty basic. It just looks at the generated list of services and forwards the request to the next service in the chain. It also adds some headers to the request to help with tracing.

This is what the pods look like in the cluster once they're deployed:

```
$ k get pods
NAME                                       READY   STATUS    RESTARTS        AGE
chain-link-deployment-0-855875c8d5-rz8wd   1/1     Running   2 (47d ago)     96d
chain-link-deployment-1-6cc5965f45-ch8cj   1/1     Running   6 (5d5h ago)    96d
chain-link-deployment-2-65dd5b4878-tnq2r   1/1     Running   6 (5d5h ago)    96d
chain-link-deployment-3-7bf888dddb-lwwdr   1/1     Running   6 (5d5h ago)    96d
chain-link-deployment-4-6c47c7dcb5-4sf4v   1/1     Running   2 (47d ago)     96d
chain-link-deployment-5-85655c8d4f-2z2r6   1/1     Running   6 (5d5h ago)    96d
loadgenerator                              1/1     Running   0               23m
zipkin-deployment-69c4598df6-js95l         1/1     Running   209 (16m ago)   99d
```

Each deployment is a separate "app" that the overall service chain is composed of. (Now that I write "service chain" this reminds me of my telecom days.)

## What does it look like in Zipkin?

![Zipkin](/img/chain-link-zipkin1.png)
![Zipkin](/img/chain-link-zipkin2.png)

## The CLI

Here's the CLI help:

```
$ ./chain-link-cli 
WARNING Using existing config file: /home/curtis/.config/chain-link/chain-link-cli.conf
usage: chain-link-cli [-h] [--instances NUM_INSTANCES]
                      [--namespace NAMESPACE]
                      [--chain-link-image IMAGE_NAME]
                      [--sleep-time SLEEP_TIME] [-d] [-v]
                      [--config-file CONFIG_FILE]
                      {deploy,validate,generate,dry-run} ...

Deploy the chain-link application to a Kubernetes cluster

positional arguments:
  {deploy,validate,generate,dry-run}
    deploy              Deploy chain-link to Kubernetes
    validate            Validate chain-link configuration
    generate            Generate chain-link kubernetes yaml
    dry-run             Generate chain-link kubernetes yaml

options:
  -h, --help            show this help message and exit
  --instances NUM_INSTANCES
                        Number of instances to deploy
  --namespace NAMESPACE
                        Namespace to deploy to
  --chain-link-image IMAGE_NAME
                        ChainLink image to deploy
  --sleep-time SLEEP_TIME
                        Time to sleep between loadgenerator requests
  -d, --info            Print lots of infoging statements
  -v, --verbose         Be verbose
  --config-file CONFIG_FILE
                        Specify the path to the config file
```

As you can see there are a few subcommands, like deploy, validate, etc.

Here's validate:

```
$ ./chain-link-cli validate
WARNING Using existing config file: /home/curtis/.config/chain-link/chain-link-cli.conf
INFO    Using the following configuration...
INFO    Number of instances: 6
INFO    Namespace: chain-link-2
INFO    ChainLink image: ghcr.io/ccollicutt/chain-link:latest
INFO    Loadgenerator sleep time: 60
INFO    Validating chain-link configuration...
INFO    Validating deployments...
INFO    Deployments ready
INFO    Validating pods...
INFO    Pods ready
INFO    All objects ready
```

All the config files are generated and placed in `~/.config/chain-link/manifests`:

```
$ ls ~/.config/chain-link/manifests/
chain-link-2-namespace.yaml
chain-link-deployment-0-deployment.yaml
chain-link-deployment-1-deployment.yaml
chain-link-deployment-2-deployment.yaml
chain-link-deployment-3-deployment.yaml
chain-link-deployment-4-deployment.yaml
chain-link-deployment-5-deployment.yaml
chain-link-service-0-service.yaml
chain-link-service-1-service.yaml
chain-link-service-2-service.yaml
chain-link-service-3-service.yaml
chain-link-service-4-service.yaml
chain-link-service-5-service.yaml
chain-link-services-configmap.yaml
loadgenerator-pod.yaml
zipkin-deployment-deployment.yaml
zipkin-service-service.yaml
```

There's a config file there too:

```
$ cat ~/.config/chain-link/chain-link-cli.conf 
[DEFAULT]
instances = 6
namespace = chain-link-2
chain_link_image = ghcr.io/ccollicutt/chain-link:latest
sleep_time = 60
```

There's about 800 lines of Python for the CLI:

```
$ wc -l *.py
   95 arg_parser.py
  569 chainlink.py
   80 cli_manager.py
   82 config.py
    0 __init__.py
   25 log_utils.py
   36 utils.py
  887 total
```

Versus 170 or so for the app itself.

```
$ wc -l app.py 
169 app.py
```

I need to do some research and see if there are some higher level abstractions in Python that can help reduce the number of lines in the CLI.

## Why a Python CLI and not Helm?

YAML isn't a programming language. Obviously you can do a lot "more" with Kubernetes using a real programming language. But of course, no one writes a Python CLI for every app they deploy to Kubernetes--that would make no sense.

I mean, it would make more sense to do this in Helm if I thought other people would actually use it. But I don't expect anyone ever would. This app was really just a learning experience for me, and part of the learning experience I wanted was to create a CLI that could manage the app for me in Kubernetes.

Ultimately, I might prefer to use Python to manage Kubernetes environments. Then again, Helm's ability to manage upgrades would be required in production. This is something I need to look into--how tools like Helm are looking at the state of a k8s app. Plus whatever other tooling exists--for example Pulumi (which I have never used).

## Conclusion

I learned a lot about Python and Kubernetes writing this application and the associated CLI. I'm sure there are tons of bugs, and there is, embarrassingly, little testing. That's something I learned I need to improve on: testing. 

For this app I just started writing, and this is what I ended up with. Is the end result perfect? No. Was the process of learning great? It sure was.