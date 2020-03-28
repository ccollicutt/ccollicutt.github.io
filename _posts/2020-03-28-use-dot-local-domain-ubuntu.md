---
layout: post
title: Use .local Domain in Ubuntu 18.04
categories:
header_image: "/img/dot-local.jpg"
header_permalink: "https://unsplash.com/photos/acNPOikiDRw"
summary: "Don't tell me what domain I can or can't use!!"

---

# {{ page.title }}

I have a homelab and part of that homelab is a vSphere deployment. For whatever reason I used vsphere.local as the domain for some functionality. But Ubuntu doesn't like the .local domain, I believe because it's usually used with multicast DNS. TBH I'm not going to look to deep into why or why not one should use .local, the fact is that I am and I'm not changing it right now. :)

To use .local in Ubuntu I did this:

```
$ grep Domain /etc/systemd/resolved.conf 
Domains=vsphere.local
```

And moved the "dns" option in /etc/nsswitch.conf to be before mdns...

```
#hosts:          files mdns4_minimal [NOTFOUND=return] dns myhostname
hosts:          files dns mdns4_minimal [NOTFOUND=return]
```

After that I could resolve .local domains.

Happy .local domaining!

