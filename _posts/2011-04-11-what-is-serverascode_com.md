---
layout: post
title: What is serverascode.com?
---

# {{ page.title }}

serverascode.com has two major functions: 1) to document how I am working through the process of learning to treat servers as code, and 2) to be my systems administration resume.

Starting with #2, I believe that in my line of work--Unix/Linux systems administration--in todays "market", a two page traditional resume will not get me the job I'm looking for. The only thing that will is a good set of open source contributions; that if servers are code, then much of what I do as a systems administrator should be available on-line in a source code repository. As I write this post, I only have a few lines of code available at [github](https://github.com/ccollicutt/kicker) but as time goes on that will increase.

As far as #1, I love to read about the latest way systems administrator are working; how they are being successful leveraging technology to manage large numbers of often diverse, highly interconnected systems while still ensuring they are highly operational and secure. I read sites like Hacker News and other blogs that describe a sort of “new age” methodology to systems administration; so called "devops" or "agile sysadmin" or just plain 2010+ systems administration.  However, I have not had an opportunity to apply those concepts, and I would like to use this blog, and services like github, to begin treating a @server as code@.

For better or for worse, systems administration has changed greatly--and will continue to change--especially with the recent trend in virtualization. I believe at the core of this change is the concept of treating server(s) as code. My feeling is that in 2010+ we don't admin a server, we code it. From procurement to deployment to maintenance to decommissioning--it's all code now.

On top of every Linux distribution, which can perhaps now be a called a OS framework, comes configuration conventions, and a packaging system. Then, over top of the framework, we add a centralized management server instance (perhaps the only server I should be logging into) which runs configuration management software, such as chef, puppet, and others, which control installation and configuration of applications, alerting things such as change management, and other systems I have not yet determined.

Suffice it to say that I am looking forward to working towards treating @servers as code@ and documenting that process in blog posts and, hopefully more-so, as code!
