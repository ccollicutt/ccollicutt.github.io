---
layout: post
title: Thoughts on "no reliable cloud"
categories:
---

# {{ page.title }}

Recently [Hendrik Volkmer](http://blog.hendrikvolkmer.de/about/) put up a blog post entitled [There will be no reliable cloud](http://blog.hendrikvolkmer.de/2013/04/03/there-will-be-no-reliable-cloud-part-1/). Part of it was based on a [presentation](http://engineering.cloudscaling.com/2013/03/service-resiliency-doesnt-always-mean-ha-or-cluster/) I watched at the last OpenStack summit (wish I was going to the Portland summit, but alas is not to be). 

The Cloud Scaling presentation was one I enjoyed and considered thought provoking. I wrote a [few notes](http://serverascode.com/2012/10/17/openstack-summit-day-3.html) on that presentation last year.

## No reliable cloud

Here's a quote from the top of the [first post](http://blog.hendrikvolkmer.de/2013/04/03/there-will-be-no-reliable-cloud-part-1/:)

> Stop wasting your time trying to [find a reliable cloud]. Stop wasting your time (and money) trying to build one. If you find a service provider that claims that they have it: Maybe question their understanding of cloud - and business.

I put that there to remind me of the point of the series of posts, and because it essentially defines the attention grabbing headline. :)

## tl;dr

My thoughts on these posts come down to this:

- He's mostly talking web-scale applications
- A single zone will not be reliable
- But still have to make zones as reasonably reliable as possible (where's the line?)
- We should design reliable applications on top of unreliable zones (but how?)
- Contain failure!
- HA pairs are probably not the direction to go in to gain reliability
- Clustering software often brings in complexity that can destroy reliability gains
- Stateless systems are a lot more fun :)
- Keep the stateful part of an application or system small

Thinking about reliability in a cloud, especially an OpenStack cloud, is an interesting thought experiment. Fortunately, the OpenStack cloud I help to run, which is the back-end for a single application, is actually mostly stateless--except for machine images, the OpenStack database, and the application database. Not a lot of stateful information, except those darn windows images that are many tens of times the size of a standard Linux cloud image.

## Notes from the part one post

For a [short post](http://blog.hendrikvolkmer.de/2013/04/03/there-will-be-no-reliable-cloud-part-1/) it sure goes over a lot of information and links!

- HA pairs fail catastrophically
- HA pairs don't scale
- Classic HA example: NFS + DRBD and clustering, such as Pacemaker...then problems?
- HA pairs often end up cheating CAP theorem
- Cluster software causes more system outages than hardware failures of software bugs (this I can attend to having used clustered LVM)
- Distributed systems
** Eg. Percona Xtradb Cluster
- Availability vs reliability
** HA systems that need to go down for maintenance are a joke

- How to build a reliable cloud
- A cloud is a distributed system
- Use the stateless (from [Cloud Scaling](http://www.cloudscaling.com/) presentation) approach for stateless parts
- Distributed data stores for the stateful parts (eg. distributed mysql, distributed file systems such as ceph)
- But the distributed stateful part is often what fails (eg. EBS in Amazon)
- Notes from blog post comments (notably Randy Bias of Cloud Scaling)
- On OpenStack
- Move to MySQL Cluster with the NBDEngine running 2-4 mysql instances, and load balancing across them
- Or perhaps OpenStack will get rid of the RDBMS and replace with K/V store
- Even with 1000s of nodes, metadata use is still low in OpenStack, could be put in memory and persist data using any appropriate back-end
- No point in having highly redundant hardware for stateless services
- Build reliable applications on unreliable clouds

Ok, now on to part two.

## Notes from part two

The [second post](http://blog.hendrikvolkmer.de/2013/04/09/there-will-be-no-reliable-cloud-part-2/) builds on the basic information provided in the first.

> Complexity + Scale => Reduced Reliability + Increased Chance of catastrophic failures

- Complexity
- Complex system fail catastrophically
- Failure domains
- OpenStack example
- Single controller, single cloud (or zone)
- HA setup -- two controllers in an HA mode of some kind
- Single controller, multiple cloud (or multiple zones)
- A single zone is unreliable
- If both HA nodes fail, still unreliable, and HA is more complex
- Two zones is two failure domains, which is more reliable than a single HA-enabled zone
- (But of course you should make each zone as reliable as possible)
- Reliability engineering (aka math)
- [Reliability engineering matters except when it doesn't](http://www.infoq.com/presentations/Reliability-Engineering-Matters-Except-When-It-Doesnt)
- "The higher the number of dependent components => the lower the overall availability and the bigger the impact of failure"
- In a cloud with many nodes, adding the ability for live migration will actually decrease reliability, because all nodes are now tied together
- Many reliability calculations come from mechanical engineering, which is much different than software engineering
- Many complex systems fail by cascading, failure starts small and grows big, until it engulfs the entire system
- General approach is to make failure local and contained
- Partial failure is desirable
- Business side
- Software reliability is cheaper
- Most web scale applications consist of a large stateless part and a small stateful piece
- It does not make business sense to  provide a super-reliable cloud
- A single compute node or even zone will never be reliable
- Best not to consider virtual machines, such as those in EC2, [as servers](http://www.jamiebegin.com/why-an-ec2-instance-isnt-a-server/)

NOTE: There will eventually be a part three post, but as of this writing it's not up yet.

## Conclusion

To me, it boils down to building reliable applications on unreliable clouds, which I think is what a lot of people are doing, and is what seems to come out every time AWS fails. 

The first issue that pops into my mind though is RDBMS systems, and how to replicate data between zones, which is often a network concern. Actually, replicating any data between zones could be a problem, which is why, I'm guessing, that he's (perhaps) suggesting to keep stateful pieces small.
