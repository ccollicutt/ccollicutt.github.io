---
layout: post
title:  "Using kubectl run to create privileged container"
categories:
header_image: "/img/run-priv.png"
summary: "Sometimes you need to test if you've disabled it"

---

# {{ page.title }}

This is the whole post. (Is there an easier way to do this?)

```
 kubectl run --rm -i --tty busybox --image=busybox --restart=Never --overrides='{"spec": {"template": {"spec": {"containers": [{"securityContext": {"privileged": true} }]}}}}' -- whoami
```
<br/>
Boom! Now make sure you can't do that in your cluster.