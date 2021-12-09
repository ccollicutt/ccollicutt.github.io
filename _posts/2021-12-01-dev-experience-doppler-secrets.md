---
layout: post
title:  "Dev Experience: Managing Secrets with Doppler"
categories:
header_image: "/img/doppler.jpg"
summary: "Starting my walkabout amongst the dev platforms"

---

# {{ page.title }}

Secrets. I need to manage them. As part of my exploration of developer experience I plan on having many micro-services running on different platforms. However, I don't want to have to manage the secrets across all of them individually...that would be a nightmare, never mind thinking about various environments (dev, test, prod...). For example, I'm using Basic Authentication as a simple API key and API secret key, and I need to manage those secrets across all services, my CLI, and of course, various environments (which should have different secrets).

So I need something to help me do that.

One option is [Doppler](https://www.doppler.com/). Doppler bills itself as a "universal secrets manager" and I think, after a bit of use, that's a pretty accurate description.

## About Doppler

What's Doppler?

>Doppler [has] launched the industry’s first Universal Secrets Manager, a modern secrets manager offering built to win the hearts and minds of developers. It works across every language, stack and infrastructure, increasing developer productivity while strengthening a company’s overall security. Early adopters, including Stripe, Point Banking, Snackpass, Kopa and Convictional, use Doppler to securely store secrets such as API keys, credentials, ENV variables and database URLs. - [Press Release](https://blog.doppler.com/press-release)

Secrets and credentials are a major problem. Organizations have more environments in use than they are often willing to admit (or even track).

>Secrets and credentials management is widely considered to be the most overlooked aspect of software development. Many teams struggle daily to organize and sync secrets between environments, with manually maintained .env files being one of the most common sources of frustration for developers and DevSecOps. - [Doppler](https://blog.doppler.com/what-is-a-secrets-manager)

Focus on developer experience.

>Security tools are often process heavy and come with horrible experiences which leads to low usage. At Doppler we strongly believe in building tools that developers will love. The more you love it, the more you will want to use it. - [Doppler](https://www.doppler.com/about)

What a great attitude.

## tl;dr

I really like Doppler. I like pretty much everything about it. I like how it's setup, I like how it guides you through using it. I like that it has a CLI that I can integrate with everything. I like that it had examples for Firebase. Looks like there are integrations for Netlify and Vercel (two services I plan on checking out). I also like that it will copy secrets to each environment, and let you know when environments have secrets that don't exist in other environments.

It's just a really well thought out secrets as a service, and I've only--just barely--scratched the surface of using it.

## Installing doppler cli

I'm running Linux as my main OS for writing software.

To install `doppler`, get the package.

```
# Install pre-reqs
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

# Add Doppler's GPG key
curl -sLf --retry 3 --tlsv1.2 --proto "=https" 'https://packages.doppler.com/public/cli/gpg.DE2A7741A397C129.key' | sudo apt-key add -

# Add Doppler's apt repo
echo "deb https://packages.doppler.com/public/cli/deb/debian any-version main" | sudo tee /etc/apt/sources.list.d/doppler-cli.list

# Fetch and install latest doppler cli
sudo apt-get update && sudo apt-get install doppler
```

Now we've got `doppler`.

```
$ which doppler
/usr/bin/doppler
```

Login.

## Initial Configuration

Login.

```
$ doppler login
? Open the authorization page in your browser? No
Complete authorization at https://dashboard.doppler.com/workplace/auth/cli
Your auth code is:
<SNIP!>

Waiting...

Welcome, Curtis
```

I've already setup a project called sparrow-dns.

```
$ doppler setup
? Select a project: sparrow-dns
? Select a config: dev
┌─────────┬─────────────┬──────────────────────────────────┐
│ NAME    │ VALUE       │ SCOPE                            │
├─────────┼─────────────┼──────────────────────────────────┤
│ config  │ dev         │ /home/curtis/working/sparrow-dns │
│ project │ sparrow-dns │ /home/curtis/working/sparrow-dns │
└─────────┴─────────────┴──────────────────────────────────┘
```

By default there are three environments configured, of course you could have more or fewer.

```
$ doppler environments
┌─────┬─────────────┬───────────────┬──────────────────────────┬─────────────┐
│ ID  │ NAME        │ INITIAL FETCH │ CREATED AT               │ PROJECT     │
├─────┼─────────────┼───────────────┼──────────────────────────┼─────────────┤
│ dev │ Development │               │ 2021-12-04T12:45:27.568Z │ sparrow-dns │
│ stg │ Staging     │               │ 2021-12-04T12:45:27.568Z │ sparrow-dns │
│ prd │ Production  │               │ 2021-12-04T12:45:27.568Z │ sparrow-dns │
└─────┴─────────────┴───────────────┴──────────────────────────┴─────────────┘
```

## Using Doppler

To use it with Firebase I've added the following to the scripts section of my package.json.

```
    "serve": "doppler run -- firebase emulators:start",
    "shell": "doppler run -- firebase functions:shell",
    "update_config": "firebase functions:config:unset env && firebase functions:config:set env=\"$(doppler secrets download --config prd --no-file --silent)\"",
    "deploy": "firebase functions:config:unset env && firebase functions:config:set env=\"$(doppler secrets download --config prd --no-file --silent)\" && firebase deploy --only functions",
```

I've also got a CLI for using the DNS arecords demo API (I'm currently calling it Sparrow DNS) and I can easily integrate with it, and set the endpoint URL to the local URL that is setup when I use the Firebase emulator. This helps a lot with local testing.

For example, if I want to use the CLI with the dev secrets...

```
doppler run -- ./scripts/sparrow-cli arecord list
```

But if I want to use the production environment...

```
doppler run --config prd -- ./scripts/sparrow-cli arecord list
```

I'll need to think about how to integrated the `doppler` CLI into the Sparrow CLI.

## Conclusion

Again, so far I'm a big fan of Doppler, 1) because it works and 2) because it's obvious that they are not only talking about creating a great developer experience, they are actually doing it!

I would imagine I'll write another post once I've worked more with Doppler and various "serverless" platforms, as well as tooling like [Spring Cloud Config](https://cloud.spring.io/spring-cloud-config/reference/html/).