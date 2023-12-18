---
layout: post
title:  "My Cyberpunk Weekend - Part 3: Using Docker and GPUs"
categories:
header_image: "/img/cyberpunk-3.png"
summary: "Not the game Cyberpunk; the real thing"

---

# {{ page.title }}

I'm working on running LocalAI. But I feel like running that out of Docker.

So how to use a GPU with Docker (on Linux).

First, need the `nvidia-docker2` driver. Otherwise you get an error like this:

```
docker: Error response from daemon: could not select device driver "" with capabilities: [[gpu]].
```

So install that.

```
sudo apt install nvidia-docker2
```

I had a fun thing to fix in that I had added some things to the "daemon.json" so had to fix that.

```
$ sudo dpkg --configure -a
Setting up nvidia-docker2 (2.13.0-1) ...

Configuration file '/etc/docker/daemon.json'
 ==> File on system created by you or by a script.
 ==> File also in package provided by package maintainer.
   What would you like to do about it ?  Your options are:
    Y or I  : install the package maintainer's version
    N or O  : keep your currently-installed version
      D     : show the differences between the versions
      Z     : start a shell to examine the situation
 The default action is to keep your current version.
*** daemon.json (Y/I/N/O/D/Z) [default=N] ? D
--- /etc/docker/daemon.json     2023-04-10 15:23:11.735382489 -0400
+++ /etc/docker/daemon.json.dpkg-new    2023-03-31 09:10:49.000000000 -0400
@@ -1,4 +1,8 @@
 {
-  "registry-mirrors": ["http://10.8.24.123"],
-  "insecure-registries": ["https://some.registry"]
+    "runtimes": {
+        "nvidia": {
+            "path": "nvidia-container-runtime",
+            "runtimeArgs": []
+        }
+    }
 }
```

Next, I have two NVIDIA GPUS, one old one and one newer, better one, the 3090, which is what I want to be using for LLMs.

So, locally I have two, as shown below.

```
$ nvidia-smi 
Mon Dec 18 11:35:39 2023       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.223.02   Driver Version: 470.223.02   CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:06:00.0 Off |                  N/A |
|  0%   32C    P8    12W / 350W |     10MiB / 24268MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   1  NVIDIA GeForce ...  Off  | 00000000:07:00.0 N/A |                  N/A |
| 44%   71C    P0    N/A /  N/A |   2574MiB /  3015MiB |     N/A      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1445      G   /usr/lib/xorg/Xorg                  4MiB |
|    0   N/A  N/A      3231      G   /usr/lib/xorg/Xorg                  4MiB |
+-----------------------------------------------------------------------------+
```

But we can specify to use "device=0" only in the container, so we should only see one GPU.

```
$ docker run -it --gpus "device=0" nvidia/cuda:11.4.3-base-ubuntu20.04 nvidia-smi
Mon Dec 18 16:33:29 2023       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.223.02   Driver Version: 470.223.02   CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:06:00.0 Off |                  N/A |
|  0%   32C    P8    10W / 350W |     10MiB / 24268MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```

BOOM!

One of the hard parts is figuring out what tag to use on the NVIDIA image. They are all listed here:

* [https://gitlab.com/nvidia/container-images/cuda/blob/master/doc/supported-tags.md](https://gitlab.com/nvidia/container-images/cuda/blob/master/doc/supported-tags.md)

Examples:

```
11.4.3-base-ubuntu20.04 (11.4.3/ubuntu20.04/base/Dockerfile)
11.4.3-cudnn8-devel-ubuntu20.04 (11.4.3/ubuntu20.04/devel/cudnn8/Dockerfile)
11.4.3-cudnn8-runtime-ubuntu20.04 (11.4.3/ubuntu20.04/runtime/cudnn8/Dockerfile)
11.4.3-devel-ubuntu20.04 (11.4.3/ubuntu20.04/devel/Dockerfile)
11.4.3-runtime-ubuntu20.04 (11.4.3/ubuntu20.04/runtime/Dockerfile)
```

Note that these will change over time, of course. But if Docker reports it can't find the tag, it's likely because the tag is wrong, or has changed.