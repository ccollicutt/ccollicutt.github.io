---
layout: post
title: Connecting to Juniper with Ansible
categories:
header_image: /img/network.jpg
header_permalink: https://unsplash.com/photos/Ud4GcZW3rOY
---

# {{ page.title }}

Recently I wanted to test out using Ansible to connect to Juniper vSRX routers. Unfortunately this was not as easy as I had thought/hoped it would be.

## Juniper vSRX

Juniper provides a virtual machine image for the virtual SRX firewall.

The version I'm using here is:

```
media-vsrx-vmdisk-17.3R1.10.qcow2
```

I'm running it in kvm on Ubuntu 16.04...but that's another story.

## Configure vSRX

This is the easy part.

```
set system services netconf ssh
set system services ssh hostkey-algorithm no-ssh-ecdsa
set system services ssh hostkey-algorithm ssh-rsa
```

*NOTE: In the future the ssh hostkey-algorithm settings might not be needed. More on this later in the post.*

Also, the `known_hosts` file for where ever Ansible is running would need to be setup. HINT: ssh-keyscan.

## Setup Playbook

Here's an example playbook. Note the `provider` section. This took a little while to get right. The `junos_facts` module requires some connection information, and it uses a dictionary called `provider` that can be setup in the task. Obviously these variables could be pulled from somewhere else, like an Ansible Vault file, but I'm leaving it bare as an example here.

Ansible's default facts gathering doesn't understand how to gather facts from Juniper, so the `juniper_facts` module is required to get facts. Note I'm also gathing a subset of information in the example.

```
{% raw %}- name: test
  hosts:
    - routers
  roles:
    - Juniper.junos
  connection: local
  gather_facts: no
  tasks:

    - name: check is netconf port is listening
      wait_for:
        host: "{{ ansible_host }}"
        timeout: 5
        port: 830

    - name: collect default set of facts and configuration
      junos_facts:
        gather_subset: "config"
        provider:
            username: "root"
            password: "coolpass"
            transport: "netconf"
            timeout: 5
            host: "{{ ansible_host }}"{% endraw %}
```

<br />

## Issues with ECDSA keys

It seems that the ncclient code can't handle, at this time, ECDSA keys. It just fails with an "unknown key" message.

I had to disable ECDSA keys in the Juniper routers.

I tried this super simple test code, prior to forcing ECDSA on the Junipers, and it errored out with hostkey_verify=True, and succeeded with it set to False.

```
$ cat test-netconf.py
from ncclient import manager

with manager.connect(host="<hostname>", port=830, username="root", password="coolpass", hostkey_verify=True) as m:
    c = m.get_config(source='running').data_xml
    print c
```

As far as I know at this time this is the latest ncclient code.

```
$ pip freeze | grep ncclient
ncclient==0.5.3
```

Maybe there is another issue hidden away somewhere. Worth some more digging when I have time.

## Conclusion

Overall, I'm not happy with how this looks. I like Ansible, but sometimes the whole module thing, which exists with almost all configuration management tools, just seems hacky. Here I have to turn of fact gathering, and then turn on a task based Juniper fact gathering. Further, getting this to work with a recent vSRX node took like an hour due to the ECDSA issue. One simple little thing like that breaks connectivity via Ansible. Dependencies and versioning...these are hard and don't seem to be getting any easier, this is too brittle.

I know the Ansible and networking (vendors) have done a lot of work to get many devices supported, it just doesn't seem quite right to me. That said, Ansible is probably one of the best options for managing many individual network devices.
