---
layout: post
title:  Use a Github Personal Access Token with the Concourse CI/CD System
categories:
header_image: "/img/concourse-github.jpg"
header_permalink: "https://unsplash.com/photos/Wwgb1aGCXGk"
summary: "An easy solution to Github auth with Concourse"

---

# {{ page.title }}

It's actually pretty easy, but I don't see many examples of setting it up online.

First, you need a Gitub Personal Access Token configured.

Next, here's the github resource. I'm using a forked [Spring Petclinic](https://github.com/spring-projects/spring-petclinic) as an example repository, which comes from a variable, as well as the access token. Note the use of `password: x-oauth-basic`.

```
resources:

  - name: spring-petclinic
    type: git
    source:
      uri: ((spring-petclinic-repo-uri))
      icon: github
      branch: main
      username: ((github-apptoken))
      password: x-oauth-basic
```

This is the credentials file:

```
spring-petclinic-repo-uri: https://github.com/ccollicutt/spring-petclinic.git

github-apptoken: YOUR_GITHUB_TOKEN
```

That's it!