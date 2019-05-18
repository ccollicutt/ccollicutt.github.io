---
layout: post
title: DevOps - You Need a Factory
categories:
header_image: "/img/factory.jpg"
header_permalink: "https://unsplash.com/photos/ipyFZIl6Ok0"

---

# {{ page.title }}

A few months ago I listened Kelsey Hightower speak at a function in Toronto. During that talk he said something that has stuck with me for the last few days, and that is that you need a factory.

## DevOps Produces Artifacts

The definition and application of DevOps is oft-debated. However, the fact is that in as technologists we produce things--perhaps they are applications, ie. code, or systems that manage these applications (with automation now also code). Maybe the product is an IoS application or maybe it’s complex infrastructure--OpenStack, Kubernetes, large networks, what have you. Regardless, we create a product. Increasingly that product is generated or operated via the DevOps paradigm.

DevOps has, at this point, a relatively long history and is well documented in terms of its influences: W. Edward Deming, Agile, Toyota Kata, and Lean Manufacturing to name a few. Many of these theories and processes are designed for factories; for the creation of physical products. If we are doing “the DevOps”, and we are automating in the spirit of its influences, such as lean manufacturing, then...where is the factory?

## CI/CD - The (Too) Easy Answer

The easy answer to the factory question is the use of continuous integration and delivery: CI/CD. Many organizations participating in DevOps know that CI/CD is important, but I’m not sure they know why. Often CI/CD is simply “cargo culted” into an organization. (You can usually tell because of the paralysis around selecting a CI/CD system.)

In my opinion, to truly participate in a DevOps model you need a factory, and your CI/CD pipeline is the floor of that factory. The CI/CD system takes input, resources, materials, and work from real live people and produces artifacts which can be evaluated in terms of quality. Like a factory, it can create overstock and has bottlenecks. Most importantly, it is the main system that can be improved over time as part of an ongoing, continuous process.

## Build a (Virtual) Factory

I have talked to many organizations that say they are following the DevOps paradigm. They use small teams, have removed walls between groups, and *cough* use Slack. They might even do some CI/CD...but do they really have a factory? If not, that is something that should be striven for. Without a factory full of people there’s nowhere to continuously improve.