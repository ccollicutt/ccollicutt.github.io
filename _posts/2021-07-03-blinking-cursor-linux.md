---
layout: post
title:  How to Fix a Blinking Cursor on Ubuntu Boot
categories:
header_image: "/img/blinking-cursor.png"
summary: "First, don't panic!"

---

# {{ page.title }}

Today I figured I'd update my Linux workstation. Honestly I don't like to do it because stuff breaks all the time. But I don't want an out of date system either, so I just ran it (*eep*), and of course the update crashed midway through, and when I went to hard reboot I had a blinking cursor, which, from an existential perspective is awful and yet perfect at the same time. Thanks Linux! Haha.

Fortunately the fix is pretty easy, certainly easier than fixing urban ennui and lockdown depression.

First, don't panic.

Second, use "CTRL + ALT + F3" to switch to a console, then login from there. (See, Linux is up and running, it's just the GUI login that's borked.)

Third, fix gdm3 and the failed update.

```
sudo dpkg --configure -a
sudo dpkg-reconfigure gdm3
sudo service gdm3 restart
```

And that should fix it. Or at least it did in my case. Best of luck!


