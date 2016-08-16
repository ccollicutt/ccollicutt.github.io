---
layout: post
title: Ansible and Ubuntu 16.04 Xenial - Get Python 2.7
categories:
header_image: /img/baby-python.jpg
header_permalink: https://flic.kr/p/63ouQ2 
---

# {{ page.title }}

Quick post on getting Ansible working with an Ubuntu 16.04/Xenial server.

I have a bunch of nodes running in Amazon Web Services (AWS) and I need to put them under configuration management. This is a default Xenial image which does not have Python 2.7 apparently. No problem!

I can use the raw module to get Python 2.7.

~~~bash
ansible_host$ ansible --sudo -m raw -a "apt-get install -y python2.7 python-simplejson" some_group
SNIP!
Setting up python2.7 (2.7.12-1~16.04) ...
Setting up libpython-stdlib:amd64 (2.7.11-1) ...
Setting up python (2.7.11-1) ...
Setting up javascript-common (11) ...
Setting up libjs-jquery (1.11.3+dfsg-4) ...
Setting up python-simplejson (3.8.1-1ubuntu2) ...
~~~

Now I can run ansible ping.

~~~bash
$ ansible -m ping some_group
10.20.1.30 | success >> {
    "changed": false, 
    "ping": "pong"
}

10.20.1.25 | success >> {
    "changed": false, 
    "ping": "pong"
}

10.20.1.113 | success >> {
    "changed": false, 
    "ping": "pong"
}
~~~

Done!

Of course it would probably be a better idea to create an AWS image that has Python 2.7 already installed.

Futher, if you want to do it from a playbook, just make sure to set gather_facts to false.

~~~yaml
---

- hosts: all
  gather_facts: False
  tasks:
    - name: ensure python 2.7 is installed
      raw: apt-get install -y python2.7 python-simplejson
   
    # try pinging now
    - ping:
~~~
