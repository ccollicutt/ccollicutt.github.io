---
layout: post
title: My Thoughts on Meltdown and Spectre
categories:
header_image: /img/fire.jpg
header_permalink: https://unsplash.com/photos/AtPbmIH97mA
---

# {{ page.title }}

First off, I don't really expect anyone to care what I think about all this. That's the beauty of the Internet, I can just write what I'd like on my blog and it's easy to ignore this backwater HTML. :)

But I do have some fairly strong thoughts on Meltdown and Spectre, which I consider to be a rather large mess and frankly fairly concerning for the future of humanity. (Certainly computing is only one of our problems though.)

## Computing power

I'm not an academic or a philosopher, but I've always had this strong feeling that humanity would continue to push computing to the farthest it can go, that we would build [Matrioshka](https://en.wikipedia.org/wiki/Matrioshka_brain) brains or other similarly massive computers, and maybe have to move them to cold parts of the universe to run them.

But the Meltdown and Spectre issues have set me back a bit on that thinking. I'm sure it's just a bump in the road, but it is a bit depressing. Computer security is so difficult. Here we are moving everything into the public cloud which is based on CPUs that were never meant to be multi-tenant. If we can't understand the CPUs that we are building now, imagine the issues we will have in the future when we have to deal with even more complexity. Perhaps Meltdown/Spectre will bring about change.

## Computer security

With the recent WIFI hacks, SSL being in a poor state, and now massive CPU issues, things are not looking well (and in fact may never have never been)--I may have to do online banking on a [Raspberry Pi](https://www.raspberrypi.org/blog/why-raspberry-pi-isnt-vulnerable-to-spectre-or-meltdown/).

This [NYT](https://www.nytimes.com/2018/01/06/opinion/looming-digital-meltdown.html) article makes a good point:

>As things stand, we suffer through hack after hack, security failure after security failure. If commercial airplanes fell out of the sky regularly, we wouldn’t just shrug. We would invest in understanding flight dynamics, hold companies accountable that did not use established safety procedures, and dissect and learn from new incidents that caught us by surprise.

## Who was part of the embargo?

One thing that I hope comes out in the near future is a time line of who and what companies knew of the issue. I applaud Google for figuring this out, and letting people know. I think there was a lot of unfairness regarding how this all came about.

## Kernel devs working the Holidays

[Greg Kroah-Hartman](http://kroah.com/log/blog/2018/01/06/meltdown-status/) has a pretty good post on the status of kernel devs.

>Right now, there are a lot of very overworked, grumpy, sleepless, and just generally pissed off kernel developers working as hard as they can to resolve these issues that they themselves did not cause at all. Please be considerate of their situation right now. They need all the love and support and free supply of their favorite beverage that we can provide them to ensure that we all end up with fixed systems as soon as possible.

## Canonical is a small company

I've seen a bit of complaining about Ubuntu not having a patch yet (as of me writing this they do not) but one thing people don't know is that Canonical is a very small company, the best stats I could find suggest ~550 staff. RedHat has ~10,000. That's a big difference. I have a lot of empathy right now for their kernel team.

## Google P0 Team

I've got to give Google props for having the Project Zero (P0) team. Google does provide some pretty useful things for the global community, and P0 is one of the more important. What other large companies or public clouds do the same? Not many.

## Spectre is not fixed...

Something I think is getting a little lost is that Spectre is not fixed. It seems Meltdown is mitigated, with some workloads having considerable performance impact, but Spectre is not solved. Again I go back to Kroah-Hartman's blog. Note the part about "claim."

>Again, if you are running a distro kernel, you might be covered as some of the distros have merged various patches into them that they claim mitigate most of the problems here. I suggest updating and testing for yourself to see if you are worried about this attack vector
>
>For upstream, well, the status is there is no fixes merged into any upstream tree for these types of issues yet. There are numerous patches floating around on the different mailing lists that are proposing solutions for how to resolve them, but they are under heavy development, some of the patch series do not even build or apply to any known trees, the series conflict with each other, and it’s a general mess.

Google [argues](https://blog.google/topics/google-cloud/answering-your-questions-about-meltdown-and-spectre/) that public clouds are better equipped to deal with these issues. Perhaps it's true. Though live migration is available in other IaaS systems, and, of course, if your app is "cloud native" you don't have to live migrate anything, just drain (perhaps using Kubernetes).

>In many respects, public cloud users are better-protected from security vulnerabilities than are users of traditional datacenter-hosted applications. Security best practices rely on discovering vulnerabilities early, and patching them promptly and completely. Each of these activities is aided by the scale and automation that top public cloud providers can offer — for example, few companies maintain a several-hundred-person security research team to find vulnerabilities and patch them before they're discovered by others or disclosed. Having the ability to update millions of servers in days, without causing user disruption or requiring maintenance windows, is difficult technology to develop but it allows patches and updates to be deployed quickly after they become available, and without user disruption that can damage productivity.

## Abstractions get all the hype, and yet we still need baremetal knowledge

Many technical people don't know have much knowledge CPUs work. Or can't compile a Linux kernel. We now have all these high level abstractions (lambda) but still have to have knowledge of low level things. People who I admire on twitter aren't working on figuring out how to deploy to AWS better, and instead are figuring out [gdb](https://jvns.ca/blog/2018/01/04/how-does-gdb-call-functions/) and writing baremetal code in Rust or Go. This is a curious state to be in.

## Diversity in Technology

I believe that diversity in technology is a good thing, and by this I mean different kinds of CPUs and other hardware, as well as operating systems and network protocols, etc. Thus Intel's effective monopoly is not positive. I don't know if trading Intel for AWS is better. I think we need something more.

I know that many organizations are seeking to reduce cost by using commodity hardware, which usually just means virtualization on x86. I think that may be an overly simplistic way to look at the problem of reducing cost.

Perhaps investing in other [CPUs](http://www.tomshardware.com/news/risc-v-not-vulnerable-meltdown-spectre-cpu-bugs,36231.html) is a good idea if only to achieve some diversity and avoid systemic risk. (Frankly it would be pretty cool to have a non-x86 CPU laptop, we should just do that anyways.)

## Conclusion

This is a pretty rambling post, but I think it makes sense given the what's happening this week in technology.

## Bonus: fun tweets

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">It&#39;s impossible to reason about computer security in a meaningful manner anymore.  The gap between &quot;architectural behavior&quot; and &quot;micro-architectural implementation&quot; is so great, so dark, and is basically, &quot;Here be Dragons.&quot;  We cannot build solid structures on faulty foundations.</p>&mdash; Bitweasil (@Bitweasil) <a href="https://twitter.com/Bitweasil/status/949405590631022592?ref_src=twsrc%5Etfw">January 5, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">This means the 6 month embargo of <a href="https://twitter.com/hashtag/meltdown?src=hash&amp;ref_src=twsrc%5Etfw">#meltdown</a> and <a href="https://twitter.com/hashtag/Spectre?src=hash&amp;ref_src=twsrc%5Etfw">#Spectre</a> cost those that weren&#39;t in on the club one full year of time responding to it.  <a href="https://twitter.com/intel?ref_src=twsrc%5Etfw">@intel</a> and <a href="https://twitter.com/Google?ref_src=twsrc%5Etfw">@Google</a> decided who would get that advantage and who wouldn&#39;t.</p>&mdash; John-Mark Gurney (@encthenet) <a href="https://twitter.com/encthenet/status/949344637398863872?ref_src=twsrc%5Etfw">January 5, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="und" dir="ltr"><a href="https://t.co/KRKypYAEiw">pic.twitter.com/KRKypYAEiw</a></p>&mdash; 防毒面を着ているサイバーテロ狼 (@wolfniya) <a href="https://twitter.com/wolfniya/status/948863886131941377?ref_src=twsrc%5Etfw">January 4, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">5 lines of JavaScript broke every single Intel processor made in the past 15 years. <a href="https://t.co/fyQcHk6haJ">pic.twitter.com/fyQcHk6haJ</a></p>&mdash; Mike Pan (@themikepan) <a href="https://twitter.com/themikepan/status/949059784908484608?ref_src=twsrc%5Etfw">January 4, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">“A CPU predicts you will walk into a bar, you do not. Your wallet has been stolen”<br>— The Internet</p>&mdash; Mike Skalnik (@skalnik) <a href="https://twitter.com/skalnik/status/948998374384025600?ref_src=twsrc%5Etfw">January 4, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Don&#39;t panic y&#39;all!<br><br>Step 1) Don&#39;t use Intel processors.<br>Step 2) Don&#39;t use AMD or anything ARM based.<br>Step 3) You know what? Just give up technology altogether.<br>Step 4) Retreat to the woods and build a cabin out of derelict silicon.<br>Step 5) You&#39;re now Ted Kaczynski, you psycho.</p>&mdash; Josh Cincinnati (@acityinohio) <a href="https://twitter.com/acityinohio/status/948741317789564928?ref_src=twsrc%5Etfw">January 4, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">CERT brings the harsh truth. <a href="https://twitter.com/hashtag/Meltdown?src=hash&amp;ref_src=twsrc%5Etfw">#Meltdown</a> <a href="https://twitter.com/hashtag/Spectre?src=hash&amp;ref_src=twsrc%5Etfw">#Spectre</a> <a href="https://t.co/UFPiYA39hd">pic.twitter.com/UFPiYA39hd</a></p>&mdash; Sciuridae Hero (@attritionorg) <a href="https://twitter.com/attritionorg/status/948759303153856512?ref_src=twsrc%5Etfw">January 4, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Update #7 - Due to the incomplete information provided by hardware manufacturers, we joined forces with other impacted cloud providers including <a href="https://twitter.com/linode?ref_src=twsrc%5Etfw">@linode</a>, <a href="https://twitter.com/packethost?ref_src=twsrc%5Etfw">@packethost</a> and <a href="https://twitter.com/OVH?ref_src=twsrc%5Etfw">@ovh</a> to share information and work all together. <a href="https://t.co/iVHi72nmFJ">https://t.co/iVHi72nmFJ</a></p>&mdash; scaleway (@scaleway) <a href="https://twitter.com/scaleway/status/949014513487171585?ref_src=twsrc%5Etfw">January 4, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">8/ <a href="https://twitter.com/Canonical?ref_src=twsrc%5Etfw">@Canonical</a> engineers have been working on this since we were made aware under the embargoed disclosure (Nov 2017) and have worked through the holidays, testing and integrating an incredibly complex patch set into a broad set of <a href="https://twitter.com/ubuntu?ref_src=twsrc%5Etfw">@ubuntu</a> kernels and CPU architectures.</p>&mdash; Dustin Kirkland (@dustinkirkland) <a href="https://twitter.com/dustinkirkland/status/949011894395985920?ref_src=twsrc%5Etfw">January 4, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
