---
layout: post
title: Overriding Docker Entrypoint when Running from CLI
categories:
header_image: "/img/entrypoint.jpg"
summary: "How do you change the entrypoint of a Docker image when running from the CLI?"

---

# {{ page.title }}

I have a very simple Dockerfile that I'm using as an example of layers.

```
$ cat Dockerfile 
FROM ubuntu 
RUN echo "layer 1" >> /layers
RUN echo "layer 2" >> /layers
RUN echo "layer 3" >> /layers
RUN echo "layer 4" >> /layers
ENTRYPOINT cat /layers
```

I've built it. Now if I run it...

```
$ docker run --rm layer-example
layer 1
layer 2
layer 3
layer 4
```

But what if I don't want to use that entrypoint? I can override the entyrpoint and then provide another command.

```
docker run --entrypoint "/usr/bin/env" --name layer-example layer-example /bin/bash -c 'while true; do echo sleeping; sleep 2; done'
```

Note that I'm using `env` as the entrypoint, and then the actual command I want ot run is `/bin/bash -c 'while true; do echo sleeping; sleep 2; done'`. 

Interestingly `env` just forwards the command on and runs it.

```
$ /usr/bin/env echo hi
hi
```

If I run it:

```
$ docker run --entrypoint "/usr/bin/env" --name layer-example layer-example /bin/bash -c 'while true; do echo sleeping; sleep 2; done'
sleeping
sleeping
sleeping
sleeping
sleeping
...
```

In another terminal I stop and remove the container via:

```
docker rm -f layer-example
```