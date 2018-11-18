---
layout: post
title: Configure Bind to Respond with a Single IP to Any Query
categories:

---


# {{ page.title }}

In this post I'll lay out how to setup bind to be authoritative for a single domain, and to respond with a single IP address for any request for that domain.

## Why?

I'm doing a bunch of DNS performance testing. I want to see how fast bind can respond to authoritative domains, but the requests could be for any hostname. Eg. If I request the IP for somerandomhost.example.com I want it to report the same IP as for someotherrandomhost.example.com.

## Configure Bind

Running on Ubuntu 16.04, I first install bind9.

Note that I'm using example.com. It might be better to use a internal domain in your case. At any rate, try to make sure that your testing doesn't egress outside the boundaries of your lab.

```
sudo apt update
sudo apt install bind9
```

Next, configure an example.com zone file in /etc/bind.

```
$TTL 86400
@               IN      SOA     ns.yourdomain.com. hostmaster.yourdomain.com. (
                                2008032701      ; Serial
                                8H      ; Refresh
                                2H      ; Retry
                                1W      ; Expire
                                1D)     ; Minimum
                        NS      ns
*                       A       127.0.0.1
```

Note the "*" line that means respond to any request with 127.0.0.1.

Add the below to named.conf.local to get bind to pickup the example.com domain.

```
zone "example.com" in {
           type master;
           file "/etc/bind/example.com";
};
```

Add these options to named.conf.options. This will disable recursive queries. I'm only going to be testing authoritative requests and don't want external requests at all.

```
recursion no;
additional-from-auth no;
additional-from-cache no;
```

Start/restart bind9.

```
systemctl restart bind9
systemctl status bind9
```

Run a quick test to ensure recursion is not allowed.

```
$ dig @localhost news.google.com

; <<>> DiG 9.10.3-P4-Ubuntu <<>> @localhost news.google.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 47477
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;news.google.com.		IN	A

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Sun Nov 18 11:09:29 UTC 2018
;; MSG SIZE  rcvd: 44
```

Note in the above "WARNING: recursion requested but not available". That is what we want to see: no recursion.

Once that has all been setup and bind9 restarted, we can do something like this:

```
$ dig @localhost `cat /proc/sys/kernel/random/uuid`.example.com

; <<>> DiG 9.10.3-P4-Ubuntu <<>> @localhost 69a65fd2-2223-485b-a6aa-156152db4318.example.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3370
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;69a65fd2-2223-485b-a6aa-156152db4318.example.com. IN A

;; ANSWER SECTION:
69a65fd2-2223-485b-a6aa-156152db4318.example.com. 86400	IN A 127.0.0.1

;; AUTHORITY SECTION:
example.com.		86400	IN	NS	ns.example.com.

;; ADDITIONAL SECTION:
ns.example.com.		86400	IN	A	127.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Sun Nov 18 11:27:45 UTC 2018
;; MSG SIZE  rcvd: 126

```

bind responds that the IP for the host is at 127.0.0.1.

Let's run it once more, with +short.

```
$ dig +short @localhost `cat /proc/sys/kernel/random/uuid`.example.com
127.0.0.1
```

Note how I'm using /proc/sys/kernel/random/uuid to generate a new...er random uuid. Neat huh.

```
$ cat /proc/sys/kernel/random/uuid
f6759215-323d-438b-b4da-535a8aabc63f
```

If you ever need a uuid, that is an easy way to get one without having to install any other software.

## Conclusion

If, for some reason, you want to configure bind to be

* authoritative only (not resolve)
* respond to any request for a single domain to be a single IP

then at this point you should be happy!