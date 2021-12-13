---
layout: post
title:  "Dev Experience: My First Look at Azure Functions"
categories:
header_image: "/img/azure-functions-post.jpg"
summary: "How long will it take me to get a function deployed?"

---

# {{ page.title }}

## What Do I Want to Do?

I've not used Azure Functions before, so I'm going to run through a quick start to deploy an example nodejs function.

What I have/want to do:

* Already have `az` installed and an Azure account to work with
* Running from a Linux workstation
* Don't want to use VSCode integration currently, prefer to use CLI for now
* Deploy a NodeJS 16 "hello world" function manually

## Installing

* [Docs](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=v4%2Clinux%2Ccsharp%2Cportal%2Cbash%2Ckeda#v2)

Get the core tools, which presumably includes the func CLI.

```
$ sudo apt-get install azure-functions-core-tools-4
Reading package lists... Done
SNIP!
Preparing to unpack .../azure-functions-core-tools-4_4.0.3971-1_amd64.deb ...
Unpacking azure-functions-core-tools-4 (4.0.3971-1) ...
Setting up azure-functions-core-tools-4 (4.0.3971-1) ...

Telemetry
---------
The Azure Functions Core tools collect usage data in order to help us improve your experience.
The data is anonymous and doesn't include any user specific or personal information. The data is collected by Microsoft.

You can opt-out of telemetry by setting the FUNCTIONS_CORE_TOOLS_TELEMETRY_OPTOUT environment variable to '1' or 'true' using your favorite shell.
```

Now I have the `func` command.

```
$ which func
/usr/bin/func
```

Init a new project. I'm going to use nodejs.

```
$ func init .
Select a number for worker runtime:
1. dotnet
2. dotnet (isolated process)
3. node
4. python
5. powershell
6. custom
Choose option: 3
node
Select a number for language:
1. javascript
2. typescript
Choose option: 1
javascript
Writing package.json
Writing .gitignore
Writing host.json
Writing local.settings.json
Writing /home/curtis/working/sparrow-dns-azure-function/.vscode/extensions.json
g /home/curtis/working/sparrow-dns-azure-function/MyFunctionProj/.vscode/extensions.json
```

That creates a few files.

```
$ tree
.
├── host.json
├── local.settings.json
└── package.json

0 directories, 3 files
```

Next, let's create a function.

## Creating a Function

* [Docs](https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-cli-node?tabs=azure-cli%2Cbrowser)

Create a function from a template "HTTP Trigger".

```
$ func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
Select a number for template:HTTP trigger
Function name: [HttpTrigger] Writing /home/curtis/working/sparrow-dns-azure-function/HttpExample/index.js
Writing /home/curtis/working/sparrow-dns-azure-function/HttpExample/function.json
The function "HttpExample" was created successfully from the "HTTP trigger" template.
```

Interesting that there is a template option.

### Test Locally

Now in one terminal:

```
$ func start
```

Example output:

```
$ func start

Azure Functions Core Tools
Core Tools Version:       4.0.3971 Commit hash: d0775d487c93ebd49e9c1166d5c3c01f3c76eaaf  (64-bit)
Function Runtime Version: 4.0.1.16815


Functions:

	HttpExample: [GET,POST] http://localhost:7071/api/HttpExample

For detailed output, run func with --verbose flag.
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 POST http://127.0.0.1:40553/AzureFunctionsRpcMessages.FunctionRpc/EventStream application/grpc -
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /AzureFunctionsRpcMessages.FunctionRpc/EventStream'
[2021-12-13T11:29:47.400Z] Worker process started and initialized.
[2021-12-13T11:29:52.128Z] Host lock lease acquired by instance ID '000000000000000000000000AC5DB4CC'.
```

And in another terminal, curl..

```
$ curl http://localhost:7071/api/HttpExample
This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response
```

OK, great, but now how to publish to "the cloud"...

### Configure Azure to be able to Deploy the function to azure...

First we need to configure a resource group, etc.

Login.

```
$ az login
```

Create a resource group.

```
export RG=sparrow-dns-functions-rg
export REGION=canadacentral
export APPNAME=<project name>
export STORAGE=sparrowdnsfuncstorage
az group create --name $RG --location $REGION
```

Storage account.

>NOTE: What's the deal with the storage names, yeesh. Lower case letters or numbers only. Don't like it.

>Question: What does a function need a storage account for?

```
az storage account create --name $STORAGE --location $REGION --resource-group $RG --sku Standard_LRS
```

Create the function app.

>NOTE: Using node 16, not 14.

>"...replace <STORAGE_NAME> with the name of the account you used in the previous step, and replace <APP_NAME> with a globally unique name appropriate to you. The <APP_NAME> is also the default DNS domain for the function app."

```
az functionapp create --resource-group $RG \
--consumption-plan-location $REGION \
--runtime node --runtime-version 16 --functions-version 4 \
--name $APPNAME \
--storage-account $STORAGE
```

oh no error.

```
$ az functionapp create --resource-group $RG \
> --consumption-plan-location $REGION \
> --runtime node --runtime-version 16 --functions-version 4 \
> --name $APPNAME \
> --storage-account $STORAGE
az functionapp create: '4' is not a valid value for '--functions-version'. Allowed values: 2, 3.

TRY THIS:
az functionapp create --resource-group MyResourceGroup --plan MyPlan --name MyUniqueAppName --storage-account MyStorageAccount
Create a basic function app.

https://docs.microsoft.com/en-US/cli/azure/functionapp#az_functionapp_create
Read more about the command in reference docs
```

Must need newer az CLI?

```
$ sudo apt-get --only-upgrade install azure-cli
$ az version
{
  "azure-cli": "2.31.0",
  "azure-cli-core": "2.31.0",
  "azure-cli-telemetry": "1.0.6",
  "extensions": {}
}
```

Now run again...

```
$ az functionapp create --resource-group $RG\
 --consumption-plan-location $REGION \
 --runtime node --runtime-version 16 --functions-version 4 \
 --name $APPNAME \
 --storage-account $STORAGE
Resource provider 'Microsoft.Web' used by this operation is not registered. We are registering for you.
Registration succeeded.
SNIP!
```

That gives you a link to this page to see "Application Insights" which it seems will be deprecated.

![application insights](/img/azure-func-app-insight.jpg)

Onward!

## Deploy the Function

Now we actually push the function to the function app.

```
export APPNAME=<project name>
func azure functionapp publish $APPNAME
```

Now can access.

```
$ func azure functionapp publish $APPNAME
Getting site publishing info...
Creating archive for current directory...
Uploading 1.3 KB [#####################################################################]
Upload completed successfully.
Deployment completed successfully.
Syncing triggers...
Functions in sparrow-dns:
    HttpExample - [httpTrigger]
        Invoke url: https://<project name>.azurewebsites.net/api/httpexample

```

Connect with httpie.

```
$ http https://<project name>.azurewebsites.net/api/httpexample
HTTP/1.1 200 OK
Content-Encoding: gzip
Content-Type: text/plain; charset=utf-8
Date: Mon, 13 Dec 2021 12:13:02 GMT
Request-Context: appId=cid-v1:dd33b7ce-86e0-4822-a59b-9dd8b4116385
Transfer-Encoding: chunked
Vary: Accept-Encoding

This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.
```

Provide a name.

```
$ http post https://<project name>.azurewebsites.net/api/httpexample name=curtis
HTTP/1.1 200 OK
Content-Encoding: gzip
Content-Type: text/plain; charset=utf-8
Date: Mon, 13 Dec 2021 12:14:01 GMT
Request-Context: appId=cid-v1:dd33b7ce-86e0-4822-a59b-9dd8b4116385
Transfer-Encoding: chunked
Vary: Accept-Encoding

Hello, curtis. This HTTP triggered function executed successfully.
```

That's it.

## Conclusion

Hereare some basic thoughts:

* I find Azure naming unusual
* I like that there is a "template" option for functions, should explore what that means (Can I create my own templates? Probably not. That's something that the [Tanzu Application Acclerator](https://docs.vmware.com/en/Application-Accelerator-for-VMware-Tanzu/index.html) can do, template any application including Azure Functions)
* Keep multiple functions in the same repo
* Not sure why the `az` and `func` CLIs exist, can't deploy a function with `az`?
* I like the domain: your-project.azurewebsites.net/api/somefunction
* Having links point me to services that are being deprecated is a bit concering, but Azure is a massive ecosystem so not unexpected, definitely good that there are application metrics/monitoring integrated of course
* The localhost name of the function is "HttpExample" and the deployed version is "httpexample"
* As is common with functions, I'm not sure what version of nodejs is being used everywhere....presumably 16 is being used in the cloud, as that is what I specified, but not sure locally what `func` does...

Obviously this is my first time using Azure Functions, but so far, other than setting up the resource group and such, I quite like it. Didn't take long to get a function deployed. Several languages are supported, including Java which I should experiment with.

I'll take Azure Functions a bit deeper in future posts. So far looks really great.

## Links

* [Azure Functions Developer Guide](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node?tabs=v2)

