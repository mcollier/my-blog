---
title: "Azure Functions System Key Change"
author: "Michael S. Collier"
tags: [azure-functions]
date: 2020-12-08T10:15:27-05:00
draft: true
comments: true
---

I was recently asked about a strategy for managing the various keys used by Azure Functions.  Specifically, the question centered around how to change the system keys, such as the ones automatically created by the Event Grid or Durable Functions extensions.

It's possible to change these keys via the Azure portal.  There is a button in the portal to generate a new key.

![Azure portal to generate new function keys]()

What if you want to change the keys programmatically?  I couldn't find documentation which stated how to do so.  After a bit of splunking through GitHub issues ([here](https://github.com/Azure/azure-functions-host/issues/4728), [here](https://github.com/Azure/azure-functions-host/issues/3994#issuecomment-472108298)), I think I found an approach that, so far, seems to work.

Before changing the keys, I'd like to retrieve the current keys.  If I can retrieve the keys through an API, I feel that is a good indication that I should be able to update the keys.

## Get the keys

To get the keys, I'm going to use the Azure CLI's `az rest` command.

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

```

Now that I can retrieve the keys, I'd like to change the Event Grid extension key, ``.

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
az rest --method put --debug --body @body.json --url https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/$resourceGroupName/providers/Microsoft.Web/sites/$functionAppName/host/default/systemkeys/$keyName?api-version=2016-08-01
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

I had Application Insights hooked up to my function app so I could monitor what happens when I attempt to update the key.  By watching Live Metrics, I can see the runtime detects the change.

![Azure portal Live Metrics]()

## Conclusion

## Resources
