---
layout: post
title: Provision and Configure OpenStack Instances in One Ansible Run 
categories:

---

# {{ page.title }}

In order to configure servers with tools like Ansible, they need to be up and running. These servers could be hardware systems in your data center, or, more likely, virtual machines running in any number of infrastructure-as-a-service (IaaS) providers, including a private OpenStack cloud (which is what I will be using here).

The goal in this blog post is not just to provision the instances, but to provision *and* configure them in *one Ansible playbook run*. This process is somewhat complicated by the fact that you don't know the ip address of the instance until it's created, which typically means provisioning and configuration happens in at least two runs, perhaps even with different tools.

## Ansible modules

Two Ansible modules make what I'm doing possible: [nova_compute](http://docs.ansible.com/nova_compute_module.html) and [add_host](http://docs.ansible.com/add_host_module.html).

_nova_compute_ can create instances in most OpenStack clouds, and _add_host_ can update information about hosts, such as the instances _ansible_ssh_host_ ip address, and do so while the playbook is running.

In combination with a custom inventory script these two modules can be used to provision and configure/converge instances in one playbook run.

## The playbook

In order to use _nova_compute_, at least in the way that I am using it, we need a minimum of five files, which are shown below. I'm using the Ansible roles model to configure the playbook.

1) site.yml
2) hosts
3) group_vars/openstack_instances
4) roles/openstack_instances/tasks/main.yml
5) nova.py - a custom inventory script (ugly as it may be)

Note that there is a [github repository](https://github.com/ccollicutt/ansible-provision-openstack) that has all the files used in this example.

For simplicities sake, all this playbook is going to do is create all of the virtual machines in OpenStack, and then ping them via a role called common.

### nova.py and the hosts file

Because of the [custom inventory script](https://github.com/ccollicutt/ansible-provision-openstack/blob/master/nova.py) I'm using, called nova.py (but not the same as what comes with Ansible by default), I have a somewhat unusual ansible hosts file, though it does follow the typical Ansible hosts file format.

Below we can see there is a group called _openstack_instances_ and there are four instances listed there, each with a flavor_id and group variable associated.

<pre>
<code>curtis$ cat hosts
[openstack_instances]
lb flavor_id=1 group=load_balancers
app flavor_id=2 group=application_servers
db flavor_id=2 group=database_servers
app2 flavor_id=2 group=application_servers
</pre>
</code>

The _nova.py_ script reads each line of the hosts file and sets up the flavor_id and group for each instance. If I run _nova.py_ I get json output that looks like this:

<pre>
<code>curtis$ ./nova.py
{
    "_meta": {
        "hostvars": {
            "app": {
                "flavor_id": "2"
            },
            "app2": {
                "flavor_id": "2"
            },
            "db": {
                "flavor_id": "2"
            },
            "lb": {
                "flavor_id": "1"
            }
        }
    },
    "application_servers": {
        "hosts": [
            "app",
            "app2"
        ]
    },
    "database_servers": {
        "hosts": [
            "db"
        ]
    },
    "load_balancers": {
        "hosts": [
            "lb"
        ]
    },
    "openstack_instances": {
        "hosts": [
            "lb",
            "app",
            "db",
            "app2"
        ]
    },
    "undefined": {
        "hosts": [
            "lb",
            "app",
            "db",
            "app2"
        ]
    }
}
</code>
</pre>

As can be seen above, each inventory entry has it's _flavor_id_ meta variable set, as well as being put into a specific group.

## The openstack_instances file

I've included an example openstack_instances file. Copy that to _group_vars/openstack_instances_ and fill it out with your OpenStack credentials.

<pre>
<code>curtis$ ls group_vars/openstack_instances.example
group_vars/openstack_instances.example
curtis$ cp group_vars/openstack_instances.example
group_vars/openstack_instances
curtis$ vi group_vars/openstack_instances # and enter your credentials
</code>
</pre>

Just as an example, this is what the _nova_compute_ task looks like.

<pre>
<code>curtis$ cat roles/openstack_instances/tasks/main.yml
{% raw %}---

- name: ensure instance exists in openstack
  nova_compute:
    state: present
    login_username: "{{ os_username }}"
    login_password: "{{ os_password }}"
    login_tenant_name: "{{ os_tenant_name }}"
    auth_url: "{{ os_auth_url }}"
    region_name: "{{ os_region_name }}"
    name: "{{ inventory_hostname }}"
    image_id: "{{ os_image_id }}"
    key_name: "{{ os_key_name }}"
    flavor_id: "{{ flavor_id }}"
    security_groups: default
{% endraw %}
</code>
</pre>

## ssh_config

I should also mention that I have a gateway server setup in my OpenStack tenant and that is configured to be used with the private OpenStack network for that tenant. So while my OpenStack instances have a private ip address, they can still be accessed remotely via the ssh gateway server. Another option would be to run Ansible from inside the tenant.

<pre>
<code>host openstack-gw
   Hostname some.floating.ip.address
   User ubuntu
host 10.2.*.*
   ProxyCommand ssh -q openstack-gw netcat %h 22
   User ubuntu
</code>
</pre>

## Run the playbook

Finally, with all those files created and OpenStack credentials entered, we can run the playbook and create the instances if necessary.

<pre>
<code>curtis$ ansible-playbook -i nova.py site.yml

PLAY [openstack_instances] ****************************************************

GATHERING FACTS ***************************************************************
ok: [db]
ok: [lb]
ok: [app]
ok: [app2]

SNIP!

TASK: [common | debug msg="{{ hostname.stdout }}"] ****************************
ok: [app] => {
    "msg": "app"
}
ok: [app2] => {
    "msg": "app2"
}
ok: [db] => {
    "msg": "db"
}
ok: [lb] => {
    "msg": "lb"
}

PLAY RECAP ********************************************************************
app                        : ok=12   changed=2    unreachable=0    failed=0
app2                       : ok=12   changed=2    unreachable=0    failed=0
db                         : ok=12   changed=2    unreachable=0    failed=0
lb                         : ok=12   changed=2    unreachable=0    failed=0
</pre>
</code>

If we run it again, it won't recreate the instances, because they already exist.

<pre>
<code>curtis$ ansible-playbook -i nova.py site.yml

PLAY [openstack_instances] ****************************************************

GATHERING FACTS ***************************************************************
ok: [app]
ok: [app2]
ok: [db]
ok: [lb]

SNIP!

PLAY RECAP ********************************************************************
app                        : ok=11   changed=1    unreachable=0    failed=0
app2                       : ok=11   changed=1    unreachable=0    failed=0
db                         : ok=11   changed=1    unreachable=0    failed=0
lb                         : ok=11   changed=1    unreachable=0    failed=0
</code>
</pre>

If I run nova list I can see the instances. (Note that in the private cloud I am using IPv6 addresses are also provided.)

<pre>
<code>curtis$ nova list | grep "app\|db\|lb"
| 11b0fcae-b296-44fd-9105-ea0edc8e796b | app   | ACTIVE | -          | Running     | private=10.2.0.160, 2605:fd00:4:1001:f816:3eff:fec3:b1b1                 |
| 79fa841a-6cc9-4542-b523-e7f55e13663d | app2  | ACTIVE | -          | Running     | private=10.2.0.127, 2605:fd00:4:1001:f816:3eff:fe9d:3287                 |
| 89085c7b-eee6-4cb2-b2ee-c2bf0cf7931a | db    | ACTIVE | -          | Running     | private=10.2.0.159, 2605:fd00:4:1001:f816:3eff:feda:41f                  |
| 6330e609-463a-47e7-93c3-864d04e5a840 | lb    | ACTIVE | -          | Running     | private=10.2.1.18, 2605:fd00:4:1001:f816:3eff:fe67:aa12                  |
</code>
</pre>

## Conclusion

At this point we have an ansible playook that can provision OpenStack instances, find out their ip address, and then configure them, all in one playbook run. Of course to do this we have to use a custom inventory script, but I don't mind that. Python is a great language to do things like this in, and since Ansible is written in Python it's much easier.

## Issues

- My custom nova.py script isn't very smart.

- Depending on your OpenStack provider you may need to change the name of the network in the _set_fact_ task, which below is set to the name "private." Sometimes different clouds have different default network names.

<pre>
<code>{% raw %}- set_fact: ansible_ssh_host={{ nova.info.addresses.private[0].addr }}{% endraw %}
</code>
</pre>

- Also, sometimes a particular instance won't come up. Just run the playbook again and as long as your OpenStack provider is working, everything should complete at some point. I found that sometimes it would take about a minute for some instances sshd to become available; that Ansible would have connection problems on the first run. Right now there is a 30 second pause in the playbook to try to account for that in a non-intelligent way.

- Another problem could be that OpenStack can have many instances with the same name, so there could be 10 "app2" servers. _nova_compute_ is using the name rather than the OpenStack uuid to see if instances are instantiated. Something to look into because that could go sideways quickly.

- Maybe there is a better way to do this? If so, let me know. :)
