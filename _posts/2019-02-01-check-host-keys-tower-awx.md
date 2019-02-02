---
layout: post
title: Check Host Keys in Ansible Tower/AWX
categories:
header_image: "/img/key.jpg"
header_permalink: "https://unsplash.com/photos/pY_AZJfdbHQ"

---

# {{ page.title }}

By default AWX doesn't validate host keys.

Users of plain old Ansible will know this is an option that you can enable or disable--by default in Ansible it's enabled. However, by default in AWX it is **disabled**, which means that AWX does not validate host keys. However, you can enable it. The way that is done is a bit clunky, but it can be done.

In the AWX web interface, go to "Settings -> Jobs". There you will see a section called "EXTRA ENVIRONMENT VAIRABLES." Add `"ANSIBLE_HOST_KEY_CHECKING": "true"` as one of the variables. In my deployment the full configuration looked like this (yours might be different):

```
{
 "HOME": "/var/lib/awx",
 "ANSIBLE_HOST_KEY_CHECKING": "true"
}
```

Then ensure you save your new settings. At this point AWX should now start validating SSH host keys.

*NOTE: I'm not aware of a way to do this outside of using the GUI, but maybe there is? If so, please let me know!*

## Example Failed Job Run

I have an inventory that has already imported some hosts and a job has run against them, and thus the initial host key has already been set in AWX.

In one of the hosts in the inventory I'll force a new host key.

```
ubuntu@c03-01:/etc/ssh$ sudo rm /etc/ssh/ssh_host_*
ubuntu@c03-01:/etc/ssh$ sudo dpkg-reconfigure openssh-server
Creating SSH2 RSA key; this may take some time ...
2048 SHA256:4dFI8aTPISOOZssvPKG+jUnASrft0xTQdWOJhqQLnPo root@c03-01 (RSA)
Creating SSH2 DSA key; this may take some time ...
1024 SHA256:t3bYKZW2FFIUnDxD5uvqpPJbh3h2jjT80GLxttzRU6U root@c03-01 (DSA)
Creating SSH2 ECDSA key; this may take some time ...
256 SHA256:GaONanT4kLLUMJuTH4rghjXfrrahAY0aa98jEL8pXEA root@c03-01 (ECDSA)
Creating SSH2 ED25519 key; this may take some time ...
256 SHA256:qvw390bj+BEOd15NjcHLYtdbk9qzaN5rCE4Xaoi6teQ root@c03-01 (ED25519)
ubuntu@c03-01:/etc/ssh$ 
```

Now that that is done, let's copy the key from `/etc/ssh/ssh_host_ecdsa_key.pub`.

```
ubuntu@c03-01:/etc/ssh$ cat ssh_host_ecdsa_key.pub 
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBK/6oDS05/WR6Cn8camUDFW3OuyRS3ThiTg+8AzS4s68fkP4EUS86IPVjvNb7w180JxGCBA2Dkmi5QdSCPZsLdg= root@c03-01
```

Later we will add that to the AWX task container in `/root/.ssh/known_hosts`.

I have a job that constantly runs. It just ensures all new hosts have python 2.7, for $reasons... 

Because I've changed the host key and set AWX to check host keys, that job now fails for that host.

```
(tower-cli) ubuntu@awx-client:~$ tc job stdout 22314
Identity added: /tmp/awx_22314_cU7Zqd/credential_2 (/tmp/awx_22314_cU7Zqd/credential_2)

PLAY [all] *********************************************************************


TASK [ensure python 2.7 is installed] ******************************************
ok: [c02-03]
ok: [c02-01]
ok: [c02-04]
ok: [c03-02]
ok: [c02-02]
fatal: [c03-01]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@\\r\\n@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @\\r\\n@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@\\r\\nIT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!\\r\\nSomeone could be eavesdropping on you right now (man-in-the-middle attack)!\\r\\nIt is also possible that a host key has just been changed.\\r\\nThe fingerprint for the ECDSA key sent by the remote host is\\nSHA256:GaONanT4kLLUMJuTH4rghjXfrrahAY0aa98jEL8pXEA.\\r\\nPlease contact your system administrator.\\r\\nAdd correct host key in /root/.ssh/known_hosts to get rid of this message.\\r\\nOffending ECDSA key in /root/.ssh/known_hosts:13\\r\\nECDSA host key for 172.20.50.16 has changed and you have requested strict checking.\\r\\nHost key verification failed.\\r\\n", "unreachable": true}
ok: [c01-01]
ok: [c01-02]
ok: [c01-03]

TASK [ping] ********************************************************************
ok: [c02-04]
ok: [c02-03]
ok: [c02-02]
ok: [c02-01]
ok: [c03-02]
ok: [c01-03]
ok: [c01-01]
ok: [c01-02]

PLAY RECAP *********************************************************************
c03-01                 : ok=0    changed=0    unreachable=1    failed=0   
c03-02                 : ok=2    changed=0    unreachable=0    failed=0   
c02-01                 : ok=2    changed=0    unreachable=0    failed=0   
c02-02                 : ok=2    changed=0    unreachable=0    failed=0   
c02-03                 : ok=2    changed=0    unreachable=0    failed=0   
c02-04                 : ok=2    changed=0    unreachable=0    failed=0   
c01-01                 : ok=2    changed=0    unreachable=0    failed=0   
c01-02                 : ok=2    changed=0    unreachable=0    failed=0   
c01-03                 : ok=2    changed=0    unreachable=0    failed=0   


OK. (changed: false)
```

Note the big "IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!" warning.

## Add New Key to AWX and Rerun Job

To set the new host key, I'll login to the AWX host. In my deployment AWX is "containerized" which means AWX is actually made up of several different Docker based containers.

```
root@awx:~# docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                                 NAMES
1169cf7d8918        ssl-haproxy                  "/bin/bash"              4 weeks ago         Up 4 weeks          0.0.0.0:443->443/tcp                                  haproxy
c4bfceeba785        ubuntu:xenial                "/bin/bash"              4 weeks ago         Up 4 weeks                                                                tower-cli
75ac023a773b        ansible/awx_task:2.1.2       "/tini -- /bin/sh -c…"   4 weeks ago         Up 4 weeks          8052/tcp                                              awx_task_1
0773f80c42cc        ansible/awx_web:2.1.2        "/tini -- /bin/sh -c…"   4 weeks ago         Up 4 weeks          0.0.0.0:80->8052/tcp                                  awx_web_1
54d91fe6c00f        memcached:alpine             "docker-entrypoint.s…"   4 weeks ago         Up 4 weeks          11211/tcp                                             awx_memcached_1
1540a632c139        ansible/awx_rabbitmq:3.7.4   "docker-entrypoint.s…"   4 weeks ago         Up 4 weeks          4369/tcp, 5671-5672/tcp, 15671-15672/tcp, 25672/tcp   awx_rabbitmq_1
6b8e0dfd7aab        postgres:9.6                 "docker-entrypoint.s…"   4 weeks ago         Up 4 weeks          5432/tcp                                              awx_postgres_1
root@awx:~# 
```

I'll login to the awx_task host and edit the line for this particular host, adding the ecdsa key.

```
root@awx:~# docker exec -it 75ac023a773b /bin/bash
[root@awx awx]# vi ~/.ssh/known_hosts 
# Add the key to the correct entry
```

The next time the job runs the host key will not cause an error.

```
(tower-cli) ubuntu@awx-client:~$ tc job stdout 22329
Identity added: /tmp/awx_22329_zl5Pd7/credential_2 (/tmp/awx_22329_zl5Pd7/credential_2)


PLAY [all] *********************************************************************

TASK [ensure python 2.7 is installed] ******************************************
ok: [c01-01]
ok: [c02-04]
ok: [c01-02]
ok: [c01-03]
ok: [c03-02]
ok: [c02-01]
ok: [c02-02]
ok: [c03-01]
ok: [c02-03]

TASK [ping] ********************************************************************
ok: [c02-04]
ok: [c01-01]
ok: [c01-03]
ok: [c01-02]
ok: [c03-02]
ok: [c02-01]
ok: [c02-03]
ok: [c02-02]
ok: [c03-01]

PLAY RECAP *********************************************************************
c03-01                 : ok=2    changed=0    unreachable=0    failed=0   
c03-02                 : ok=2    changed=0    unreachable=0    failed=0   
c02-01                 : ok=2    changed=0    unreachable=0    failed=0   
c02-02                 : ok=2    changed=0    unreachable=0    failed=0   
c02-03                 : ok=2    changed=0    unreachable=0    failed=0   
c02-04                 : ok=2    changed=0    unreachable=0    failed=0   
c01-01                 : ok=2    changed=0    unreachable=0    failed=0   
c01-02                 : ok=2    changed=0    unreachable=0    failed=0   
c01-03                 : ok=2    changed=0    unreachable=0    failed=0   


OK. (changed: false)
```

## Conclusion

All that I did in this post was enable  host key checking in AWX and then do a simple verification that it was indeed checking hots keys, and failing with unknown host keys.