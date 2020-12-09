---
title: "How to change the Azure Functions system keys"
author: "Michael S. Collier"
tags: [azure-functions]
date: 2020-12-08T10:15:27-05:00
draft: true
comments: true
---

I was recently asked how to change Azure Functions' [system keys](https://docs.microsoft.com/azure/azure-functions/security-concepts#system-key), such as the ones automatically created by the Event Grid or Durable Functions extensions.

It's possible to change these keys via the Azure portal.  There is a button in the portal to generate a new key.

![Azure portal to generate new function keys](/images/azure-functions-system-key-change/portal-renew-key-value-with-arrows.png)

What if you want to change the keys programmatically?  I couldn't find official documentation which stated how to do so.  After a bit of splunking through GitHub issues ([here](https://github.com/Azure/azure-functions-host/issues/4728), [here](https://github.com/Azure/azure-functions-host/issues/3994#issuecomment-472108298)) and reading [Mark Heath's excellent blog post on Azure Function keys](https://markheath.net/post/managing-azure-functions-keys-2), I think I found an approach that, so far, seems to work.

Before changing the keys, I'd like to retrieve the current keys.  If I can retrieve the keys through an API, I feel that is a good indication that I should be able to update the keys.

## Get the keys

To get the keys, I'm going to use the [Azure CLI's](https://docs.microsoft.com/cli/azure/what-is-azure-cli) `az rest` [command](https://docs.microsoft.com/cli/azure/reference-index?view=azure-cli-latest#az_rest).

```bash
#!/bin/bash

resourceGroupName=myResourceGroup
functionAppName=myawesomefunction

functionId=$(az functionapp show --name $functionAppName --resource-group $resourceGroupName --query id --output tsv)

echo "Getting the keys for function $functionId"

# Get the current key.
az rest --method post --url https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/$resourceGroupName/providers/Microsoft.Web/sites/$functionAppName/host/default/listkeys?api-version=2016-08-01
```

I should get a response like the following:

```json
{
  "functionKeys": {
    "default": "xxxxxxxxxxx"
  },
  "masterKey": "yyyyyyyyyyy",
  "systemKeys": {
    "eventgrid_extension": "zzzzzzzzzzzzz"
  }
}
```

Now that I can retrieve the keys, I'd like to change the Event Grid extension key, `eventgrid_extension`.

## Update the keys

I'll use the Azure CLI's `az rest` command again to update the keys.

```bash
#!/bin/bash

resourceGroupName=myResourceGroup
functionAppName=myawesomefunction
keyName=eventgrid_extension

functionId=$(az functionapp show --name $functionAppName --resource-group $resourceGroupName --query id --output tsv)

echo "Updating the keys for function $functionId"

# Change the key.
az rest --method put --body @body.json --url https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/$resourceGroupName/providers/Microsoft.Web/sites/$functionAppName/host/default/systemkeys/$keyName?api-version=2016-08-01
```

The content of the *body.json* file is as follows:

```json
{
    "properties": {
        "name": "eventgrid_extension",
        "value": "xxxxx183c4eb49f289f23940bf0yyyyy"
    }
}
```

The output of running that `az rest` command to change the key is as follows:

```json
{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Web/sites/myawesomefunction/host/default/systemkeys/eventgrid_extension",
  "location": "East US",
  "name": "eventgrid_extension",
  "properties": {
    "name": "eventgrid_extension",
    "value": "xxxxx183c4eb49f289f23940bf0yyyyy"
  },
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.Web/sites/host/systemkeys"
}
```

I had Application Insights hooked up to my function app so I could monitor what happens when I attempt to update the key.  By watching Live Metrics, I can see the runtime detects the change.

![Azure portal Live Metrics](/images/azure-functions-system-key-change/portal-ai-live-metrics-key-changed-small.png)

## Conclusion

## Resources

- Mark Heath's [Managing Azure Functions Keys (using the new ARM APIs!)](https://markheath.net/post/managing-azure-functions-keys-2) blog post. It's excellent!!
- [Key management API](https://github.com/Azure/azure-functions-host/wiki/Key-management-API). It's old, but still helpful.
