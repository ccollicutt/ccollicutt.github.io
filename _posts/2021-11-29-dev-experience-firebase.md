---
layout: post
title:  "Dev Experience: Writing a NodeJS REST API with Firebase"
categories:
header_image: "/img/dx-firebase.jpg"
summary: "Starting my walkabout amongst the dev platforms"

---

# {{ page.title }}

## Developer Experience

Working at [VMware in the Tanzu group](https://tanzu.vmware.com/) I'm always focussed on developer experience (DX). Often people think of VMware as an infrastructure only company, but we're not. Tanzu is heavily focussed on developers. I'd say 10-20% of what we do is infrastructure related (Kubernetes, Cloud Foundry) and the other 80-90% is related to devops, security, developers, and software architects to name a few. What's the point of platforms if there's no apps running on them.

Overall Tanzu is working extremely hard on improving DX. For example we recently released beta versions of the [Tanzu Application Platform](https://tanzu.vmware.com/content/blog/announcing-vmware-tanzu-application-platform), a way to de-expose kubernetes to developers...by that I mean abstract it away into a 15 lines of YAML instead of 2000.

But let me get to the point of this post--**developers should not be futzing around with Kubernetes**. They should be able to write code and put apps in production as easily as possible.

With that in mind, I like to keep my eye on any products or tools or platforms that can improve DX. I decided to take a look at Google's Firebase to see what it's like, and how it helps DX. While Firebase has been around for a long time, I've never taken a look at it to understand what it does. Time to change that. :)

## Itch to Scratch - Simple hostname/arecords REST API

Historically I'm not a developer (surprise!). I don't write code every day, and I don't normally have a reason to. But I have had an "itch to scratch" so to speak for a while, in that I want a way to easily manage my home DNS server. I have a homelab and it requires many host, ie. arecord, entries. My internal DNS server is dnsmasq and it can run off of the entries in `/etc/hosts`. So when I add internal DNS entries, I just add them to the `/etc/hosts` file on the dnsmasq server and that's it. 

I wanted an API and CLI that I can use to easily do that, and then (eventually) a templated API response that will generate `/etc/hosts` (and other config files) for me based on those entries. The idea is that if I build the main API, then I can add microservices that can template out configuration files for any DNS server (not just dnsmasq). But that's down the road....

The thing I need to build first is a simple REST API for managing DNS arecords.

## What do I want?

Base requirements:

* Runs in "the cloud"; no infra required
* Document database
* Functions (no exposed k8s)
* NodeJS support
* Easy push to prod
* Low cost entry (hopefully free for small projects)
* Easy local development

I know SQL is making a comeback in terms of the marketplace (not that it left) but for this use case I'm interested in a document database.

## Firebase

### History

>Firebase is a platform developed by Google for creating mobile and web applications. It was originally an independent company founded in 2011. In 2014, Google acquired the platform[1] and it is now their flagship offering for app development. 

Firebase has been around for 10 years. Much like App Engine, it doesn't get much press, and I'm not even sure how I came across it, but after reading a bit, it seemed like an interesting platform to look at in terms of DX.

This post, [What is Firebase, The complete story, abridged](https://medium.com/firebase-developers/what-is-firebase-the-complete-story-abridged-bcc730c5f2c0), which I didn't read until I started writing this post, gives some good perspective on what Firebase is, and isn't. 

### My Experience with Firebase

Here are my base base requirements.

| Item  |  Supported  | Comment   | 
|-------|-------------|-----------|
| nodejs  | yes  | first class, but hard to tell what version?   |
| document database | yes | firestore  |
| functions | yes  | cloud run with deep integration, but need "blaze" plan level  |
| low cost | yes | great for small projects like mine |

But what else might one need for a good development platform? Here's a few I thought about in this context.

>NOTE: Please understand that I don't write code every day. More experienced developers will have different opinions of what is important and what isn't.

| Item  |  Supported  | Comment   | 
|-------|-------------|-----------|
| local development  | yes   | firestore emulator is amazing |
| getting to prod  | yes  | easy as firebase deploy |
| authentication  |  yes | hard for me to grasp difference between admin sdk and other users  |
| data schema  | yes, for users  | filebase.rules is great, but doesn't apply to admin sdk   |
| data indexes | yes  | need to manually apply them with cli or gui  |
| logging  | yes  | didn't explore  |
| metrics  | yes  | didn't explore  |
| testing  | yes  | didn't explore  |
| ci/cd integration  |   | didn't explore |

Certainly developers need a lot more than this, but I didn't want to write out 100 needs. 

## Some Things I Ran Into Using Firebase

* Functions are not available in the "free tier" you have to go up to the Blaze level, which still has a free tier that is fairly substantial for a small app like mine, one which might see a few requests per day at most
* Firebase rules (ie. data schemas) don't apply to the admin sdk...lost some time on this
* I had trouble figuring out what nodejs version is supported

## Some Results

Here's using a simple httpie based script to talk to the API and perform CRUD operations. Right now I'm calling this project "Sparrow" for some reason.

```
$ ./scripts/sparrow-cli arecord add www.example.com 192.168.88.10
{
    "arecord": {
        "_id": "JWgEXUlfVdK5FUQeLeBr",
        "ip": "192.168.88.10",
        "name": "www.example.com"
    }
}
info: added arecord
$ ./scripts/sparrow-cli arecord list
[
    {
        "_id": "JWgEXUlfVdK5FUQeLeBr",
        "ip": "192.168.88.10",
        "name": "www.example.com"
    },
    {
        "_id": "K3fwiWQOSS3wc6srpv9h",
        "ip": "10.10.10.10",
        "name": "new2.domain.com"
    },
    {
        "_id": "O8OIERUTclPpg75cIAlq",
        "ip": "10.10.10.10",
        "name": "new.domain.com"
    },
    {
        "_id": "ytgACQTVZZyXJi0pYPJ5",
        "ip": "10.0.10.10",
        "name": "some.domain.com"
    }
]
```

The script uses these environment variables to connect. The API keys have nothing to do with Firebase...they're part of the app.

```
$ env | grep SPAR
SPARROW_API_ENDPOINT=https://us-central1-<my firebase project>.cloudfunctions.net/api
SPARROW_API_KEY=<key>
SPARROW_API_SECRET_KEY=<secret>
```

Pretty straightforward, simplistic stuff...and yet, it's a perfectly usable REST API.

## Conclusion

Overall I used Firebase to provide a place to run nodejs functions that talk to a document store, where the functions and document store are provided by the platform. As well I made heavy use of the firebase CLI and emulator to test locally. 

With just over 100 lines of NodeJS I was able to write a functional REST API for my arecords app requirement.

```
$ cloc --exclude-list-file=.clocignore .
    8045 text files.
    7117 unique files.                                          
    8051 files ignored.

github.com/AlDanial/cloc v 1.82  T=0.93 s (6.5 files/s, 293.3 lines/s)
--------------------------------------------------------------------------------
Language                      files          blank        comment           code
--------------------------------------------------------------------------------
JavaScript                        4             30             34            156
Bourne Again Shell                1              7              1             37
JSON                              1              0              0              7
--------------------------------------------------------------------------------
SUM:                              6             37             35            200
--------------------------------------------------------------------------------
```

That's pretty cool.

The CLI `firebase` lets you setup a project, run the emulators, and push to prod. 

Maybe writing nodejs REST APIs isn't the what most Firebase users do, but it certainly works for me.

My use of Firebase gives me a great data point in my path to understanding great developer experience--what's good, what's bad, where innovation is required. That said, the main focus of Firebase is not building REST APIs, AFAIK, it's more of a "backend as a service" where you don't even have to write the API (but obviously I didn't use that part of Firebase).

Now to explore other platforms... :)
