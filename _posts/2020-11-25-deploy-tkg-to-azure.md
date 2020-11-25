---
layout: post
title:  Install Tanzu Kubernetes Grid in Azure
categories:
header_image: "/img/azure.jpg"
header_permalink: "https://commons.wikimedia.org/w/index.php?curid=44832559"
summary: "Setup TKG multicloud on Azure is easy."

---

# {{ page.title }}

Tanzu Kubernetes Grid (TKG) 1.2 was recently released, and with it comes the ability to deploy TKG to Azure. Prior to 1.2 you could deploy to vSphere and AWS, but now, with 1.2, Azure is also supported. So you can now run the same Kubernetes with the same life cycle manager across vSphere, AWS, and Azure. That's pretty powerful from a multicloud perspective.

For this post, let's focus on Azure.

>NOTE: Read the official docs [here](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.2/vmware-tanzu-kubernetes-grid-12/GUID-mgmt-clusters-azure.html).

## Requirements

### TKG CLI

Use this [link](https://www.vmware.com/go/get-tkg) to access my.vmware.com and download the TKG CLI. You'll have to login to actually download.

### Azure CLI

Here are the [docs for the prerequisites](https://github.com/kubernetes-sigs/cluster-api-provider-azure/blob/master/docs/getting-started.md#prerequisites) for Azure's Cluster API implementation. But don't worry about reading that doc unless you want to. Not required.

This post assumes that the `az` CLI has been setup and configured enough so that you can run something like:

```
az account show 
```

Next up...

### Docker

TKG uses a local Docker install to setup a small, ephemeral, temporary [kind](https://kind.sigs.k8s.io/) based Kubernetes cluster to build the TKG management cluster in Azure. (More about that later.) Thus we need Docker locally to run the kind cluster.

I'm just using an Ubuntu 18.04 instance with the default docker.io package.

```
$ cat /etc/lsb-release 
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.5 LTS"
$ dpkg --list docker.io
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                   Version          Architecture     Description
+++-======================-================-================-==================================================
ii  docker.io              19.03.6-0ubuntu1 amd64            Linux container runtime
```

The Linux user doing the TKG deploy must be able to access Docker, ie. run something like the below.

```
docker ps
```

Easy peasy.

### ssh key

We'll also need an ssh key in ssh format. If there isn't an ssh key setup, then the below command will create one.

```
ssh-keygen
```

In this post I assume the key is in `~/.ssh/id_rsa.pub`.

## Setup Variables 

First, decide on an Azure application name, such as "tkg-azure" and export that as a variable. This name is up to you.

```
export AZURE_APP_NAME="tkg-azure"
```

Next get your Azure subscription ID.

```
export AZURE_SUBSCRIPTION_ID=$(az account show --query id --output tsv)
```

Now we can setup an Azure application with a service principle.

```
export RETURNED_SP_APP_JSON=$(az ad sp create-for-rbac --name $AZURE_APP_NAME)
```

And use the information returned by that command to configure the other variables we need.

```
export AZURE_CLIENT_ID=$(echo "$RETURNED_SP_APP_JSON" | jq -r '.appId')
export AZURE_CLIENT_SECRET=$(echo "$RETURNED_SP_APP_JSON" | jq -r '.password')
export AZURE_TENANT_ID=$(echo "$RETURNED_SP_APP_JSON" | jq -r '.tenant')
```

Need a couple of standard Azure variables.

>NOTE: `AZURE_ENVIRONMENT` will always be "AzurePublicCLoud" but the `AZURE_LOCATION` can be any region you'd lke. 

```
export AZURE_LOCATION="canadacentral"
export AZURE_ENVIRONMENT="AzurePublicCloud"
```

Also, decide on a management cluster name, such as "tkg-mgmt".

```
export MANAGEMENT_CLUSTER_NAME="tkg-mgmt"
```

Setup a base64 encoded string of your ssh public key. Assuming your key is in `~/.ssh/id_rsa.pub`.

```
export AZURE_SSH_PUBLIC_KEY_B64=$(base64 < ~/.ssh/id_rsa.pub | tr -d '\r\n')
```

At this point we should have this many variables setup:

```
$ env | grep "AZURE\|MANAGEMENT_CLUSTER" | wc -l
9
```

If so, you are good to go to the next step.

## Accept Image License

Accept the license agreement for the images published to Azure. (This only has to be done once.)

```
az vm image terms accept --publisher vmware-inc --offer tkg-capi --plan k8s-1dot19dot1-ubuntu-1804
```

## Deploy Management Cluster

At this point, with only a few Azure az commands, we're setup to build the TKG management cluster.

>NOTE: TKG uses [Cluster API](https://cluster-api.sigs.k8s.io/) to manage the life cycle of Kubernetes clusters. TKG will first deploy a management cluster to Azure, which will contain Cluster API. To do that it uses a local Docker-based Kind cluster to bootstrap the management cluster. Once the management cluster has been bootstrapped into Azure the local Kind cluster is deleted, and going forward TKG will use the Azure based management cluster to build workload clusters. There are a few artifacts that should be kept, eg. `~/.kube`  and `~/.tkg` on the bootstrap node prior to its removal, but once those files are stored and secured the Linux virtual machine could be deleted.

Now we can deploy the management cluster.

Run `tkg get mc` to setup the `~/.tkg` directory.

```
tkg get mc
```

Now build the management cluster.

>NOTE: We are using the `dev` plan.

```
tkg init --infrastructure=azure --name="$MANAGEMENT_CLUSTER_NAME" --plan=dev -v 6
```

With the management cluster deployed, we can now build many workload clusters.

>NOTE: Like many others, I believe that organizations will require many Kubernetes clusters as opposed to a couple large ones. So TKG controls the life cycle of many, many clusters.

Once that command has completed you'll see something like:

```
SNIP!
Deleting Cluster="tkg-mgmt" Namespace="tkg-system"
Deleting ClusterResourceSet="tkg-mgmt-cni-antrea" Namespace="tkg-system"
Resuming the target cluster
Set Cluster.Spec.Paused Paused=false Cluster="tkg-mgmt" Namespace="tkg-system"
Context set for management cluster tkg-mgmt as 'tkg-mgmt-admin@tkg-mgmt'.
Deleting kind cluster: tkg-kind-buv4teb68jjgrg38f0kg

Management cluster created!


You can now create your first workload cluster by running the following:

  tkg create cluster [name] --kubernetes-version=[version] --plan=[plan]
```

At this point we can create workload clusters.

## Create Workload Clusters

Let's create a workload cluster.

```
tkg create cluster workload-01 --plan=dev
```

eg. output looks like:

```
$ tkg create cluster workload-01 --plan=dev
Logs of the command execution can also be found at: /tmp/tkg-20201125T132139302956804.log
Validating configuration...
Creating workload cluster 'workload-01'...
Waiting for cluster to be initialized...
Waiting for cluster nodes to be available...
Waiting for addons installation...

Workload cluster 'workload-01' created
```

Boom! Workload cluster done. That workload cluster can then be used for any Kubernetes applications.

>NOTE: Use `tkg get credentials workload-01` to obtain Kubernetes credentials for the workload cluster.

## Conclusion

At this point we have two clusters, one management, and one workload, and this was all done with a few commands. While the IaaS object configuration will look slightly different in each IaaS, the use of TKG will be the same though all.

>Notice how we have Kubernets 1.19!

```
$ tkg get clusters --include-management-cluster
 NAME         NAMESPACE   STATUS   CONTROLPLANE  WORKERS  KUBERNETES        ROLES      
 workload-01  default     running  1/1           1/1      v1.19.1+vmware.2  <none>     
 tkg-mgmt     tkg-system  running  1/1           1/1      v1.19.1+vmware.2  management 
```

At first glance, the bootstrapping process can seem a bit complex, but it's only because we are using Kubernetes to bootstrap Kubernetes via Cluster API. When you view it through the lens of using Kubernetes constructs where ever possible with the product, it makes sense. Instead of building a separate bootstrapping installer, we use Cluster API, an open source Kubernetes project, the same that is used to build workload clusters. Why re-invent the wheel.

Using TKG gets you the ability to manage the same kubernetes in the same way across many infrastructure as as service products. If multicloud capability is a goal for your organization, then TKG can definitely get you there in terms of Kubernetes. So TKG lets you bootstrap Cluster API onto several common IaaS solutions, thus abstracting away the underlying IaaS. And, of course, it provides life cycle management of Kubernetes clusters.

