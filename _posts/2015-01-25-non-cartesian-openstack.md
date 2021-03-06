---
layout: post
title: Non-Cartesian OpenStack
header_image: /img/bachelard.jpg
categories:
---

_([Gaston Bachelard](https://en.wikipedia.org/wiki/Gaston_Bachelard)_)

# {{ page.title }}

Recently I gave the same talk twice, once at the Calgary OpenStack meetup and the other at a lunch and learn at a Vancouver based IT company.

In the talk I discuss quite a few things, from trying to answer the question as to what OpenStack is, to my [SimpleStack](https://github.com/ccollicutt/simplestack) automated deployment, to armchair philosophy. These topics are related to a central theme of simplicty, complication, and complexity, especially around the idea that OpenStack is difficult to define and deploy.

Some examples of thoughts on complexity:

- At the recent Edmonton Ruby meetup I was talking to someone about what I do for a living (which is essentially work with OpenStack) and they said, "oh yeah, isn't that that thing that's really complicated to install?"
- As I write this there is a [small conversation](http://www.gossamer-threads.com/lists/OpenStack/operators/43442) on the OpenStack-operators list about what a small OpenStack installed (three to five compute nodes) would look like. (Nobody seems to really know.)
- People who I follow on twitter who often say things like "computers are hard" or "how is this [computer] even working?" These are sentiments I can certainly relate to.

## OpenStack Structure Reform

OpenStack has recently decided, or at least I think the technical committee has decided, that their current model of defining components of OpenStack as either [integrated or incubating is not working](http://docs-draft.OpenStack.org/04/138504/5/check//gate-governance-docs/db0e837//doc/build/html/resolutions/20141202-project-structure-reform-spec.html).

> First, the integrated release as it stands today is not a useful product for our users...Skilled operators aren’t deploying “the integrated release”: they are picking and choosing between components they feel are useful. New users, however, are presented with a complex and scary “integrated release” as the thing they have to deploy and manage: this inhibits adoption, and this inhibits the adoption of a slice of OpenStack that could serve their need.

## Non-Cartesian

While staying in Vancouver I went into a local used book store. I needed something to read. I've always wanted to know more about philosophy and who some great thinkers are and their main thesis. While wandering around the store and I found the book [Fifty Key Contemporary Thinkers](http://www.abebooks.com/book-search/isbn/0415074088/) and decided that would be a good place to start.

The first person discussed in that book is Bachelard. On page five, author Lechte provides this passage:

> Whereas Descartes had argued that to progress, thought had to start from the point of clear and simple ideas, Bachelard charges that there are no simple ideas, only complexities.

Lechte further describes this as Bachelard's anti-Cartesian stance, and goes on to say that Bachelard suggests:

> ...that reality is never simple, and that in the history of science attempts to achieve simplicity...have invariable turned out to be over-simplification.

![](/img/continuum.jpg)

## Rejection of over-simplification

I think what has happened in OpenStack recently is the realization that trying to simplify the definition of what it is (ie. all of the "integrated components") isn't working and doesn't work. And, I think, despite not knowing much about philosophy and epistemology, that they are moving in the same general philosophical direction as some of Bachelard's work.

While I'm not even worthy of the title of armchair philosopher, I do think it's interesting to think of how we could consider technology in the same way that epistemologists think about science, and further, that trying to distill a (over?) complicated system like OpenStack down to a simple definition will likely fail.

I end this blog post with a picture of the coordinates of an OpenStack system, so that you can easily find it. :)

![](/img/coordinates.jpg)
