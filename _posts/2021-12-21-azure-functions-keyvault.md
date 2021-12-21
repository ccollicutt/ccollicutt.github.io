---
layout: post
title:  "Azure Functions, Managed Idenity, NodeJS, and Key Vault"
categories:
header_image: "/img/azure-func-managed-identity.jpg"
summary: "Apps need secrets to get secrets...or do they?"

---

# {{ page.title }}

Azure has functions. Azure as a way to manage secrets called Key Vault. How do these work together? If you create a function and you want to access a Key Vault secret, clearly it has to authenticate to the Key Vault service...but how?

Managed identity is the answer. But what is "managed identity"?

>A managed identity from Azure Active Directory (Azure AD) allows your app to easily access other Azure AD-protected resources such as Azure Key Vault. The identity is managed by the Azure platform and does not require you to provision or rotate any secrets. - [Azure Docs](https://docs.microsoft.com/en-us/azure/app-service/overview-managed-identity?tabs=dotnet)

Basically your function can authenticate without having to login in the way that we typically think of "logging in", ie. with a username and password. If we did have to use a username and password then the function would have to get that information from somewhere, and if that information became public in some way we'd have to rotate (ie. change) those secrets, which is a huge pain. But with managed identity we don't have to do that, instead we configure Azure to allow our function to access Key Vault. Thus, the platform takes care of everything in the background, which is what platforms are supposed to do. :)

Also, and this is interesting, in my NodeJS code I'm using the below to setup the credential so that I can access Key Vault secrets.

```
const credential = new DefaultAzureCredential();
```

The above assumes there are AZURE_TENANT_ID, AZURE_CLIENT_ID and AZURE_CLIENT_SECRET variables configured, ie. when developing locally, having logged in with `az login` or setup those variables. However, once the function has been pushed to Azure, if those variables are not available, the code will try to use a managed identity. So I don't have to use one method locally and another in production.

In my case, initially managed identity access wasn't configured for the functionapp, so I received this error when running in Azure:

```
2021-12-21T11:59:26.309 [Error] Executed 'Functions.etc-hosts' (Failed, Id=56fdb72b-eb86-41b9-a215-d7e7b3f22425, Duration=98ms)Result: FailureException: Error: Azure CLI could not be found.  Please visit https://aka.ms/azure-cli for installation instructions and then, once installed, authenticate to your Azure account using 'az login'.Stack: Error: Azure CLI could not be found.  Please visit https://aka.ms/azure-cli for installation instructions and then, once installed, authenticate to your Azure account using 'az login'.at AzureCliCredential.getToken 
```

I needed to setup managed identity and allow it to access a Key Vault.

## Configure Managed Identity Access for Function

First, I setup some vars representing my function deployment. Of course these are filled out when I run it in my environment. They're empty here.

```
export RG=
export REGION=
export APPNAME=
export STORAGE=
export KV=
```

Next, assign and identity.

```
az functionapp identity assign --resource-group ${RG} --name ${APPNAME}
```

Eg. output:

```
$ az functionapp identity assign --resource-group ${RG} --name ${APPNAME}
{
  "principalId": "<redacted",
  "tenantId": "<redacted>",
  "type": "SystemAssigned",
  "userAssignedIdentities": null
}
```

Now we can just grab the principalId (or copy it from the above output).

```
export PRINCIPAL_ID=$(az functionapp identity show -n ${APPNAME} --query principalId --resource-group ${RG} -o tsv)
```

Finally setup a policy for key vault to allow this  principal to access the secrets.

```
az keyvault set-policy -n ${KV} \
  --object-id ${PRINCIPAL_ID} \
  --resource-group ${RG} \
  --secret-permissions get list 
```

At this point, even when using `DefaultAzureCredential()`, when pushed into Azure the system is smart enough to use the managed identity.