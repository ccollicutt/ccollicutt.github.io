---
layout: post
title: Analysis of OpenStack Austin 2016 NFV/Telco Track
categories:
header_image: /img/chart2.jpg
---

# {{ page.title }}

I went through all the presentations in the [Telco/NFV](https://www.openstack.org/summit/austin-2016/summit-schedule/global-search?t=Telecom+/+NFV) track at the summit to find out what companies were presenting, and how many presenters each had. 

Please note this is not a scientific look at the numbers by any stretch of the imagination:

* I'm not sure about a couple of companies in terms of their size.
* I'm not counting unique presenters, just the occurrences. So if the same person did three presentations I'd count them as three different presenters.
* I wrote this up on 2016-04-14 and things may have changed by then.
* A presentation can have multiple people presenting, or it could be a panel which also has multiple people presenting.
* I took the number of employees each company has from a simple Google search, so not accurate, but ball park. A recent article in the New York Times suggested AT&T has 280000 employees.
* Some companies are owned by larger companies, for example Nuage is owned by Alcatel-Lucent which I think is now owned by Nokia. Or something.
* I did not submit a talk to the Austin 2016 summit.

## Information about Telco/NFV track presentations and presenters

Some basic numbers on the presentations.

| Item                                            | #  |
|-------------------------------------------------|----|
| Presentations                                   | 26 |
| Companies presenting                            | 22 |
| Total Presenters                                | 75 |
| Presenters with no affiliations                 | 2  |
| Presenters with unknown affiliations            | 2  |
| Presenters from companies with > 1000 employees | 62 |
| Presenters from companies with > 100,000 employees | 40 |
| Presenters from non-profit foundations          | 5  |
| Presenters from startups                        | 4  |

<br />

![# of presenters](/img/chart.jpg)

## Topics

The topics are fairly varied. I was hoping to be able to categorize them, but it does seem diverse, at least within the NFV realm.

Below are some keywords I pulled out of the blurb for each talk.

| Keyword                                                                                                |
| -------------------------- | ---------------- | ---------------- | -------------------- | ------------ |
| VNF                        | Distributed NFV  | Service Chaining | Open Source          | vCPE         |
| Panel                      | VM Placement     | Uptime           | Orchestration        | OPNFV        |
| Deployment                 | Service Chaining | IPv6             | OpenStack Congress   | Multi-site Distributed   |
| OpenStack Tacker           | Performance      | SR-IOV           | Monitoring/Telemetry | Latency      |
| Platform Aware Scheduling  | OPNFV Doctor     | CPU Pinning      | Huge Pages           | NUMA         |

<br />

Some of them were mentioned more than once, such as OPNFV, service chaining, and technologies like SR-IOV/CPU Pinning/NUMA/Huge Pages. Multi-site or distributed was mentioned a couple of times. Both OpenStack and OPNFV have ongoing Telecom/NFV related projects such as Congress and Tacker as well as Doctor that should be well represented.

## Number of employees each company has

Below is a table of the larger companies and how many people each employs. It's a considerable sum of over *two million employees!* That's a lot of people.

|       | Company       | # of Employees    |
|----   |-------------- |----------------   |
| 1     | Juniper       | 9500              |
| 2     | AT&T          | 240000            |
| 3     | HP(e?)        | 240000            |
| 4     | BT            | 99000             |
| 5     | Intel         | 107000            |
| 6     | Redhat        | 8000              |
| 7     | Ericsson      | 118000            |
| 8     | Cisco         | 72000             |
| 9     | Canonical     | 700               |
| 10    | PlumGrid      | 250*              |
| 11    | 451 Research  | 250*              |
| 12    | Midokura      | 100*              |
| 13    | Nuage/Nokia   | 62000             |
| 14    | Verizon       | 178000            |
| 15    | Big Switch    | 100*              |
| 16    | China Mobile  | 246000            |
| 17    | NTT           | 242000            |
| 18    | Orange        | 157000            |
| 19    | Brocade       | 4000              |
| 20    | Huawei        | 170000            |
| 21    | NEC           | 102000            |
| 22    | 99Cloud       | Unknown           |

<br />

Again, more than [two million](https://www.youtube.com/watch?v=r0mO6UY6uTg) employees represented from all over the world. Amazing stuff.

Please note for the "*"-ed companies I'm pretty much just guessing.

## Conclusions

The vast majority of presenters in the Telco/NFV track are from extremely large companies. In fact most have more than 100,000 employees. 40 of the 75 presenters are from companies with more than 100,000 employees. 

I'm amazed at how large these companies are. As someone who is employed at a small company that works on NFV projects, I'm not sure what to make of this. Either small companies don't submit presentations, or they are not accepted. Is NFV only important to large Telecoms? Are the only telecoms interested in OpenStack very large companies? Do small telecom companies even exist? Will they in the future?  Who are the people working on NFV in these companies...are they large teams or small, R&D type groups? Do smaller companies help telecoms to deploy OpenStack-based NFV solutions?

On one hand, the results are somewhat obvious: the majority of the companies doing presentations at the OpenStack summit on NFV related topics are very large because telecoms are very large. But on the other I'm surprised there aren't more startups and the like in this mix. There is so much work to be done--so much software to be created--that there is, I believe, considerable opportunity. Perhaps this says something about the ability of a small company to able to participate in the OpenStack ecosystem.
