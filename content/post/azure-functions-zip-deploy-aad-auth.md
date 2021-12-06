---
layout: post
title: "Azure Functions Zip Deploy with Azure AD Authentication"
date: 2021-12-05T16:00:00-05:00
categories: [azure-functions]
author: "Michael S. Collier"
tags: [azure-functions, azure-ad, azure-active-directory]
comments: true
---

I was recently faced with a scenario where I needed a script to deploy an Azure Function.  Specifically, there was a desire to use a REST API to deploy a .zip file of the Function app.

The documentation for [“Deploy ZIP file with REST APIs”](https://docs.microsoft.com/azure/azure-functions/deployment-zip-push#rest) indicates this is possible via a HTTP POST request to <https://{APP-NAME}.scm.azurewebsites.net/api/zipdeploy>.  One challenge with the current documentation is the stated need to use HTTP BASIC authentication.  Needing to use BASIC authentication is a problem if BASIC authentication is disabled.

As stated in the [documentation](https://docs.microsoft.com/azure/app-service/deploy-configure-credentials?tabs=cli#disable-access-to-the-api), the SCM REST API is backed by Azure RBAC.  I couldn’t find any reference on how to use the SCM REST API to perform a zip deployment. Does the _/zipdeploy_ endpoint work with AAD-based authentication?

I reached out to a colleague who helped figure out that, yes, it is possible to use the _/zipdeploy_ SCM REST API with AAD-based authentication!

Let’s go through the steps.

> Jump to my [GitHub repo](https://github.com/mcollier/azure-functions-zip-deploy-aad) get started quickly!

## Create an Azure Function

I’m going to use [Azure Bicep](https://docs.microsoft.com/azure/azure-resource-manager/bicep/) to create an Azure Function (Consumption) app, the required Azure Storage dependency, and Application Insights.  You can find the full Bicep files in my [GitHub repo](https://github.com/mcollier/azure-functions-zip-deploy-aad/blob/main/iac/main.bicep).  The only “special” thing about my setup is that I’ve [disabled basic authentication for the SCM site, and disabled FTP publishing](https://docs.microsoft.com/azure/app-service/deploy-configure-credentials?tabs=cli#disable-basic-authentication).

```yml
resource azureFunction 'Microsoft.Web/sites@2020-12-01' = {
  name: azureFunctionAppName
  location: location
  kind: 'functionapp'
  properties: {
    // ommitted for brevity
  }

  resource config 'config' = {
    name: 'web'
    properties: {
      ftpsState: 'Disabled'
      minTlsVersion: '1.2'
    }
  }

  resource publishingScmCredentialPolicies 'basicPublishingCredentialsPolicies' = {
    name: 'scm'
    location: location
    properties: {
      allow: false
    }
  }

  resource publishingFtpCredentialPolicies 'basicPublishingCredentialsPolicies' = {
    name: 'ftp'
    location: location
    properties: {
      allow: false
    }
  }
}

```

I can use the [Azure CLI](https://docs.microsoft.com/cli/azure/) to deploy the Bicep template.

```bash
RESOURCE_GROUP_NAME="rg-azure-function-zip-deploy"

az deployment sub create \
    --template-file ../iac/main.bicep \
    --parameter resourceGroupName="$RESOURCE_GROUP_NAME" \
    --location eastus \
    --confirm-with-what-if
```

## Create a service principal (optional)

I’m going to need a valid Azure account to authenticate against the SCM REST API.  I could use my account, which is what I’ve logged into via the Azure CLI. However, to demonstrate that the deployment works with a restricted set of rights and scope, I’m going to [create a service principal](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cl) and give that service principal [“Website Contributor”](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#website-contributor) rights to only the newly created Azure Function app.

```bash
SCOPE='/subscriptions/[SUBSCRIPTION-ID]/resourceGroups/rg-azure-function-zip-deploy/providers/Microsoft.Web/sites/[FUNCTION-APP-NAME]'
SERVICE_PRINCIPAL_NAME='azfuncdeploy-sp'

az ad sp create-for-rbac \
    --name "$SERVICE_PRINCIPAL_NAME" \
    --role "Website Contributor" \
    --scope "$SCOPE"
```

The `az ad sp create-for-rbac` command will display output similar to the snippet below.  I need to sure to save the ‘appId’, ‘password’, and ‘tenant’ returned from the command.  I’ll need that data to get an access token used to authenticate against the SCM REST API.

```text
{
  "appId": "123456aa-aaaa-bbbb-cccc-0123456789123",
  "displayName": "azfuncdeploy-sp",
  "name": "123456aa-aaaa-bbbb-cccc-0123456789123",
  "password": "xxyyxxyyxxyyxyxyxyxyxx",
  "tenant": "123456789-1234-abcd-vxyz-012345678901"
}
```

## Deploy the code

At this point, I have a very basic Azure Function deployment and service principal.

![Azure Function deployment](../../images/azure-functions-zip-deploy-aad-auth/deployed-resources.png)

Now I need to deploy the code using the _/zipdeploy_ API.  To do so, I need to create a zip file for my function app.

```bash
# Publish the .NET Azure Function.
dotnet publish --configuration Release "../src/MyFunctionApp"

# Change to the directory where the .NET Azure Function is published.
cd ../src/MyFunctionApp/bin/Release/netcoreapp3.1/publish/ 

# Create a zip archive of the published function.
zip -r ../../../../code.zip .
```

I now need to get the access token that will be used to authenticate against the SCM REST API.  I can use the `az account get-access-token` command to get the token.  However, since I created a service principal, I need to login to that account first.  I’ll then retrieve the token and save it in a variable for later use.

```bash
az login --service-principal \
    --username "$SERVICE_PRINCIPAL_NAME" \
    --password "$SERVICE_PRINCIPAL_PASSWORD" \
    --tenant "$SERVICE_PRINCIPAL_TENANT_ID"

TOKEN=$(az account get-access-token -o tsv --query accessToken)
```

I now a valid AAD authentication token, and a zip file containing my function app.  The next step is to execute an HTTP POST request against the _/zipdeploy_ endpoint of the SCM REST API.  I’ll put the AAD token in the Authorization header.

```bash
curl -X POST \
    --data-binary @../src/MyFunctionApp/code.zip \
    -H "Authorization: Bearer $TOKEN" \
    "https://${APP_NAME}.scm.azurewebsites.net/api/zipdeploy"
```

You can find the full code deployment script in [my GitHub repo](https://github.com/mcollier/azure-functions-zip-deploy-aad/blob/main/scripts/deploy-code.sh).

That’s it!  It’ll take a few seconds for the deployment to complete.  I can go to the Azure Portal and see my newly deployed function.

![Azure Function with deployed function](../../images/azure-functions-zip-deploy-aad-auth/deployed-azure-function.png)

### Summary

It is possible to use an Azure AD identity to publish an Azure Function app via zip deployment.  You can also disable HTTP BASIC authentication for the Azure Function’s SCM site to be sure BASIC authentication isn't used.  To do so, you’ll need to do the following:

1. Deploy an Azure Function.
1. Disable BASIC authentication (optional).
1. Create a zip file for the function app.
1. Obtain the Azure AD authentication token for the identity to authenticate against the SCM REST API.
1. POST the zip file to the SCM API’s _/zipdeploy_ endpoint, providing the token as part of an Authorization header.

Special thanks to [Matthew Henderson](https://twitter.com/mattchenderson) for his assistance!!!
