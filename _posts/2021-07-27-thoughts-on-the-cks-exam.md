---
layout: post
title:  Thoughts on the Certified Kubernetes Security Specialist Certification Exam
categories:
header_image: "/img/cks-exam.jpg"
summary: "Time to get really good at vi..."

---

# {{ page.title }}

First, let me say that Kuberentes is an extremely challenging piece of software to use, and, of course, to secure. I work at VMware in the Tanzu group and Kubernetes is a massive part of our portfolio--in fact it's the base of almost everything we do. But it's just the base. You have to add so much on top of Kuberentes to make it useful, and even more to secure it. But enough about that, let's talk about the Certified Kubernetes Security Specialist Certification (CKS).

## tl;dr

* It took me two tries to pass the test, but I didn't do much extra in terms of studying for the second try. I think the first try just made me realize how fast I have to be to complete the test...get fast!
* I used the [Kodecloud](https://kodekloud.com/courses/certified-kubernetes-security-specialist-cks/) and the [Kim Wüstkamp Udemy course](https://www.udemy.com/share/103O5A2@Pm5gfWJgc1MLcUdHC3ZNfj1tYFc=/)
* At this time, when you register for the test you get access to two practice test sessions on [killer.sh](https://killer.sh/). When I initially registered, I did not have access to these from the CNCF, instead they came with the Udemy course. But right now, when you register for the test, you get two sessions on killer.sh "for free". These are extremely helpful, as are any practical lab or questions available.
* I find practical labs and tests to be far the most valuable...videos and other training is not as useful
* If I went through this again I would use the Kodecloud class which has great automated labs, and the two killer.sh practice test sessions

## Some Areas to Focus On

Ultimately, in my opinion, the CKS test is--not surprisingly--a test taking exercise. Two hours is not a lot of time to answer all the questions, and it's really about speed and confidence.

* Ensure you are following all the exam requirements, eg. make sure your desk is clear so you aren't spending time worrying about that just before writing the exam. Note that these requirements can and will change over time, so double check prior to your own exam!
* Use the copy buttons on the test to copy text instead of typing it out (in case you make a typo)
* Be great with the command line, know how to edit with vi--the better you are at vi the faster you will finish questions
* Know how to setup kubectl completion and use it
* Copy initial configs to something like .orig in case you break the deployment
* Go through all the questions and use the notepad to note the number, points, and area of the question
* Do the highest value easiest questions first, moving onto lower value, etc etc
* Get all of the questions you easily can answer first, then move onto the harder questions, but don't leave any high points questions unanswered (if you do, you aren't fast enough yet)
* Double check that you have answered all of the question components before moving on (and if you have time at the end, come back and check)
* Don't worry if you don't pass the first time, you have a second try, and will do much better in the second exam

## From a Technical Perspective

**Kubernetes Services**

Know how to:

* configure the Kubernetes manifests of the major k8s services, such as kube-apiserver
* find the docs for the settings that are available for each service
* configure the services manifests and how they restart
* find the logs for the containers
* use admission controllers, especially ImagePolicyWebhook 

**Kuberentes Config**

Know how to:

* create network policies
* copy an running pod/deploy config and edit it

**3rd Party Tools**

Know how to:

* deploy an apparmor profile
* configure a pod to use an app armor profile
* add rules quickly to Falco and start falco
* implement security best practices on Dockerfiles
* use the docs for these services, but ONLY the ones that have been listed as OK to use during the CKS exam

## Conclusion

First, I have only listed a few major things on this blog post. To pass the test you'd need to know a lot more, and all of that is covered in the documentation for the test, at least in terms of what is on the test.

I personally believe that practical..er...practice is most important in terms of studying for the test. Watch fewer videos and instead practice *actually implementing* practical technical things as quickly as possible via the command line and the vi editor. Make up your own questions if you have to. This is why Killer.sh and the Kodecloud labs and practice tests are so useful. I would spend at least 75% of my time, if not more, on practical hands on (timed if possible) labs and questions as opposed to standard video training. But...this is just my opinion.

Best of luck on your exam!

## PS.

I put up my [CKS Chrome bookmarks](https://github.com/ccollicutt/cks-bookmarks) in github, but again, the allowed sites may change over time so please double check.