---
layout: post
title:  "Tanzu Application Platform, knative, and a NodeJS App"
categories:
header_image: "/img/knative-nodejs.jpg"
summary: "Let's deploy a NodeJS app into knative"

---

# {{ page.title }}

## Previously...

In an [earlier post](https://serverascode.com/2022/01/11/tap-knative.html) I deployed a simple demo container image into Kubernetes via knative, and knative itself was installed as part of VMware Tanzu's Cloud Native Runtimes, which is also Part of the Tanzu Application Platform. If that sounds like a lot, that's OK, it is a lot.

```
Tanzu Application Platform (provides ->) Cloud Native Runtimes (provides ->) knative
```

All of this is using the Tanzu Application Platform and in this example is running on a GKE cluster.t

## Deploy a NodeJS application

In this post I'll deploy a NodeJS application into knative.

Again, the same as the previous post, I have a GKE cluster with TAP installed.

I've got no pods running.

```
$ k get pods
No resources found in cnr-demo namespace.
```

Though I still have the knative service I deployed in the last post running.

```
$ /usr/local/bin/kn service list
NAME         URL                                      LATEST             AGE   CONDITIONS   READY   REASON
hello-yeti   http://hello-yeti.cnr-demo.example.com   hello-yeti-00001   17h   3 OK / 3     True
```

Now I want to add a new knative service, but this time in an image that I build, and t he app is running NodeJS.

Here's an [example knative NodeJS app](https://github.com/knative/docs/tree/main/code-samples/serving/hello-world/helloworld-nodejs).

First grab the code.

```
$ git clone https://github.com/knative/docs.git knative-docs
$ cd knative-docs/code-samples/serving/hello-world/helloworld-nodejs
```

There's an index.js file. There's really nothing to it.

```
$ cat index.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  console.log('Hello world received a request.');

  const target = process.env.TARGET || 'World';
  res.send(`Hello ${target}!\n`);
});

const port = process.env.PORT || 8080;
app.listen(port, () => {
  console.log('Hello world listening on port', port);
});
```

Add npm packages.

```
$ npm install
```

I'm using this version of node.

```
$ node --version
v16.13.0
```

I'm going to update the Dockerfile to use 16-slim.

```
$ git diff Dockerfile
diff --git a/code-samples/serving/hello-world/helloworld-nodejs/Dockerfile b/code-samples/serving/hello-world/helloworld-nodejs/Dockerfile
index 14fc5a7f..5593df68 100644
--- a/code-samples/serving/hello-world/helloworld-nodejs/Dockerfile
+++ b/code-samples/serving/hello-world/helloworld-nodejs/Dockerfile
@@ -1,6 +1,6 @@
 # Use the official lightweight Node.js 12 image.
 # https://hub.docker.com/_/node
-FROM node:12-slim
+FROM node:16-slim
 
 # Create and change to the app directory.
 WORKDIR /usr/src/app
```

Build the image.

```
$ docker build -t knative-hello-world-nodejs .
Sending build context to Docker daemon  37.38kB
SNIP!
found 0 vulnerabilities
npm notice
npm notice New minor version of npm available! 8.1.2 -> 8.3.0
npm notice Changelog: <https://github.com/npm/cli/releases/tag/v8.3.0>
npm notice Run `npm install -g npm@8.3.0` to update!
npm notice
Removing intermediate container 05788fe68bda
 ---> 0d87e2185381
Step 5/6 : COPY . ./
 ---> e799b6c92ec9
Step 6/6 : CMD [ "npm", "start" ]
 ---> Running in 28f3c37d21e2
Removing intermediate container 28f3c37d21e2
 ---> 4fc708b92f84
Successfully built 4fc708b92f84
Successfully tagged knative-hello-world-nodejs:latest
```

Tag and push the image to the registry.

```
$ docker tag knative-hello-world-nodejs <registry>/random-builds/knative-hello-world-nodejs
$ docker push <registry>/random-builds/knative-hello-world-nodejs
Using default tag: latest
The push refers to repository [<registry>/random-builds/knative-hello-world-nodejs]
bd83fded2ed1: Pushed
888c1936e335: Pushed
602368557b6e: Pushed
a58aa2b5afe6: Pushed
2c1769b8f2cd: Pushed
b5e79c5c6912: Pushed
18be021c4ec0: Pushed
4a67e24013ff: Pushed
ad6b69b54919: Pushed
latest: digest: sha256:ffe4ba5bed5e9e692d8ca8f441a9209f2d20ab7adef927f0128c027364d1a3e9 size: 2201
```

Aply some knative Kubernetes YAML. (Could use the knative CLI as well, but hey, this time let's write some YAML.)

```
cat << EOF | kubectl create -f -
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: knative-helloworld-nodejs
  namespace: cnr-demo
spec:
  template:
    spec:
      containers:
        - image: <registry>/random-builds/knative-hello-world-nodejs
          env:
            - name: TARGET
              value: "Node.js Sample v1"
EOF
```

Now I've got both my knative services running, one of which is the NodeJS hello world app.

```
$ /usr/local/bin/kn service list
NAME                        URL                                                                 LATEST                            AGE     CONDITIONS   READY   REASON
hello-yeti                  http://hello-yeti-cnr-demo.cnrs.gke.<redacted>                  hello-yeti-00001                  3d12h   3 OK / 3     True    
knative-helloworld-nodejs   http://knative-helloworld-nodejs-cnr-demo.cnrs.gke.<redacted>   knative-helloworld-nodejs-00001   2d19h   3 OK / 3     True   
```

Once that's up and running we can curl the app.

>NOTE: The LB variable is my load balancer fronting the Kubernetes ingress service provided by the Tanzu Application Platform.

```
$ http http://knative-helloworld-nodejs-cnr-demo.cnrs.gke.<redacted>
HTTP/1.1 200 OK
content-length: 25
content-type: text/html; charset=utf-8
date: Sat, 15 Jan 2022 10:54:45 GMT
etag: W/"19-9t2w57sw0IX9vcOiByda5bvW2a4"
server: envoy
x-envoy-upstream-service-time: 2195
x-powered-by: Express

Hello Node.js Sample v1!
```

Hello NodeJS World indeed.

## Building container Images...knative used to do this

As you can see from this post and the previous one, the container image used to run the knative service has to come from somewhere.

When the knative project originated, building images was part of its mandate.

>First version of Knative came with three parts: Serving, Eventing, and Build. These may sound like they are three orthogonal concerns, because they really were. Knative Build was the first part to get separated (and became the Tekton project). - [Did we market Knative wrong?](https://ahmet.im/blog/knative-positioning/)

But, as can be read in the above paragraph, people felt that having build be part of knative was confusing, so a proposal to move build out into Tekton was made:

>This removes Serving optional dependency on Knative Build, making Knative Build fully decoupled from the rest of the Knative components and only responsible to build images that will be using in services later on. This responsibility is shared with any projects capable of building images in Kubernetes. - [
Proposal: Knative Build deprecation in favor of Tekton Pipelines](https://github.com/knative/build/issues/614)

And "knative build" moved out into Tekton.

> Tekton Pipelines is the technological successor to Knative Build. Tekton entities are based on Knative Build’s entities but provide additional flexibility and reusability. This page explains how to convert your Knative Build entities to Tekton entities of equivalent functionality. - [Migrating from Knative Build](https://tekton.dev/docs/pipelines/migrating-from-knative-build/)

Obviously having a container image is key to using knative, so we've got to build one somehow. From a knative project perspective, they moved the build from out of knative and into Tekton. But, is Tekton the best way to build images?

What I can say for sure is that from the perspective of the Tanzu Application Platform, the way we(optionally, but by default) build container images is via the Tanzu Build Service (TBS), which is based on the open source projects [kpack](https://github.com/pivotal/kpack) and [Paketo](https://paketo.io/).

That said, Tekton Pipelines are also installed and used in the Tanzu Application Service (more on that in some other future post) but they are not used to directly *build* images, that's done by TBS, at least by default. To run a container you have to have a container image, and thus to help in simplifying and securing Kubernetes TAP provides that capability. It's a must have feature.

## Conclusion

So there's not much more to this post than the previous one, the one difference being that in this post I build the container image (using docker build) and pushed it to a registry myself, as opposed to using an image that someone, or something, else has built. To build my image I needed to *write/maintain/borrow/steal* a Dockerfile, which is not a small amount of additional work.

Thus, while I'm using knative to help simplify using Kubernetes, which is great, I still have a lot of work to do as a developer to participate in a container deployment workflow...again, for example, having to build and maintain (forever) a container image.

Another developer concern: how do I test all this? What if I change the code? Now I need to manually build the container image? Well, of course, no one would want to do that every time so there are several different ways to solve that problem, but it's still work.

Plus we have all the other fun stuff like how do we observe our app while it's running in production, how do we debug, etc, etc. More to think about!
