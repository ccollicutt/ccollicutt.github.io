---
layout: post
title:  Merge Kubernetes Config Files
categories:
header_image: "/img/fish.jpg"
summary: "Weirdly there aren't a lot of examples of merging Kubeconfigs, so I wrote it up."

---

# {{ page.title }}

Weirdly there aren't a lot of examples of merging Kubeconfigs, I always end up on stackoverflow.

Basically we use a env var to have multiple kubeconfigs set, the new standalone kube config and ~/.kube/config and merge them with --flatten.

Here are the few CLI steps.

First, make a copy of your kube config.

```
$ cp ~/.kube/config ./kubeconfig-backup 
```

Next, setup a variable, `KUBECONFIG`, to point to both the config files and run `kubectl config view --flatten`, piping the output to a new file. Here I'm use the file name `new-standalone.kubeconfig` but your file name will be different, so change that.

```
$ KUBECONFIG=./kubeconfig-backup:./new-standalone.kubeconfig kubectl config view --flatten > new-kube-config
```

Copy the new config file created to `~/.kube/config`. Note that this over writes your existing config file, but for now you still have the backup copy that we created above.

```
$ cp new-kube-config ~/.kube/config 
```

Validate that your config is working.

```
$ kubectl config get-contexts
```

Finally, if you are sure that your new config is good, remove the copy.

```
$ rm ./kubeconfig-backup
```

That's it!