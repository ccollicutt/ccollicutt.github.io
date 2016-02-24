---
layout: post
title: Nested Virtualization and KVM
categories:

---
 
# {{ page.title }}

Recently I was working on getting virtual machines to boot inside a virtual machine running on Ubuntu Trusty 14.04 (insert [inception](http://www.imdb.com/title/tt1375666/) joke here). However I was getting an error:

<pre>
<code>libvirtError: internal error no supported architecture for os type 'hvm'
</code>
</pre>

In my case that mean that nested virtualization was not turned on in the operating system that is running on the baremetal. (I should note there's another good [blog post here](http://kashyapc.com/2012/01/14/nested-virtualization-with-kvm-intel/) that describes turning on nested virtualization in KVM as well.)

To enable it, I removed the kvm_intel module and re-added it with nested=1.

<pre>
<code># modprobe -r kvm_intel
# modprobe kvm_intel nested=1
</code>
</pre>

Now I should see a "Y" after the below command. (Why it reports a "Y" I don't understand.)

<pre>
<code># cat /sys/module/kvm_intel/parameters/nested
Y
</code>
</pre>

And we are good to go with nested virtualization.

I also setup this file:

<pre>
<code># cat /etc/modprobe.d/kvm_intel.conf 
options kvm_intel nested=1
</code>
</pre>

So that it will be added on a reboot.

Happy nesting!
