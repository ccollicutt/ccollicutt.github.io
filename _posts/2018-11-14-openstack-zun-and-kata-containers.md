---
layout: post
title: OpenStack Zun and Kata Containers
categories:
header_image: /img/kata-containers.jpg
header_permalink: https://thenewstack.io/kata-containers-secure-lightweight-virtual-machines-container-environments/

---

*(diagram of kata containers)*

# {{ page.title }}

In this post I'll explore using DevStack to deploy OpenStack Zun. After that, I'll setup Zun to use Kata Containers as the container runtime.

## OpenStack Zun

So what is OpenStack Zun? The [wiki page](https://wiki.openstack.org/wiki/Zun) for Zun actually does a pretty good job of explaining it:

>Zun is for users who want to create and manage containers as OpenStack-managed resource. Containers managed by Zun are supposed to be integrated well with other OpenStack resources, such as Neutron network and Cinder volume. Users are provided a simplified APIs to manage containers without the need to explore the complexities of different container technologies. Magnum is for users who want a self-service solution to provision and manage a Kubernetes (or other COEs) cluster. 

To unpack that paragraph, Zun provides an API that lets you manage containers. It is not Kubernetes, or Docker, or LXD, or any other Container Orchestration Engine (COE). The Zun API is its own API. It uses many pieces from OpenStack and other container related systems, but it's its own API.

I should say here that if one *does* want a COE, then the [OpenStack Magnum](https://docs.openstack.org/magnum/latest/) project can provide that. What Zun and Magnum do are quite different.

## Kata Containers

>Kata Containers is an open source project and community working to build a standard implementation of lightweight Virtual Machines (VMs) that feel and perform like containers, but provide the workload isolation and security advantages of VMs. -- [Kata Containers](https://katacontainers.io/):

Basically, Kata containers are virtual machines which act as much like containers as possible. 

(I don't want to discuss what containers are in this blog post, as it would take up too much room, but it's worth doing some reading about cgroups, namespaces, etc, and what exactly makes up containers.)

## DevStack + Zun + Kata Containers 

The best way to quickly try out Zun and Kata Containers is to use DevStack to deploy a small OpenStack deployment with the Zun plugin.  I'm mostly following [this document](https://github.com/kata-containers/documentation/blob/master/zun/zun_kata.md) with a few small changes (overall the linked document does work).

First, we need a DevStack with Zun.

DevStack + Zun will be deployed onto Ubuntu 16.04. Note that the stable/rocky release of both DevStack and Zun is used, more recent versions should work as well.

*NOTE: I am disabling Horizon because I kept running into a permissions bug that stops DevStack from completing. By the time you read this perhaps that bug is fixed in DevStack.*

```
apt update
apt install git screen -y

screen -R install 
sudo useradd -s /bin/bash -d /opt/stack -m stack
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
su - stack
sudo mkdir -p /opt/stack
sudo chown $USER /opt/stack
git clone https://github.com/openstack-dev/devstack /opt/stack/devstack
cd /opt/stack/devstack 
git checkout stable/rocky

HOST_IP="$(ip addr | grep 'state UP' -A2 | tail -n1 | awk '{print $2}' | cut -f1 -d'/')"
git clone https://github.com/openstack/zun /opt/stack/zun
cd /opt/stack/zun
git checkout stable/rocky
cd /opt/stack/devstack
cat /opt/stack/zun/devstack/local.conf.sample \
    | sed "s/HOST_IP=.*/HOST_IP=$HOST_IP/" \
    > /opt/stack/devstack/local.conf
sed -i "s/KURYR_CAPABILITY_SCOPE=.*/KURYR_CAPABILITY_SCOPE=local/" /opt/stack/devstack/local.conf
echo "ENABLE_CLEAR_CONTAINER=true" >> /opt/stack/devstack/local.conf
# bug in horizon deployment...
echo "disable_service horizon" >>  /opt/stack/devstack/local.conf
```

Finally, run the stack.sh script to install DevStack.

```
./stack.sh
```

*NOTE: stack.sh can take an hour to complete, depending on network connectivity.*

Once stack.sh completes, we should be able to access OpenStack and Zun.

```
source /opt/stack/devstack/openrc admin admin
# Make a couple of aliases for less typing
alias os=openstack 
alias ac="openstack appcontainer"
os token issue
ac list
```

Hopefully `os token issue` returns a token, and `ac list` runs and shows no containers built yet. If not, then DevStack did not deploy properly.

## Setting up Zun and Kata Containers Post DevStack Deploy

DevStack has deployed OpenStack and Zun, but we still need to setup Zun to use Kata Containers.

To install Kata Containers:

```
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/katacontainers:/release/xUbuntu_$(lsb_release -rs)/ /' >> /etc/apt/sources.list.d/kata-containers.list"
curl -sL  https://download.opensuse.org/repositories/home:/katacontainers:/release/xUbuntu_$(lsb_release -rs)/Release.key | sudo apt-key add -
sudo -E apt-get update
sudo -E apt-get -y install kata-runtime kata-proxy kata-shim
```

Next, we configure Docker to use the kata-runtime.

```
sudo sed -i 's/"cor"/"kata-runtime"/' /etc/docker/daemon.json
sudo sed -i 's/"\/usr\/bin\/cc-oci-runtime"/"\/usr\/bin\/kata-runtime"/' /etc/docker/daemon.json
sudo systemctl daemon-reload
sudo systemctl restart docker
```

At this point Zun should be able to use the kata-runtime.

*NOTE: We could also setup Zun to use kata-runtime as the default. If you'd like to do that, you can add the below line into the default section of `/etc/zun/zun.conf`.*

```
container_runtime = kata-runtime
```

Then restart Zun.

```
for i in `systemctl list-unit-files | grep devstack@zun | cut -f 1 -d " "`; do sudo systemctl restart $i; done
```

If that line is not added, then the default container runtime is runc, but you can request the kata-runtime with an option, `--runtime kata-runtime`.

```
openstack appcontainer run --name kata --runtime kata-runtime nginx:latest
```

With the above command the container will use the kata-runtime. If the default run time is kata-container, then that option is not needed.

## Using Zun and Kata Containers

Now that OpenStack with Zun has been deployed, and Kata Containers installed and Zun/Docker configured to use the kata-runtime, we can finally boot a Kata Container.

First, lets create a Kata based container.

```
openstack appcontainer run --name kata nginx:latest
```

After that command completes, a container should boot up.

```
# openstack appcontainer list
+--------------------------------------+------+--------------+---------+------------+--------------------------+-------+
| uuid                                 | name | image        | status  | task_state | addresses                | ports |
+--------------------------------------+------+--------------+---------+------------+--------------------------+-------+
| c9f8e734-d616-404d-b3e1-425a0882e45b | kata | nginx:latest | Running | None       | 172.24.4.10, 2001:db8::7 | [80]  |
+--------------------------------------+------+--------------+---------+------------+--------------------------+-------+
```

Let's introspect the container and see what runtime it is using.

```
# openstack appcontainer show kata | grep runtime
| runtime           | kata-runtime
```

As can be seen above, it's using the kata-runtime.

Now lets create a runc based container.

```
# openstack appcontainer run --name runc --runtime runc nginx:latest
```

What runtime is it using?

```
# openstack appcontainer show runc | grep runtime
| runtime           | runc   
```

So it's using runc.

Now let's introspect each containers kernel.

Here's the physical hosts kernel version.

```
# uname -a
Linux kata6 4.4.0-134-generic #160-Ubuntu SMP Wed Aug 15 14:58:00 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

Now here's the kernel version of the runc container.

```
# openstack appcontainer exec runc uname -a
Linux 8697605d373b 4.4.0-134-generic #160-Ubuntu SMP Wed Aug 15 14:58:00 UTC 2018 x86_64 GNU/Linux
```

Note that the host kernel and the runc container kernel are the same.

Let's see what kernel is in the kata-runtime based container.

```
# openstack appcontainer exec kata uname -a
Linux a41bb53476f6 4.14.67-139.container #1 SMP Mon Oct 22 22:43:15 UTC 2018 x86_64 GNU/Linux
```

Aha!, that is `4.14.67-139.container` not `4.4.0-134-generic`. So that container is actually a virtual machine with its own kernel.

We can also use the kata-runtime command to list the containers it is supporting.

```
# kata-runtime list
ID                                                                 PID         STATUS      BUNDLE                                                                                                                 CREATED                          OWNER
a41bb53476f64576bcfd2db999f8245c794ea9fce0784594a2239af1c972aaf3   10543       running     /run/containerd/io.containerd.runtime.v1.linux/moby/a41bb53476f64576bcfd2db999f8245c794ea9fce0784594a2239af1c972aaf3   2018-11-14T14:06:12.414823873Z   #0
```

Also, if we do a `ps ax | grep qemu` we can see the container is actually running via a custom qemu process.

```
# ps ax | grep qemu
10526 ?        Sl     0:02 /usr/bin/qemu-lite-system-x86_64 -name sandbox-a41bb53476f64576bcfd2db999f8245c794ea9fce0784594a2239af1c972aaf3 SNIP!
```

Note that it is `qemu-lite-system-x86_64`.

## Conclusion

If you like virtual machines, for whatever reason, perhaps security (all though that is tough to quantify), but you want to use them like containers, then Kata Containers is perfect for you. Also, if you are not interested in using COEs like Docker or Kubernetes, then Zun presents another possibility for obtaining a container.

Ultimately deploying OpenStack and Zun and configuring Zun to use Kata Containers will not be the most common way to use containers (Kubernetes has certainly won that battle). However, it is an *option*, and an option that some might choose to utilize. Diversity is good. :)