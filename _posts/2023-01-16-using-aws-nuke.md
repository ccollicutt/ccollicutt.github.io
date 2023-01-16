---
layout: post
title:  "Using AWS Nuke"
categories:
header_image: "/img/aws-nuke.jpg"
summary: "How to remove everything from your AWS account"

---

# {{ page.title }}

I recently set up a second AWS account just to use for testing. I have a primary account, but I want one that I can easily wipe out absolutely everything in, specifically using AWS Nuke.

# What is AWS Nuke?

[AWS Nuke](https://github.com/rebuy-de/aws-nuke) is a CLI applicaiton that can wipe out everything in an AWS account, if you want it to. 

What does it do? It removes everything from your AWS account. And I quote:

> Remove all resources from an AWS account.

Big red warning light:

>NOTE: Of course, using AWS Nuke can be extremely...dangerous. You could wipe out everything in your account. AWS Nuke tries to be as safe as possible, but the point is to use it to delete everything. 

## Installation

I downloaded the [latest release](https://github.com/rebuy-de/aws-nuke/releases), untarred it and insatlled the binary in my local bin.

```
$ which aws-nuke
/home/curtis/bin/aws-nuke
```

Help:

```
$ aws-nuke -h
A tool which removes every resource from an AWS account.  Use it with caution, since it cannot distinguish between production and non-production.

Usage:
  aws-nuke [flags]
  aws-nuke [command]

Available Commands:
  completion     Generate the autocompletion script for the specified shell
  help           Help about any command
  resource-types lists all available resource types
  version        shows version of this application
SNIP!!
```

## Usage

The most important thing is the config file and below is a configuration file I've used.

Notes on the configuration file example:

* I'm just looking at us-east-1 for now (and global, i.e. IAM)
* I don't want AWS Nuke to remove the "curtis" user, or their key
* Also filter out the MFA configuration for that user (though I don't believe Nuke can remove it)

```
regions:
- us-east-1
- global

account-blocklist:
- "REDACTEDID_ACCOUNT_NOT_TO_NUKE" # personal i.e. prod account

accounts:
  # awstesting account to run nuke in
  "REDACTEDID_ACCOUNT_TO_NUKE":
    filters:
      IAMUser:
      - "curtis"
      IAMUserPolicyAttachment:
      - "curtis -> AdministratorAccess"
      IAMUserAccessKey:
      - "curtis -> REDACTEDKEY1"
      IAMVirtualMFADevice:
      - "arn:aws:iam::REDACTEDID2:mfa/googleauth"
```

Otherwise, we're going to remove everything from that account that we can, except the "curtis" user.

## Alias Accounts

AWS Nuke wants you to alias accounts. It's going to try to save you from deleting production by looking for the letters "prod" in the account alias.

>"To avoid just displaying a account ID, which might gladly be ignored by humans, it is required to actually set an Account Alias for your account. Otherwise aws-nuke will abort." - AWS Nuke documentation

* For my testing account, I gave it this alias.

```
aws iam create-account-alias --profile awstesting --account-alias awstesting
```

* To validate...


```
$ aws iam list-account-aliases --profile awstesting
{
    "AccountAliases": [
        "awstesting"
    ]
}
```

Now I can use AWS Nuke.

## Example Use

Let's create a user in the AWS Account I want to run Nuke in, i.e. I want this new account to be *removed* by AWS Nuke.

First, validate I'm using my testing account.

```
$ export AWS_PROFILE=awstesting
$ aws sts get-caller-identity
{
    "UserId": "REDACTED",
    "Account": "REDACTED",
    "Arn": "arn:aws:iam::REDACTED:user/curtis"
}
```

* Next, create a user that will be removed by AWS nuke

```
aws iam create-user --user-name nukeme
```

* Now list users in the account, there should be only two...

```
aws iam list-users
```

E.g. output:

```
$ aws iam list-users
{
    "Users": [
        {
            "Path": "/",
            "UserName": "curtis",
            "UserId": "REDACTED",
            "Arn": "arn:aws:iam::REDACTED:user/curtis",
            "CreateDate": "2023-01-13T15:14:00Z"
        },
        {
            "Path": "/",
            "UserName": "nukeme",
            "UserId": "REDACTED",
            "Arn": "arn:aws:iam::REDACTED:user/nukeme",
            "CreateDate": "2023-01-16T16:28:15Z"
        }
    ]
}
```

* Run AWS Nuke in **dry-run** mode

>NOTE: Without a specific option, which I won't show here, AWS Nuke will always run in **dry-run** mode.

```
aws-nuke -c aws-nuke.yaml --profile awstesting
```

It will ask you to type in the alias of the account.

If you want to actually **for real** delete everything, you will need to add the no dry run option, and you'll be asked to type in the account profile name twice.


```
$ aws-nuke -c aws-nuke.yaml --profile awstesting -q
aws-nuke version v2.21.2 - Fri Dec  9 20:36:12 UTC 2022 - e76d21c263477ebd6648fae19f9e539049ad2b51

Do you really want to nuke the account with the ID REDACTED and the alias 'awstesting'?
Do you want to continue? Enter account alias to continue.
> awstesting
SNIP!!
2023/01/16 13:19:18 This operation, ListFleets, has been deprecated
global - IAMUser - nukeme - [Name: "nukeme"] - would remove
Scan complete: 67 total, 1 nukeable, 66 filtered.

The above resources would be deleted with the supplied configuration. Provide --no-dry-run to actually destroy resources.
```

* Run it again, but with the option to *really delete everything*

```
$ aws-nuke -c aws-nuke.yaml --profile awstesting -q --no-dry-run
aws-nuke version v2.21.2 - Fri Dec  9 20:36:12 UTC 2022 - e76d21c263477ebd6648fae19f9e539049ad2b51

Do you really want to nuke the account with the ID REDACTED_ACCOUNT_TO_NUKE and the alias 'awstesting'?
Do you want to continue? Enter account alias to continue.
> awstesting
SNIP!!
Do you really want to nuke these resources on the account with the ID REDACTED_ACCOUNT_TO_NUKE and the alias 'awstesting'?
Do you want to continue? Enter account alias to continue.
> awstesting

global - IAMUser - nukeme - [Name: "nukeme"] - triggered remove

Removal requested: 1 waiting, 0 failed, 66 skipped, 0 finished

global - IAMUser - nukeme - [Name: "nukeme"] - waiting

Removal requested: 1 waiting, 0 failed, 66 skipped, 0 finished

global - IAMUser - nukeme - [Name: "nukeme"] - removed

Removal requested: 0 waiting, 0 failed, 66 skipped, 1 finished

Nuke complete: 0 failed, 66 skipped, 1 finished.
```

Goodbye new account!

## Conclusion

So far I like AWS Nuke. I had tested out some Fargate workloads and removed them, but they had left a NAT Gateway running, which AWS Nuke found. As we all know, those NAT gateways cost a lot of money. I'm really thankful tools like this exist.