---
layout: post
title:  "My Cyberpunk Weekend - Part 1: The Video Card (Or Neural Core, Take Your Pick)"
categories:
header_image: "/img/cyberpunk-1.png"
summary: "Not the game Cyberpunk; the real thing"

---

# {{ page.title }}

There are a couple of ways to think about this post:

Option 1 (boring):

>*I bought a video card and installed it in my computer.*

Option 2 (cheesy cliché cyberpunk; more fun):

>*In the gray low-rent business suburbs on the edge of the city, where the air hums with the buzz of a thousand illicit transactions, I found myself trudging through a seedy strip mall, its flickering signs casting long shadows over an assortment of massage parlors. Here, amid the cacophony of distant traffic and the murmur of hushed conversation, lay my destination: a dubious, fly-by-night eBay store. The place was a cybernetic bazaar, a maze of used technology and questionable merchandise covered in handwritten labels. Navigating the cramped aisles, I sought a particular treasure--a used AI processor, a critical component for powering my large language model efforts. The store's operators, engaged in a rapid exchange in a language completely foreign to me, barely acknowledged my presence as their faces, etched with lines of weary experience, hesitated for a brief moment before extracting the neural core straight from the guts of a humming, overworked system host. The device, a relic of technological ambition, was burning hot and singed their fingertips, but not enough to deter them from accepting the cash I offered.*

LOL. I'm not sure which one is better. I'll let you decide.

## Generative Artificial Intelligence

Like most people, I have been surprised by the big changes in Artificial Intelligence (AI) over the last few years...surprised, caught off guard, out of the "know" and out of the loop.

Also, like many people, I've been a big user of generative AI, but I don't have a good understanding of how it works. I hope to change that. I want to be able to run Large Language Models (LLMs) locally, so first, I needed to get a video card--a GPU--capable of running these models.

## It's Just a Video Card?

It's kind of amazing that I can use the phrase "video card" in connection with AI; that there's any connection between the two at all. What's a video card for? Connecting to a monitor. Playing video games. But for AI? It's a bit of a stretch, but it's true.

So my first step was to find the right video card, the right graphics processing unit (GPU), to work with AI. After a bit of research it seemed like my best bet, the best value card, was to find a used NVIDIA 3090, mostly because it has 24GB of memory and is a good price at this time.

There were a lot of comments and thoughts on sites like Reddit with this kind of advice:

>*The 3090 is the best bang for your buck. It comes with 24gb of nvram in a single consumer card with a built-in cooling solution and plugs right into your home rig. It lets you run 33b GPTQ models without fuss.*

## Kijiji - The Canadian Craigslist

Living in Toronto has some advantages in that you can find anything you need used--it's out there, you just have to search and wait. It's kind of like what I imagine living in a big city in China would be like - everything is available, you just have to go out and find it, maybe meet some interesting people along the way. 

In Canada we have a site called Kijiji (not even sure how to spell it) which is like Craigslist--but a Canadian Craigslist--so I started looking for a used NVIDIA 3090 GPU with 24GB of memory. 

Of course, there are all kinds of problems with buying a used video card on Kijiji, or anything else for that matter, but I was willing to take the risk in this case. Plus, it can be fun if you don't mind possibly losing the money on a bad purchase. I've bought quite a few things on Kijiji and never had a problem, it's really about finding the right person to buy from, like anything else in life. I've never been ripped off, but you will find some difficult people. I have a whole story about buying a canoe on Kijiji, but that's for another time. Of course, you always want to keep your wits about you and meet in a public place.

I set up a search on Kijiji and there are usually a few 3090s for sale, usually around $1000 to $1200. Then I saw a post from a local person, just a few blocks away in fact, who was selling one for $700. "Quick sale," the ad said. I contacted them, but I wasn't quick enough, and they sold it in a couple of hours before I could get over there. 

Eventually, I saw another ad for a used 3090 that had been pulled from a Dell Alienware workstation for $800 (Canadian) and contacted them about it. They said to give them a few hours notice before coming by to pick it up. Seemed like a good deal, so I said I'd give it a shot. Presumably, if it was from an Alienware computer, it was probably used for gaming, not crypto mining, which is a positive. On the other hand, the people selling it probably knew the value if they were going to part it out, i.e., sell the Alienware box as pieces instead of the entire thing, which has positives and negatives.

A day or two later I went to pick it up. Their store was in a strip mall surrounded by massage parlors, which seemed a little seedy at first because there were more than one, but next to the computer store was a regular car dealership, so I figured it couldn't be that bad. I pulled open the door to the shop, which was so jammed I was not sure I could get it open, and walked into a room completely filled with old computers and a couple of people working feverishly testing them and putting large strips of tape with non-English words on them. Stacks and stacks of computers, half of them falling over.


I told them I was there for the video card and they asked me to wait a few minutes and showed me the card, which looked to be in perfect condition. I asked them if they would benchmark it for me, i.e. put it in a computer and run some tests. They hummed and hawed, but finally agreed to do it. He put it in a computer and ran Furmark and it seemed to work fine. To be honest, I don't know that much about graphics cards or how they're supposed to work, I mostly just watched the temperature and made sure the card was working. While the benchmark was running, they were talking to each other in a language I didn't recognize, so I was never sure exactly what they were saying to each other. Sadly I only speak one language. But they were busy, which means they don't have time to mess around with people. Frankly, they seemed like exactly the kind of place where you'd buy a used video card pulled from a high-end workstation.

During the benchmark, the temperature of the card went up quite a bit, I think around 85 degrees, but I wasn't surprised. I asked them where they sold all these computers, the ones stacked around the place, and the elderly gentleman gruffly gave me a one-word answer: "ebay." Then he went to pull out the card, but didn't let it cool down and almost burned his fingers, which was a bit worrying; you'd think he'd know it was hot. I sure did.

In the end, I paid them $800 cash and took the card home. Surprisingly, they gave me a 30-day warranty card.

It felt very much like a William Gibson-esque cyberpunk experience, and I was happy to have the card.

## Power

In preparation for getting this card, I did some research on maybe building a whole new computer. It was around Black Friday time, so there were a lot of deals. I could have just bought a whole new workstation, but my current one is only a few years old and works just fine. Also, while there was a lot of stuff on sale, there were no good CPUs available; they were all out of stock. Theoretically, I could put the 3090 in my current computer, it would be louder, which is annoying since the computer is in my office, and I would need a new power supply and have to replace it myself, but it should work and it would save some money as well. So for now, I'm just using my existing Linux workstation to host the 3090.

These 3090s can draw up to about 350 watts, which is quite a bit of power. So I had to get a properly sized power supply, as my current workstation only has a 550 watt power supply. I would need a lot more than that, at least 1000 watts. So I started looking for a bigger power supply. I ended up buying a refurbished Corsair RM1000x for $150 from Canada Computers. It's one of the last remaining computer stores in Toronto. That and Memory Express, which doesn't even have a Toronto location. [Canada Computers](https://www.canadacomputers.com/) is about the best place we have to buy computer parts.

## Installation

I'm a bit of an odd person in that I have a lot of computers, like a lot, so much that I won't write it down here. It's just part of what I do for a living, and if you do it for long enough they start to accumulate. However, I don't particularly like computer hardware, especially desktop computers. I don't mind network switches for some reason, and rackmount computers, well, they're okay. But everything else...not much fun. A lot of people take a considerable pride in their workstation setup, LCD lights and all that, but that is not for me. With that in mind, I wasn't super happy about having to change the power supply and open up the computer and move things around, but I did it. It took me a couple of hours, but I did it.

Honestly, the new power supply went in really easily. There was a [Youtube video](https://www.youtube.com/watch?v=yafbKAuyntw&ab_channel=TheProvokedPrawn) that showed my exact power supply and a similar 3090, so that made me feel better about the power swap. I just had to pull three wires and put the new power supply in.

However, my motherboard is a little unusual in that if you use the second M2 slot, the second PCIe slot is disabled, which is where I would put the 3090. I assumed that my NVMe card was in the first slot, so I installed the card and rebooted. But I couldn't see the 3090 from Linux. Looking at the motherboard again, I realized that the technician who built my computer had put the NVMe card in the second slot, probably to get it farther away from the GPU so it wouldn't be affected by the card's heat. As soon as I moved the NVMe card to the first M2 slot, the second PCIe slot was enabled and I could see the 3090!

```
$ nvidia-smi -L | grep 3090
GPU 0: NVIDIA GeForce RTX 3090
```

As you can see, I have an old 660ti as the video card that is connected to my monitors, and the 3090 is the second card. Nice to see the 24GB of memory, which is the whole point of all this "cyberpunk" work!

```
$ nvidia-smi	 
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.223.02   Driver Version: 470.223.02   CUDA Version: 11.4 	|
|-------------------------------+----------------------+----------------------+
| GPU  Name    	Persistence-M| Bus-Id    	Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|     	Memory-Usage | GPU-Util  Compute M. |
|                           	|                  	|           	MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:06:00.0 Off |              	N/A |
|  0%   30C	P8 	8W / 350W | 	10MiB / 24268MiB |  	0%  	Default |
|                           	|                  	|              	N/A |
+-------------------------------+----------------------+----------------------+
|   1  NVIDIA GeForce ...  Off  | 00000000:07:00.0 N/A |              	N/A |
| 34%   53C	P8	N/A /  N/A |	976MiB /  3015MiB | 	N/A  	Default |
|                           	|                  	|              	N/A |
+-------------------------------+----------------------+----------------------+
                                                                          	 
+-----------------------------------------------------------------------------+
| Processes:                                                              	|
|  GPU   GI   CI    	PID   Type   Process name              	GPU Memory |
|    	ID   ID                                               	Usage  	|
|=============================================================================|
|	0   N/A  N/A  	1417  	G   /usr/lib/xorg/Xorg              	4MiB |
|	0   N/A  N/A  	2346  	G   /usr/lib/xorg/Xorg              	4MiB |
+-----------------------------------------------------------------------------+
```

## Cooling

I assume I'll have to find ways to cool this chassis once I start putting the 3090 through its paces.

## Drivers

Because I had the 660ti installed already, I didn't have to add any additional drivers to get the 3090 to show up. Finally a nice piece of luck!

```
$ dpkg --list | grep nvidia-kernel
ii  nvidia-kernel-common-470                   470.223.02-0ubuntu0.20.04.1                   amd64        Shared files used with the kernel module
ii  nvidia-kernel-source-470                   470.223.02-0ubuntu0.20.04.1                   amd64        NVIDIA kernel source package
```

## Conclusion

So far, I've spent about $1000 CDN on this, which isn't too bad. It remains to be seen if my older computer is up to the task of running the 3090; that it doesn't get too hot and too loud; that I don't end up buying a new computer anyway after all this power supply swapping. I might end up doing that if, for example, I decide I want to run multiple GPUs (two 3090s would be optimal) and/or reduce the noise, because I could put the second computer in the basement with all the other computers where I can't hear it, and leave my trusty old relatively quiet workstation in my office.
