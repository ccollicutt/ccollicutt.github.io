---
layout: post
title: Distributed Cron With Nomad
categories:
header_image: /img/nomad.jpg
header_permalink: https://flic.kr/p/peBMfY 
---

# {{ page.title }}

Everybody loves [Hashicorp!](https://www.hashicorp.com/) Most technologists have probably used [Vagrant](https://www.vagrantup.com/) at some point. Besides Vagrant, Hashicorp has other great technologies, however, in this post I'd like to talk about [Nomad](https://www.nomadproject.io/).

## Nomad

Nomad is:

> A Distributed, Highly Available, Datacenter-Aware Scheduler

Nomad has many use cases. For example it has a Docker driver which means it can handle running Docker containers. I prefer to think of it as a job scheduler. You want something done, you create a job in Nomad and Nomad handles it for you. In this blog post, I describe a specific example: using Nomad to handle cron jobs, such as backing up a MySQL database.

## Caveats

* I should note that the example discussed is in a lab setting. But it is backing up an OpenStack MySQL Galera cluster in a permanent lab deployment, so it's important data.
* Instead of running Nomad as root, I run it as the nomad user. This might get in the way if you are planning on using it to run docker workloads. I don't know because I haven't done that.
* Because I'm not running Nomad as root, I'm allowing the use of raw_exec.
* I'm not using a Consul cluster. Instead I'm using Nomad's built in clustering capability. I would imagine in a large production environment you would have a Consul cluster running.

## Setting Up a Nomad Cluster

One thing I like about Nomad, and various Go-based systems, is that they are distributed in a single binary. Add a user, and a systemd startup file, and you are done. Well, installing anyways.

I have some Ansible that takes care of this, but in way of illustration, below is the configuration on a server member.

First, the base.hcl file.

~~~bash
ubuntu@NODE-1-nomad-server:/etc/nomad$ cat base.hcl 
# Increase log verbosity
log_level = "DEBUG"

# Setup data dir
data_dir = "/var/lib/nomad"

bind_addr = "172.17.5.129"

# Enable debug endpoints.
enable_debug = true
~~~

Now the server.hcl file. The retry join IP is the IP of the first Nomad cluster member.

~~~bash
ubuntu@NODE-1-nomad-server:/etc/nomad$ cat server.hcl 
server {
    enabled = true
    bootstrap_expect = 3
    retry_join = ["172.17.5.129"]
}
~~~

The nice thing about Nomad configuration is that it can be additive, so to speak, in that we can have a base file and then a server configuration and a client configuration. The server configuration would only be deployed to servers, and the client with the clients, but the base configuration to all nodes. It's good for configuration management.

Once the server members come up you can check their status:

~~~bash
ubuntu@NODE-3-nomad-server:~$ nomad server-members -address http://172.17.5.126:4646
Name                           Address       Port  Status  Leader  Protocol  Build  Datacenter  Region
NODE-1-nomad-server.global  172.17.5.129  4648  alive   true    2         0.4.1  dc1         global
NODE-2-nomad-server.global  172.17.5.127  4648  alive   false   2         0.4.1  dc1         global
NODE-3-nomad-server.global  172.17.5.126  4648  alive   false   2         0.4.1  dc1         global
~~~

And we are done with the cluster.

## Setting Up a Nomad Client

Same binary, same startup, same user, same base.hcl, remove server.hcl and add in client.hcl.

The server IPs are those of the three Nomad cluster servers.

~~~bash
ubuntu@NODE-1-mysql-backup:/etc/nomad$ cat client.hcl 
client {
    enabled = true
    servers = [ "172.17.5.129","172.17.5.127","172.17.5.126" ]
    # Enable raw_exec. We are not running nomad as root.
    options = {
        "driver.raw_exec.enable" = "1"
    }
}
~~~

Once the clients startup you can check their status:

~~~bash
ubuntu@NODE-3-nomad-server:~$ nomad node-status -address http://172.17.5.126:4646
ID        DC   Name                    Class   Drain  Status
306dcf6b  dc1  NODE-3-mysql-backup  <none>  false  ready
6a3109d5  dc1  NODE-1-mysql-backup  <none>  false  ready
8e3abc4a  dc1  NODE-2-mysql-backup  <none>  false  ready
~~~

## Periodic Job

Nomad supports [periodic jobs](https://www.nomadproject.io/docs/jobspec/). To my simplistic layperson mind that means cron jobs...but *distributed* cron jobs!

~~~bash
job "mysql-backup" {

  datacenters = ["dc1"]

  type = "batch"

  constraint {
    attribute = "${attr.unique.hostname}"
    regexp = ".*mysql-backup"
  }

  periodic {
    // Launch every hour
    cron = "0 * * * * *"

    // Do not allow overlapping runs.
    prohibit_overlap = true
  }

  task "run-mysql-backup" {
    driver = "raw_exec"

    config {
      # When running a binary that exists on the host, the path must be absolute
      command = "/usr/local/bin/mysql-backup.sh"
    }

    resources {
      # defaults
    }
  }
}
~~~

Certainly the job shown above is overly simplistic. I haven't put in many jobs or read through the jobs documentation properly. But the above job seems to be working and it is using at least one constraint in that the hostname of the nomad client must end in "mysql-backup." I have three mysql-backup nodes running, thus on an hourly basis the job will run on one, and only one, of the three nodes. Each of the three mysql-backup nodes are LXC containers running on a different physical hosts, so the idea is that if one of the containers, or the host, becomes unavailable, the job will still continue to run on the surviving nodes.

Also I should read up more on resources. I don't think the default resources provide much in the way of CPU and memory.

Once the job is added to the cluster, it looks like this:

~~~bash
ubuntu@NODE-3-nomad-server:~$ nomad status -address http://172.17.5.126:4646 mysql-backup
ID                   = mysql-backup
Name                 = mysql-backup
Type                 = batch
Priority             = 50
Datacenters          = dc1
Status               = running
Periodic             = true
Next Periodic Launch = 10/18/16 23:00:00 UTC (37m51s from now)

Previously launched jobs:
ID                                Status
mysql-backup/periodic-1476824400  dead
mysql-backup/periodic-1476828000  dead
~~~

In this example there have only been two runs of the job created, and strangely they are listed as having a status of "dead", but I believe that is actually the correct status as the jobs have completed. Slightly confusing terminology.

~~~bash
ubuntu@NODE-3-nomad-server:~$ nomad status -address http://172.17.5.126:4646 mysql-backup/periodic-1476824400
ID          = mysql-backup/periodic-1476824400
Name        = mysql-backup/periodic-1476824400
Type        = batch
Priority    = 50
Datacenters = dc1
Status      = dead
Periodic    = false

Summary
Task Group        Queued  Starting  Running  Failed  Complete  Lost
run-mysql-backup  0       0         0        0       1         0

Allocations
ID        Eval ID   Node ID   Task Group        Desired  Status    Created At
5e661d1e  373600be  306dcf6b  run-mysql-backup  run      complete  10/18/16 21:00:00 UTC
~~~

## Conclusion

High performance computing experts are probably used to being able to run batch jobs like this, certainly the idea has been around for a long time, but I am not an HPC specialist. I just want a simple distributed (lol) system that can run a backup script on any available node. Plus I like trying out "new stuff" even if it partially implements older ideas.

More to come in future posts!
