---
layout: post
title: Spring Cloud Gateway - Animal Rescue
categories:
header_image: "/img/animal-rescue1.jpg"
header_permalink: "https://unsplash.com/photos/zBvVuRJ71vU"
summary: "Using Spring Cloud Gateway on Tanzu Application Service"

---

# {{ page.title }}

[Animal Rescue](https://github.com/spring-cloud-services-samples/animal-rescue) is a sample application used to demonstrate Spring Cloud Gateway, and Single Sign On, in the [Tanzu Application Service](https://tanzu.vmware.com/application-service) (TAS), which is based on Cloud Foundry. This particular demo has an an automated script that will deploy the microservice based Animal Rescue into TAS, and in doing so will setup a Spring Cloud Gateway (SCG) instance via a service in TAS. This makes it extremely easy to use and is completely available by self-service for developers.

## What's an API gateway?

>An API gateway takes all API calls from clients, then routes them to the appropriate microservice with request routing, composition, and protocol translation. Typically it handles a request by invoking multiple microservices and aggregating the results, to determine the best path. It can translate between web protocols and web‑unfriendly protocols that are used internally. - [Nginx](https://www.nginx.com/learn/api-gateway/s)

An API gateway organizes client requests to various microservices. It can manipulate those requests, aggregate multiple microservices together, translate protocols, etc, etc. They can consolidate authorization and other cross cutting concerns. Gateways can also help reduce the number of calls clients need to make, i.e. they don't need to understand every backend service, just enough to talk to the gateway. Another useful thing about gateways is that they can add in security. They can kind of do anything, and frankly that is what often makes them confusing, especially to people like myself who don't write code 100% of the time.

## Spring Cloud Gateway

If you write apps in Java there is a very high likelihood that you also are using the Spring Framework and Spring has it's own gateway, thoughtfully called  [Spring Cloud Gateway](https://tanzu.vmware.com/content/blog/microservices-essentials-getting-started-with-spring-cloud-gateway).

Spring Cloud (API?) Gateway: 

>This project provides a library for building an API Gateway on top of Spring MVC. Spring Cloud Gateway aims to provide a simple, yet effective way to route to APIs and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency. - [Spring Cloud docs](https://spring.io/projects/spring-cloud-gateway)

A few key features of SCG:

* Built on Spring Framework 5, Project Reactor and Spring Boot 2.0
* Able to match routes on any request attribute
* Predicates and filters are specific to routes
* Hystrix Circuit Breaker integration
* Spring Cloud DiscoveryClient integration
* Easy to write Predicates and Filters
* Request Rate Limiting
* Path Rewriting

## Install Spring Gateway Tile into Tanzu Application Service

In order to deploy Animal Rescue with Spring Cloud Gateway, we need the Spring Cloud Gateway Tile deployed in Tanzu Application Service (TAS). This is not the only way to use Spring Cloud Gateway, but it's how Animal Rescue is expecting to access it, at least in terms of the bash script that comes with the repo that can initialize and deploy the app. So in this example, SCG will be a service managed by the platform (TAS) that can be bound to an application.

The Single Sign On Tile is also required and in my case I enabled it for all orgs.

![img](/img/animal-rescue4.jpg)

There are a few tiles installed in this TAS instance, but the important ones for Animal Rescue are the SSO and SCG tiles.

## Deploy Animal Rescue to Tanzu Application Service


Clone the repo:

```
git clone https://github.com/spring-cloud-services-samples/animal-rescue
cd animal-rescue
```

Login to TAS:

```
cf login -a api.sys.yourdomain.com
```

I needed make, g++, jq installed, there may be other requirements my Linux box already had installed.

```
sudo apt install make g++ jq -y
```

Run the init script.

```
$ ./scripts/cf_deploy.sh init
npm WARN prepare removing existing node_modules/ before installation

> core-js@2.6.11 postinstall /home/curtis/working/animal-rescue/frontend/node_modules/babel-runtime/node_modules/core-js
> node -e "try{require('./postinstall')}catch(e){}"

Thank you for using core-js ( https://github.com/zloirock/core-js ) for polyfilling JavaScript standard library!

The project needs your help! Please consider supporting of core-js on Open Collective or Patreon: 
> https://opencollective.com/core-js 
> https://www.patreon.com/zloirock 
SNIP!
BUILD SUCCESSFUL in 3s
4 actionable tasks: 4 executed
```

Then deploy to TAS:

```
$ ./scripts/cf_deploy.sh deploy
Service instance gateway-demo not found
Gateway service does not exist, creating...
Creating service instance gateway-demo in org java / space spring-cloud-gateway as admin...
OK

Create in progress. Use 'cf services' or 'cf service gateway-demo' to check operation status.
Waiting for service instance to be ready...
SNIP!
< X-Content-Type-Options: nosniff
< X-Frame-Options: DENY
< X-Vcap-Request-Id: 8c89f955-543f-4b3a-7e51-a9b42b1fbe31
< X-Xss-Protection: 1 ; mode=block
< Date: Thu, 07 May 2020 15:37:11 GMT
< 
* Connection #0 to host gateway-demo.apps.sf.vsphere.local left intact

=====
Bound app animal-rescue-backend route configuration update response status: 204
```

Now I've got the frontend and backend apps running on internal urls.

```
$ cf apps
Getting apps in org java / space spring-cloud-gateway as admin...
OK

name                     requested state   instances   memory   disk   urls
animal-rescue-backend    started           1/1         1G       1G     animal-rescue-backend.apps.internal
animal-rescue-frontend   started           1/1         1G       1G     animal-rescue-frontend.apps.internal
```

I can access that via the browser, login, and adopt animals.

If I use the gateway org I can see the instance of the gateway service:

```
$ cf apps
Getting apps in org p-spring-cloud-gateway-service / space fb081bbf-fa60-4a40-bbb0-4f4c47ab2c0a as admin...
OK

name      requested state   instances   memory   disk   urls
gateway   started           1/1         1G       1G     gateway-demo.apps.sf.vsphere.local, gateway-fb081bbf-fa60-4a40-bbb0-4f4c47ab2c0a.apps.internal
```

And then that can be accessed in the browser. The login is via SSO.

Look at all those cute animals up for microservice adoption.

![img](/img/animal-rescue.jpg)

## Spring Cloud Gateway Configuration and Service

In the apps GUI, we can see the SCG instance.

![scg](/img/animal-rescue2.jpg)

And the corresponding routes.

![routes](/img/animal-rescue3.jpg)

Note that the route is internal.

It's worthwhile to have a look at the [routes](https://docs.pivotal.io/spring-cloud-gateway/1-0/configuring-routes.html) that are configured in the gateway, frontend, and backend gateway-config.json files.

## Conclusion

API gateways can be really helpful with microservices. There are several solutions out there, but one of the most flexible is Spring Cloud Gateway. It's absolutely worth looking at and is a serious competitor and will be improving, and innovating, rapidly. In this particular example, SCG was deployed and managed as a service in TAS, which makes it extremely easy for developers to use, but it can also be managed in other manners.

This post only barely scratches the surface of what Spring Cloud Gateway can do, and in combination with TAS. There's tons of [docs](https://spring.io/guides/gs/gateway/) out there on getting started with SCG.