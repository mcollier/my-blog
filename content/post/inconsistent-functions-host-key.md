---
title: "The case of the inconsistent Azure Functions host key"
author: "Michael S. Collier"
tags: [azure-functions, azure-resource-manager]
date: 2020-12-01T16:01:45-05:00
comments: true
---

I recently ran into a situation using the Azure Functions default host key where I did not understand the behavior I was observing.  Thanks to the help of some fantastic colleagues, we figured out what was going on.  I understand what is happening now.  I want to share here in hopes that my experience will help others that may run into a similar challenge.

## Scenario

I needed to create an Azure Function app via an ARM template.  The ARM template should create the Function app resource and set the necessary application settings.  One of those app settings should be a [Key vault reference](https://docs.microsoft.com/azure/app-service/app-service-key-vault-references) to a secret which contains the [default host key](https://docs.microsoft.com/azure/azure-functions/security-concepts#function-access-keys) for the function app.  My function's application code would retrieve that and invoke another function in the app.

_Also, please see Benjamin Perkin's ["Azure Function keys, keys and more keys, regenerated and synced"](https://www.thebestcsharpprogrammerintheworld.com/2020/11/23/azure-function-keys-keys-and-more-keys-regenerated-and-synced/) blog post for more observations on Azure Function key generation._

It all seems straight-forward enough.  I'm very comfortable with Azure Functions and feel that I've got a decent grasp on ARM templates. Famous last words.

## How it started

The snippet below shows my starting ARM template.  Can you spot the error?

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        // Omitted for brevity.
    },
    "variables": {
        // Omitted for brevity.
    },
    "resources": [
        {
            "type": "Microsoft.KeyVault/vaults",
            "apiVersion": "2019-09-01",
            "name": "[variables('keyVaultName')]",
            "location": "[parameters('location')]",
            "dependsOn":[
                "[resourceId('Microsoft.Web/sites',variables('functionAppName'))]"
            ],
            "properties": {
                // Omitted for brevity.
        },
        {
            "type": "Microsoft.KeyVault/vaults/secrets",
            "apiVersion": "2019-09-01",
            "name": "[concat(variables('keyVaultName'), '/', 'functionKey')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.KeyVault/vaults', variables('keyVaultName'))]",
                "[resourceId('Microsoft.Web/sites',variables('functionAppName'))]"
            ],
            "properties": {
                "value": "[listKeys(concat(resourceId('Microsoft.Web/sites', variables('functionAppName')), '/host/default'), '2016-08-01').functionKeys.default]"
            }
        },
        {
            "type": "Microsoft.Web/serverfarms",
            "apiVersion": "2020-06-01",
            "name": "[variables('appServicePlanName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Y1",
                "tier": "Dynamic",
                "size": "Y1",
                "family": "Y",
                "capacity": 0
            },
            "properties": {
               // Omitted for brevity.
            },
            "kind": "functionapp"
        },
        {
            "type": "Microsoft.Web/sites",
            "apiVersion": "2020-06-01",
            "name": "[variables('functionAppName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]"
            ],
            "kind": "functionapp",
            "identity": {
                "type": "SystemAssigned"
            },
            "properties": {
                "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]",
                "httpsOnly": true
            },
            "resources": []
        },
        {
            "type": "Microsoft.Web/sites/config",
            "apiVersion": "2020-06-01",
            "name": "[concat(variables('functionAppName'), '/appsettings')]",
            "dependsOn": [
                "[resourceId('Microsoft.Web/sites', variables('functionAppName'))]",
                "[resourceId('Microsoft.KeyVault/vaults/secrets', variables('keyVaultName'),'functionKey')]"
            ],
            "properties": {
                "AzureWebJobsStorage": "[concat('DefaultEndpointsProtocol=https;AccountName=',variables('storageAccountName'), ';AccountKey=',listkeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value, ';EndpointSuffix=', environment().suffixes.storage)]",
                "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING": "[concat('DefaultEndpointsProtocol=https;AccountName=',variables('storageAccountName'), ';AccountKey=',listkeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value, ';EndpointSuffix=', environment().suffixes.storage)]",
                "WEBSITE_CONTENTSHARE": "[variables('functionAppName')]",
                "FUNCTIONS_EXTENSION_VERSION": "~3",
                "FUNCTIONS_WORKER_RUNTIME": "dotnet",
                "APPINSIGHTS_INSTRUMENTATIONKEY": "[reference(resourceId('Microsoft.Insights/components', variables('appInsightsName')), '2018-05-01-preview').instrumentationKey]",
                "WEBSITE_RUN_FROM_PACKAGE": "1",
                "MyFunctionHostKey":"[concat('@Microsoft.KeyVault(SecretUri=', reference(resourceId('Microsoft.KeyVault/vaults/secrets', variables('keyVaultName'), 'functionKey')).secretUriWithVersion, ')')]"
            }
        }
    ]
}
```

_You can find the full problematic ARM template on my GitHub repo [here](https://github.com/mcollier/azure-functions-host-key/blob/incorrect-app-settings/azure-deploy.json)._

## The problem

After running the template, I noticed that the Key Vault secret value for the default host key was _not_ the same value as the Azure Function portal showed.

![keys do not match](../../images/inconsistent-functions-host-key/keys-not-matching.png)

I had two different host keys!  How is this possible?

## The analysis

The problem is where the [FUNCTIONS_EXTENSION_VERSION](https://docs.microsoft.com/azure/azure-functions/functions-app-settings#functions_extension_version) application setting was set.  By default, Azure Functions apps are set to use the v1 runtime.  The Azure Functions [v1 runtime persists its key in Azure Files](https://github.com/Azure/azure-functions-host/wiki/Changes-to-Key-Management-in-Functions-V2) (see also [this](https://docs.microsoft.com/azure/azure-functions/functions-app-settings#azurewebjobssecretstoragetype)).

When creating the function app via the ARM template, I did not explicitly set the FUNCTIONS_EXTENSION_VERSION app setting.  Thus, the runtime was set to the v1 runtime.  When retrieving the key in the ARM template (setting to a Key Vault secret), the Azure Functions runtime returned the key from Azure Files (_the v1 key_).

In my template, another ARM resource block sets the function's application settings, and it is there where the FUNCTIONS_EXTENSION_VERSION was set to ~3 (the v3 runtime).  By setting the application setting in a separate step, that forced a restart of the function.  Any change to the application settings triggers an application restart.  A consequence of this restart was the Azure Functions' runtime changing to v3.

The v3 runtime uses Azure Blob storage for persisting the keys.  Thus, when inspecting at the host key via the Azure portal after the template finished executing, the key returned is the key from Azure Blob storage (_the v3 key_).

To summarize, the Key Vault secret value was set, and _then_ the function app restarted.  Meaning, the default host key that was retrieved and set in the Key Vault secret was a v1 key, not a v3 key.  The process sequence was as follows:

1. Create a function app with the default runtime (~1).
1. Retrieve the function's default host key and set as a Key Vault secret.
1. Set the function app's application settings.
1. The function app restarts.
1. Function is running a v3 (~3) runtime.

## The solution

The "fix" is to set the FUNCTIONS_EXTENSION_VERSION at the **same time** as the function app is created.  Which, is exactly what [this documentation](https://docs.microsoft.com/azure/azure-functions/functions-infrastructure-as-code#create-a-function-app) indicates.  Figures.

If I set the FUNCTIONS_EXTENSION_VERSION to ~3 at app creation time, I get the expected default host key value in Key Vault, and in the portal.  They keys are the same!

It all seems so simple . . . until it is not.

The downside is that there is now a need to define the application settings _twice_!  The majority of the application settings should be set during function creation.  The site resource needs to be [created before the Key Vault resource in order to set the Key Vault's access policy allowing the site access to the Key Vault's secrets](https://docs.microsoft.com/azure/app-service/app-service-key-vault-references#azure-resource-manager-deployment).  Thus, I need to define, again, the app settings in a separate resource block, just like I had originally!  The template's `Microsoft.Web/Sites` resource looks as follows:

```json
{
            "type": "Microsoft.Web/sites",
            "apiVersion": "2020-06-01",
            "name": "[variables('functionAppName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Web/serverfarms',variables('appServicePlanName'))]",
                "[resourceId('Microsoft.Storage/storageAccounts',variables('storageAccountName'))]",
                "[resourceId('Microsoft.Insights/components', variables('appInsightsName'))]"
            ],
            "kind": "functionapp",
            "identity": {
                "type": "SystemAssigned"
            },
            "properties": {
                "name": "[variables('functionAppName')]",
                "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]",
                "httpsOnly": true,
                "siteConfig": {
                    "appSettings": [
                        {
                            "name": "AzureWebJobsStorage",
                            "value": "[concat('DefaultEndpointsProtocol=https;AccountName=',variables('storageAccountName'), ';AccountKey=',listkeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value, ';EndpointSuffix=', environment().suffixes.storage)]"
                        },
                        {
                            "name": "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING",
                            "value": "[concat('DefaultEndpointsProtocol=https;AccountName=',variables('storageAccountName'), ';AccountKey=',listkeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value, ';EndpointSuffix=', environment().suffixes.storage)]"
                        },
                        {
                            "name": "WEBSITE_CONTENTSHARE",
                            "value": "[variables('functionAppName')]"
                        },
                        {
                            "name": "FUNCTIONS_EXTENSION_VERSION",
                            "value": "~3"
                        },
                        {
                            "name": "FUNCTIONS_WORKER_RUNTIME",
                            "value": "dotnet"
                        },
                        {
                            "name": "APPINSIGHTS_INSTRUMENTATIONKEY",
                            "value": "[reference(resourceId('Microsoft.Insights/components', variables('appInsightsName')), '2018-05-01-preview').instrumentationKey]"
                        },
                        {
                            "name": "WEBSITE_RUN_FROM_PACKAGE",
                            "value": "1"
                        }
                    ]
                }
            },
            "resources": [
                {
                    "type": "config",
                    "name": "appsettings",
                    "apiVersion": "2020-06-01",
                    "dependsOn": [
                        "[resourceId('Microsoft.Web/sites', variables('functionAppName'))]",
                        "[resourceId('Microsoft.KeyVault/vaults/', variables('keyVaultName'))]",
                        "[resourceId('Microsoft.KeyVault/vaults/secrets', variables('keyVaultName'), 'functionKey')]"
                    ],
                    "properties": {
                        // Need to set the application settings seperate from the web site due to the Key Vault reference dependency.
                        "AzureWebJobsStorage": "[concat('DefaultEndpointsProtocol=https;AccountName=',variables('storageAccountName'), ';AccountKey=',listkeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value, ';EndpointSuffix=', environment().suffixes.storage)]",
                        "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING": "[concat('DefaultEndpointsProtocol=https;AccountName=',variables('storageAccountName'), ';AccountKey=',listkeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value, ';EndpointSuffix=', environment().suffixes.storage)]",
                        "WEBSITE_CONTENTSHARE": "[variables('functionAppName')]",
                        "FUNCTIONS_EXTENSION_VERSION": "~3",
                        "FUNCTIONS_WORKER_RUNTIME": "dotnet",
                        "APPINSIGHTS_INSTRUMENTATIONKEY": "[reference(resourceId('Microsoft.Insights/components', variables('appInsightsName')), '2018-05-01-preview').instrumentationKey]",
                        "WEBSITE_RUN_FROM_PACKAGE": "1",
                        "MyFunctionHostKey": "[concat('@Microsoft.KeyVault(SecretUri=', reference(resourceId('Microsoft.KeyVault/vaults/secrets', variables('keyVaultName'), 'functionKey')).secretUriWithVersion, ')')]"
                    }
                }
            ]
        }
```

_You can find the [full ARM template on my GitHub repo](https://github.com/mcollier/azure-functions-host-key/blob/main/azure-deploy.json)._

Application settings are defined as a block.  Meaning, I cannot set just the `MyFunctionHostKey`.  All the settings need to be defined again.  I end up duplicating the settings, which I do not like.

## Summary

I learned the key is to explicitly set the FUNCTIONS_EXTENSION_VERSION to the desired runtime at creation time.  Setting it afterwards will result in an app restart and may yield unexpected results.

### Resources

- [Sample repo](https://github.com/mcollier/azure-functions-host-key)
- [Azure Function keys, keys and more keys, regenerated and synced](https://www.thebestcsharpprogrammerintheworld.com/2020/11/23/azure-function-keys-keys-and-more-keys-regenerated-and-synced/)
