---
layout: post
title:  "Things I learned: Computer Security Acronyms"
categories:
header_image: "/img/acronyms.jpg"
summary: "There’s a lot of these"

---

# {{ page.title }}

## tl;dr

* I recently starting working in security again, at a great company called [Sysdig](https://sysdig.com/)
* I think the security industry has really been improving as of late...more work to be done, but I see progress
* I need to learn what a lot of security related acronyms mean
* [Here's a good podcast on security acronyms](https://cloud.withgoogle.com/cloudsecurity/podcast/ep94-meet-cloud-security-acronyms-with-anna-belak/)
* [Categorization and definitions are ongoing](https://venturebeat.com/security/gartner-research-finds-no-single-tool-protects-app-security/)

## Background

I started my career in security. That was a long time ago, back when Sun Microsystems was still around and quite popular. You know, before the Dotcom boom. What I remember from that time was 1) I managed a Checkpoint Firewall running on a Sun Microsystems box with 16 interfaces, and when you went to compile the rules often the whole box would crash (not good) and 2) security was just a person that said "NO"...regardless of what the question was, the answer was typically, if not always, no.

(ASIDE: I recently bought a [Sun Microsystems](https://www.redbubble.com/i/t-shirt/Sun-Microsystem-T-Shirt-by-SebastianHapy/109917061.FB110?ref=product-title) shirt off of Redbubble. Well before I wrote this post. Not completely sure why...but I digress.)

To me, the failing firewall wasn't as difficult as saying no. At the time, I didn’t have a lot of experience and made many mistakes, had the wrong (perhaps bad) attitude, but even then, saying no so often was hard on me. I didn't see a good path forward in this part of the industry. It felt like the security world was failing, and eventually I stopped working strictly security focussed jobs and moved into open source infrastructure.

## Security is Improving

Over the last few years I think things have improved in security. It might not seem like it, from a high level, but I see the ecosystem doing a lot of great things. We're getting to the point where we're doing a lot of work to shift security left, moving security more towards development practices, and, for example, starting to try to understand what software makes up our applications (SBOMs and the like). This is good progress. Lots more still to be done, and maybe it can never be "done done", but good progress nonetheless.

I recently came back to the security world, and started working at a great organization called [Sysdig](https://sysdig.com/)--a company that is doing some great work to shift security left while still watching right (i.e. runtime) and was built from the ground up for modern workloads and modern infrastructure.

After a long time off from full time security work there are many newly invented acronyms that I need to learn, which is the real point of this post.

## Acronyms

Here's a few that I've come across so far.

>NOTE: I should say that it's quite possible I've got some things wrong. Let me know if I do. I'll try to keep this up to date.

GRC - Governance, Risk and Compliance

CIEM - Cloud Infrastructure Entitlements Management

KSPM - Kubernetes Security Posture Management

CSPM - Cloud Security Posture Management

SOAR - Security Orchestration Automation and Response

SIEM - Security Information and Event Management

CNAPP - Cloud Native Application Protection Platform

XDR - eXtended Detection and Response

CWPP - Cloud Workload Protection Platform

CASB - Cloud Access Security Broker

RASP - Runtime Application Self-Protection

SAST - Static Application Security Testing

DAST - Dynamic Application Security Testing

IAST - Interactive Application Security Testing

IOC - Indicator of Compromise

TDR - Threat Detection and Response

TI - Threat Intelligence

CVSS - Common Vulnerability Scoring System

DART - Detection and Response Team

CDR - Cloud Detection and Response

VM - Vulnerability Management (not Virtual Machine)

MDR - Managed Detection and Response

## Cloud Security Podcast with Anna Belak

To get better insight into security acronyms than I can provide, have a listen to this podcast:

* [EP94 Meet Cloud Security Acronyms with Anna Belak](https://cloud.withgoogle.com/cloudsecurity/podcast/ep94-meet-cloud-security-acronyms-with-anna-belak/)

## Conclusion

Technology is complicated. We need ways to simplify and understand what all this complex technology does, what it means, and how it works. This is why organizations like Gartner exist. They create functional areas and categories such as "Cloud Security and Posture Management" to help reduce the cognitive load of the vast, vast security ecosystem. In a lot of ways they provide an important function.

However, I think it's paramount to understand that these acronyms and categories are not static, and in some cases not even accurate as to what end users need or are already doing. These acronyms change over time. They come into existence, and they disappear. Sometimes they are popular, other times not so much. They are adjusted over time. They merge and they split apart. I expect that we will see considerable change in these major categories, especially the ones that exist in fast moving areas like modern applications and public clouds as we, as an industry, better understand what problems we have and how best to solve them. On the one hand this might be obvious, but on the other sometimes we put too much faith in these categories.


