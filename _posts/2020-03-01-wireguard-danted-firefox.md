---
layout: post
title: Wireguard, Dante, and Firefox
categories:
header_image: "/img/wg-ff.jpg"
summary: "Proxying Firefox traffic through Wireguard and Danted"

---

# {{ page.title }}

I usually proxy my Firefox through to a remote server running in a public cloud.  Typically I just do that with ssh. 

e.g. I run the below in a separate terminal and just leave it open.  (I could have used autossh, but never quite got around to it.)

```
ssh -D 8888 remote-vpn
```

Then Firefox is configured to use the local proxy. Note that I set it to proxy DNS as well.

![firefox proxy settings](/img/ff-proxy.jpg)


It's a bit of a weird setup, but it's simple and I'm used to it.

## Wireguard Instead of ssh

I've been using Wireguard in another situation, and decided it's time to move from a manually setup ssh command to letting wireguard take care of it.

On local laptop:

```
# pwd
/etc/wireguard
# cat wg0.conf 
[Interface]
Address = 192.168.100.3/32
PrivateKey = <redacted>
ListenPort = 21842

[Peer]
PublicKey = <redacted>
Endpoint =<redacted>:<redacted>
AllowedIPs = 192.168.100.1/32
PersistentKeepalive = 25
```

On the remote server. Note that I'm enabling/disabling nat for the wg0 interface IP based on whether the wg0 interface is up or down.

```
# cat wg0.conf 
[Interface]
Address = 192.168.100.1/24
PrivateKey = <redacted>
ListenPort = <redacted>
PostUp = iptables -t nat -A POSTROUTING -s 192.168.100.1/32 -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -s 192.168.100.1/32 -o eth0 -j MASQUERADE

[Peer]
PublicKey = <redacted>
AllowedIPs = 192.168.100.3/32
```

iptables config. OF course packets must be forwarded too.

```
# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
# iptables -L -n -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
MASQUERADE  all  --  192.168.100.1        0.0.0.0/0  
```

On the laptop, enable and start wg0.

```
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```


And I'm now connected:

```
 $ sudo wg show
interface: wg0
  public key: <redacted>
  private key: (hidden)
  listening port: <redacted>

peer: <redacted>
  endpoint: <redacted>
  allowed ips: 192.168.100.1/32
  latest handshake: 1 minute, 44 seconds ago
  transfer: 149.80 MiB received, 10.38 MiB sent
  persistent keepalive: every 25 seconds
```

ssh is convenient because it can do proxying without any extra work. But that is not so with wireguard. I need a second proxy system. In this case, the easiest thing to use seemed to be dante.

I've configured danted in /etc/danted.conf. (This configuration could probably use some improvement.)

NOTE: Only listening on wg0. Don't put it on the external interface.

```
# cat /etc/danted.conf
logoutput: /var/log/socks.log
internal: wg0 port = 1080
external: wg0
clientmethod: none
socksmethod: none
user.privileged: root
user.notprivileged: nobody

client pass {
        from: 0.0.0.0/0 to: 0.0.0.0/0
        log: error connect disconnect
}
client block {
        from: 0.0.0.0/0 to: 0.0.0.0/0
        log: connect error
}
socks pass {
        from: 0.0.0.0/0 to: 0.0.0.0/0
        log: error connect disconnect
}
socks block {
        from: 0.0.0.0/0 to: 0.0.0.0/0
        log: connect error
}
```

Configure firefox.

NOTE: Firefox I guess doesn't support user/password for proxies? Very weird.

![firefox proxy danted](/img/ff-proxy2.jpg)

Sessions through dante. Note the "nobody" user.

```
# lsof -Pni :1080 | head
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
danted    683 nobody   26u  IPv4 609554      0t0  TCP 192.168.100.1:1080->192.168.100.3:56096 (ESTABLISHED)
danted    685 nobody   20u  IPv4 609980      0t0  TCP 192.168.100.1:1080->192.168.100.3:56166 (ESTABLISHED)
danted    685 nobody   22u  IPv4 609961      0t0  TCP 192.168.100.1:1080->192.168.100.3:56160 (ESTABLISHED)
danted    685 nobody   58u  IPv4 622465      0t0  TCP 192.168.100.1:1080->192.168.100.3:57406 (ESTABLISHED)
danted  30447 nobody    9u  IPv4 575223      0t0  TCP 192.168.100.1:1080 (LISTEN)
danted  32313 nobody   12u  IPv4 619529      0t0  TCP 192.168.100.1:1080->192.168.100.3:57080 (ESTABLISHED)
danted  32313 nobody   16u  IPv4 612166      0t0  TCP 192.168.100.1:1080->192.168.100.3:56324 (ESTABLISHED)
danted  32313 nobody   22u  IPv4 610614      0t0  TCP 192.168.100.1:1080->192.168.100.3:56212 (ESTABLISHED)
danted  32313 nobody   62u  IPv4 618755      0t0  TCP 192.168.100.1:1080->192.168.100.3:57010 (ESTABLISHED)
```

## Conclusion

Using ssh was definitely simpler, but I wanted to try something else, specifically wireguard. But this means 1) setting up wireguard (FYI: is an out of tree kernel module), 2) adding a proxy and 3) configuring nat. At least one valuable option in using wireguard is that I can send all traffic through wireguard if I want to. I'm not right now, but I could.

That said, I need to do some more work related to the proxy configuration, and whether dante is really the best option here. I'll experiment with this setup for a while and determine if there are better options. Do I recommend this setup? I think wireguard is an important technology, but I don't have a great understanding of it yet. So, of course, your mileage may vary.

PS. I also need to check on ipv6 support for this setup, but I don't think my home internet provider supports it (lol).