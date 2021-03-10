---
layout: post
title:  Using xfreerdp from Linux to Windows (with i3)
categories:
header_image: "/img/windows.jpg"
header_permalink: "https://unsplash.com/photos/48yI_ZyzuLo"
summary: "It works pretty darn well!"

---

# {{ page.title }}

I use Ubuntu 20.04. I have a Windows VM, mostly for doing things like Power Point. I like to connect to that VM with xfreerdp so that I can easily flit around and still use i3 on my desktop. I put the xfreerdp session on one of my virtual desktops.

```
xfreerdp /u:USER /p:SOMEPASS /v:SOMEIP /f +fonts /floatbar /smart-sizing -grab-keyboard /sound /microphone /multimon
```

I run the above command, whcih I usually alias. One of the key commands is the `-grab-keyboard` otherwise i3's MOD key might not work.

Then the xfreerdp window pops up. Then I:

* Hit "MOD + SHIFT + Spacebar" to make the window floating
* Hit "MOD" and drag the window to position it properly over the two monitors

That's it. That's the post. I really find this a useful way of working.

Sound works, but I can't get the micrphone to work.

Let me know if you have any ideas to make this better!