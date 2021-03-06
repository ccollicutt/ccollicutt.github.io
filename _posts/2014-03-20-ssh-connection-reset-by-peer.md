---
layout: post
title: ssh read from socket failed
categories: 
---

# {{ page.title }}

_UPDATE: This is likely because cloud-init is failing to get information from the various sources it can get information from._

For the people of the future, the ones who get an error message about connections being reset by peer when trying to ssh into a server, check to see if the server does indeed have server keys in /etc/ssh. 

I spent like an hour or more looking into this. I'm not sure why, but an Ubuntu image I was using wouldn't create the servers ssh keys, though the files were there...they were of zero size.

Below is an example of an attempted connection.

<pre>
<code>root@client# ssh -vvvv ubuntu@192.168.122.217
OpenSSH_5.9p1 Debian-5ubuntu1.1, OpenSSL 1.0.1 14 Mar 2012
debug1: Reading configuration data /root/.ssh/config
debug1: /root/.ssh/config line 1: Applying options for 192.168.122.*
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 19: Applying options for *
debug2: ssh_connect: needpriv 0
debug1: Connecting to 192.168.122.217 [192.168.122.217] port 22.
debug1: Connection established.
debug1: permanently_set_uid: 0/0
debug1: identity file /root/.ssh/id_rsa type -1
debug1: identity file /root/.ssh/id_rsa-cert type -1
debug1: identity file /root/.ssh/id_dsa type -1
debug1: identity file /root/.ssh/id_dsa-cert type -1
debug1: identity file /root/.ssh/id_ecdsa type -1
debug1: identity file /root/.ssh/id_ecdsa-cert type -1
debug1: Remote protocol version 2.0, remote software version OpenSSH_6.5p1 Ubuntu-4
debug1: match: OpenSSH_6.5p1 Ubuntu-4 pat OpenSSH*
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_5.9p1 Debian-5ubuntu1.1
debug2: fd 3 setting O_NONBLOCK
debug3: load_hostkeys: loading entries for host "192.168.122.217" from file "/dev/null"
debug3: load_hostkeys: loaded 0 keys
debug1: SSH2_MSG_KEXINIT sent
Read from socket failed: Connection reset by peer
</code>
</pre>

To fix this I regenerated the keys on the server (by logging into the console) and restarted. Then I was able to ssh into the server.

<pre>
<code>root@server# ssh-keygen -q -f /etc/ssh/ssh_host_key -N '' -t rsa1
root@server# ssh-keygen -f /etc/ssh/ssh_host_rsa_key -N '' -t rsa
root@server# ssh-keygen -f /etc/ssh/ssh_host_dsa_key -N '' -t dsa
root@server# service ssh restart
</code>
</pre>

I have no idea why this is happening in this image. Obviously something is broken.
