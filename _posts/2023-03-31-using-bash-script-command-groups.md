---
layout: post
title:  "Command Collections/Groups in Bash Scripts"
categories:
header_image: "/img/bash-collections.jpg"
summary: "This is fun...functionality"

---

# {{ page.title }}

I work a lot with Kubernetes. So I need to have Kubernetes clusters. The way that I have usually been building them is with the Killer.sh training courses scripts, which can be found here:

* [https://github.com/killer-sh/cks-course-environment/tree/master/cluster-setup/latest](https://github.com/killer-sh/cks-course-environment/tree/master/cluster-setup/latest)

I decided to take that script and update it and change it around a bit for my liking.

The changes I've made can be found here:

* [https://github.com/ccollicutt/install-kubernetes](https://github.com/ccollicutt/install-kubernetes)

One of the things I found that I liked when writing this script is Bash command grouping.

## Bash Command Collections/Grouping

From the [bash docs](https://www.gnu.org/software/bash/manual/html_node/Command-Grouping.html):

>Bash provides two ways to group a list of commands to be executed as a unit. When commands are grouped, redirections may be applied to the entire command list. For example, the output of all the commands in the list may be redirected to a single stream. 

```
{ list; }
```

Here's a snippet of the install Kubernetes script where I use a grouping.

```
...
### install containerd from binary over apt installed version
function install_containerd(){
  echo "Installing containerd"
  {
    wget -q https://github.com/containerd/containerd/releases/download/v${CONTAINERD_VERSION}/containerd-${CONTAINERD_VERSION}-linux-amd64.tar.gz
    tar xvf containerd-${CONTAINERD_VERSION}-linux-amd64.tar.gz
    systemctl stop containerd
    mv bin/* /usr/bin
    rm -rf bin containerd-${CONTAINERD_VERSION}-linux-amd64.tar.gz
    systemctl unmask containerd
    systemctl start containerd
  } 3>&2 >> $LOG_FILE 2>&1
}
...
```

It's a function that downloads the latest binary of containerd and installs it. Hacky, sure. But it's what I want to have done.

But what you can see here is that all the commands are wrapped into a command group, ie. with the `{}`. This is useful because I can control the output of those commands from one place, where you see the:

```
 } 3>&2 >> $LOG_FILE 2>&1
```

(More on the above later.)

Basically I can take all the output of all the commands, there's seven commands, and manage it with one command, as opposed to tagging a redirection onto each line. I think this is really useful. To create functions and put related commands into command groups. It made it a lot easier for me to understand this script.

## Outputting to a log file

What I wanted to do is have the script have a verbose flag. If that's not set, then don't output anything other than some basic information, like the below.

```
sudo ./install-kubernetes.sh -c -v
Starting install...
==> Logging all output to /tmp/install-kubernetes-XceXczAOta/install.log
Checking Linux distribution
Disabling swap
Removing packages
...
Configuring control plane node...
Initializing the Kubernetes control plane
Configuring kubeconfig for root and ubuntu users
Installing Calico CNI
==> Installing Calico tigera-operator
==> Installing Calico custom-resources
Waiting for nodes to be ready...
==> Nodes are ready
Install complete!
```

But if verbose is set, then show all the output of all the commands.

```
...
### Log file ###
E: Unable to locate package kubelet
E: Unable to locate package kubeadm
E: Can't select installed nor candidate version from package 'kubectl' as it has neither of them
E: Unable to locate package kubernetes-cni
E: No packages found
Reading package lists...
Building dependency tree...
Reading state information...
The following packages will be REMOVED:
  moby-buildx moby-cli moby-compose moby-containerd moby-engine moby-runc
0 upgraded, 0 newly installed, 6 to remove and 13 not upgraded.
After this operation, 401 MB disk space will be freed.
(Reading database ... 
(Reading database ... 5%
(Reading database ... 10%
(Reading database ... 15%
(Reading database ... 20%
...
```

Do that that, I sent all the output to a log file. And if verbose is set, then cat the contents of that file.

But I ran into one problem where because I was doing the command grouping, I couldn't cat the file. 

The error I received:

```
cat: $LOG_FILE: input file is output file
```

So I went to stack overflow and ended up here:

* [https://unix.stackexchange.com/questions/448323/trap-and-collect-script-output-input-file-is-output-file-error](https://unix.stackexchange.com/questions/448323/trap-and-collect-script-output-input-file-is-output-file-error
)

Which gives a fix:

```
#!/bin/bash
...
exit_handler () {
    # 1. Make standard output be the original standard error
    #    (by using fd 3, which is a copy of original fd 2)
    # 2. Do the same with standard error
    # 3. Close fd 3.
    exec >&3 2>&3 3>&-
    cat "$logfile"
    curl "some URL" -F "file=@$logfile"
}
...
```

This kind of hackery makes the script a bit harder to understand, but I still want it to work this way. Have functions, in the functions group commands, and then output the log file if the verbose flag is set. This definitely accomplishes that goal.

## Conclusion

I'm a big fan of command grouping and functions in Bash. Of course Bash has been used like this for years, decades, longer...I'm not sure why I haven't used them as much before. I still have a lot to learn about Bash. The learning never stops. For whatever reason, I really like this particular model of scripting.

* Use functions
* Put commands into command groups
* Control the output into a log file
* If verbose flag, cat the log file

Let me know if you have any thoughts on this model. Thanks!