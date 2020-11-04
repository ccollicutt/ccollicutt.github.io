---
layout: post
title:  Upgrade Tanzu Kubernetes Grid Multicloud 1.1.3 to 1.2
categories:
header_image: "/img/ocean.jpg"
summary: "Upgrading to TKG 1.2 is easy!"

---

# {{ page.title }}

This is just a quick example of upgrading [Tanzu Kubernetes Grid (multicloud)](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/index.html) 1.1.3 to 1.2. In this example, TKGm is running on vSphere.

For those not familiar, TKG:

>provides a consistent, upstream-compatible implementation of Kubernetes, that is tested, signed, and supported by VMware. Tanzu Kubernetes Grid is central to many of the offerings in the VMware Tanzu portfolio.

TKGm, as I call it, can be deployed into various public clouds, more all the time, and provides the same Kubernetes no matter where it is deployed. 1.2 supports vSphere, Azure, and AWS as host infrastructure, and more will be added over time.

## What's new in 1.2?

* Moving from a separate loadbalancer to kube-vip
* New default CNI: Antrea
* Addition of Harbor as a shared service
* Backup and restore management clusters with Velero

And more!

## Upgrade from 1.1.3 to 1.2 (on vSphere)

The [documenation](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.2/vmware-tanzu-kubernetes-grid-12/GUID-upgrade-tkg-management-cluster.html) for this process is great, and I'm mostly just repeating what it shows. Best to follow those docs, but sometimes having an example is nice.

Initially I have the tkg 1.1.3 CLI.

```
$ tkg version
Client:
	Version: v1.1.3
	Git commit: 0e8e58f3363a1d4b4063b9641f44a3172f6ff406
```

I'm just running one management and one workload cluster.

```
$ tkg get cluster --include-management-cluster
 NAME        NAMESPACE   STATUS   CONTROLPLANE  WORKERS  KUBERNETES        ROLES  
 my-cluster  default     running  1/1           2/2      v1.18.6+vmware.1  <none> 
 tkg-mgmt    tkg-system  running  1/1           1/1      v1.18.6+vmware.1  <none> 
```

So the first step is to download the new 1.2 CLI as well as three OVAs. These artifacts are all downloaded from [VMware](https://my.vmware.com).

* Install the new CLI first.

* Next, upload the three OVAs into vSphere and mark them as templates.

```
Kubernetes v1.19.1: Photon v3 Kubernetes v1.19.1 OVA
Kubernetes v1.18.8: Photon v3 Kubernetes v1.18.8 OVA
Kubernetes v1.17.11: Photon v3 Kubernetes v1.17.11 OVA
```

First we upgrade the management cluster.

* I'm conservative so I copied the old config files first.

```
$ cp -rp ~/.tkg/ ~/.tkg-pre-1.2-upgrade
```

* List the management cluster.

```
$ tkg get management-cluster
It seems that the TKG settings on this system are out-of-date. Proceeding on this command will cause them to be backed up and overwritten by the latest settings.
Do you want to continue? [y/N]: y
the old providers folder /home/ubuntu/.tkg/providers is backed up to /home/ubuntu/.tkg/providers-20201102220133-xryjaxet
The old bom folder /home/ubuntu/.tkg/bom is backed up to /home/ubuntu/.tkg/bom-20201102220133-sk8je1f4
 MANAGEMENT-CLUSTER-NAME  CONTEXT-NAME             STATUS  
 tkg-mgmt *               tkg-mgmt-admin@tkg-mgmt  Success 
```

* Make sure to be using the mgmt cluster context.

```
$ kubectl config use-context tkg-mgmt-admin@tkg-mgmt 
Switched to context "tkg-mgmt-admin@tkg-mgmt".
```

* Add labels (new in 1.2):

```
$ kubectl label -n tkg-system cluster.cluster.x-k8s.io/tkg-mgmt cluster-role.tkg.tanzu.vmware.com/management="" --overwrite=true
cluster.cluster.x-k8s.io/tkg-mgmt labeled
```

* Run the upgrade of the mgmt cluster.

```
$ tkg upgrade management-cluster tkg-mgmt
SNIP!
Patching MachineDeployment with the kubernetes version v1.19.1+vmware.2...
Waiting for kubernetes version to be updated for worker nodes...
Management cluster 'tkg-mgmt' successfully upgraded to TKG version 'v1.2.0' with kubernetes version 'v1.19.1+vmware.2'
```

* Now the mgmt cluster has been upgraded.

```
$ tkg get cluster --include-management-cluster
 NAME        NAMESPACE   STATUS   CONTROLPLANE  WORKERS  KUBERNETES        ROLES      
 my-cluster  default     running  1/1           2/2      v1.18.6+vmware.1  <none>     
 tkg-mgmt    tkg-system  running  1/1           1/1      v1.19.1+vmware.2  management 
```

* Can list the k8s versions. Note 1.19! Nice.

```
$ tkg get kubernetesversions
 VERSIONS          
 v1.17.11+vmware.1 
 v1.17.3+vmware.2  
 v1.17.6+vmware.1  
 v1.17.9+vmware.1  
 v1.18.2+vmware.1  
 v1.18.3+vmware.1  
 v1.18.6+vmware.1  
 v1.18.8+vmware.1  
 v1.19.1+vmware.2  
```

* Finally, upgrade the workload cluster.

```
$ tkg upgrade cluster my-cluster
Logs of the command execution can also be found at: /tmp/tkg-20201102T223108680260342.log
Upgrading workload cluster 'my-cluster' to kubernetes version 'v1.19.1+vmware.2'. Are you sure? [y/N]: y
Validating configuration...
Verifying kubernetes version...
Retrieving configuration for upgrade cluster...
Create InfrastructureTemplate for upgrade...
Upgrading control plane nodes...
Patching KubeadmControlPlane with the kubernetes version v1.19.1+vmware.2...
Waiting for kubernetes version to be updated for control plane nodes
Upgrading worker nodes...
Patching MachineDeployment with the kubernetes version v1.19.1+vmware.2...
Waiting for kubernetes version to be updated for worker nodes...
Cluster 'my-cluster' successfully upgraded to kubernetes version 'v1.19.1+vmware.2'
```

Well, that was pretty easy.

## OPTIONAL: Create a New Cluster

Here I create a new cluster. Note the use of the `--vsphere-controlplane-endpoint-ip` which is available in 1.2 so you can set the k8s API IP address and presumably pre-set the DNS entry for your end users.

```
tkg create cluster --plan dev 1-2-cluster --vsphere-controlplane-endpoint-ip 10.0.6.10
Logs of the command execution can also be found at: /tmp/tkg-20201103T173324591159647.log
Validating configuration...
Creating workload cluster '1-2-cluster'...
Waiting for cluster to be initialized...
Waiting for cluster nodes to be available...
Waiting for addons installation...

Workload cluster '1-2-cluster' created
```

That's it!

## VSPHERE_TEMPLATE Issue

Due to the way I had originally deployed TKGm, I had set the `VSPHERE_TEMPLATE` in the TKG config file and to deploy a new 1.2 cluster I needed to comment that out. Most users won't have this set AFAIK, and it's a simple config file change.

```
$ tkg create cluster --plan dev 1-2-cluster --vsphere-controlplane-endpoint-ip 10.0.6.10
Logs of the command execution can also be found at: /tmp/tkg-20201103T172950440339415.log
Validating configuration...

Error: : workload cluster configuration validation failed: vSphere config validation failed: vSphere template kubernetes version validation failed: unable to get or validate VSPHERE_TEMPLATE for given k8s version: incorrect VSPHERE_TEMPLATE (/Datacenter/vm/photon-3-kube-v1.18.6_vmware.1) specified for Kubernetes version (v1.19.1+vmware.2). TKG CLI will autodetect the correct VM template to use, so VSPHERE_TEMPLATE should be removed unless required to disambiguate among multiple matching templates

Detailed log about the failure can be found at: /tmp/tkg-20201103T172950440339415.log
```

Commented out the option:

```
$ grep -i template ~/.tkg/config.yaml 
VSPHERE_HAPROXY_TEMPLATE: /Datacenter/vm/photon-3-haproxy-v1.2.4-vmware.1
# VSPHERE_TEMPLATE will be autodetected based on the kubernetes version. Please use VSPHERE_TEMPLATE only to override this behavior
#VSPHERE_TEMPLATE: /Datacenter/vm/photon-3-kube-v1.18.6_vmware.1
```


