---
title: "Azure Function Secretless Extensions First Experience"
author: "Michael S. Collier"
date: 2021-03-06T09:16:51-05:00
draft: true
comments: true
---

I've recently started experimenting with the beta versions of the new Azure Storage and Event Hub extensions for Azure Functions.  The new extensions use the [new Azure SDK](https://aka.ms/azsdk), and as a result, include support for Azure AD-based authentication.  I'm a fan of having fewer secrets to manage.  This intent with this post is to share my initial experiences with the extensions.  It's still early, and thus I expect a few bumps in the road.

## Getting Started

### Create the resources

To get started, you'll need to [create an Azure Storage account](https://docs.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-cli) and/or an [Event Hub namespace and event hub](https://docs.microsoft.com/azure/event-hubs/event-hubs-create), depending on which resources your Azure Function will use.  You'll also want to [create an Azure Functions Premium plan](https://docs.microsoft.com/azure/azure-functions/functions-premium-plan?tabs=portal#create-a-premium-plan) (an App Service plan should also work). Since the function will connect to Azure Storage or Event Hubs using an identity-based connection, the Azure Function will need to be [set up with a managed identity](https://docs.microsoft.com/azure/app-service/overview-managed-identity?tabs=dotnet).

### Create a function

For the purposes of this post, I'm going to create a few functions which interact with Azure Storage and Event Hubs.

- An Azure Storage queue triggered function
- An Azure Storage blob triggered function which uses an Azure Storage queue output binding
- An Event Hub triggered function
- An HTTP-triggered function using an Azure Storage blob output binding

These functions will not do much more than react to the trigger event and output data.  I'm going to use the default templates from Visual Studio code, making modifications as needed to work with new types from the new Azure SDK.

You will need to add the new extensions to your project:

```dotnetcli
dotnet add package Microsoft.Azure.WebJobs.Extensions.Storage --version 5.0.0-beta.2
dotnet add package Microsoft.Azure.WebJobs.Extensions.EventHubs --version 5.0.0-beta.1
```

## My first function using managed identity

My first function to use an identity-based connection is an Azure Storage queue-triggered function.  My primary objective is to establish the connection (using the managed identity of the Function app) to an Azure Storage queue so that the queue trigger executes.

I start by creating an [Azure Storage queue-triggered function](https://docs.microsoft.com/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp). In order to use the new Storage extension, I do change the _Microsoft.Azure.WebJobs.Extensions.Storage_ extension version from the default (for me, 3.0.10) to be 5.0.0-beta-2.  Additionally, instead of using a `string` or `CloudQueueMessage` as the input type, I change to using the new `QueueMessage` type.  My code now looks like this:

```csharp
using Azure.Storage.Queues.Models;

using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;

namespace Company.Function
{
    public static class QueueTriggerCSharp1
    {
        [FunctionName("QueueTriggerCSharp1")]
        public static void Run([QueueTrigger("%QueueName%", Connection = "MyStorageConnection")] QueueMessage myQueueItem, ILogger log)
        {
            log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
        }
    }
}

```

### Making a connection

With my initial Azure Storage queue-triggered function created locally in Visual Studio Code, I need to set up a connection to an Azure Storage account.  The [documentation](https://docs.microsoft.com/azure/azure-functions/functions-reference#connection-properties) mentions needing to set a "Service URI" property to the URI for the service to which I"m connecting.  I'm not sure what "Service URI" is, or how to set it.  Should there be a `ServiceURI` property on the trigger (like the `Connection` property)?  Should `serviceUri` be part of the app setting value?  It turns out neither.  

I need to set a local setting of name "MyConnectionString__endpoint" and value of the URI for my Azure Storage queue.  My function code set the `QueueTrigger` attribute's `Connection` property to "MyConnectionString".  Thus, my **local.settings.json** file looks as follows:

```json
{
  "IsEncrypted": false,
  "Values": {
    "APPINSIGHTS_INSTRUMENTATIONKEY":"[APPLICATION-INSIGHTS-KEY]",
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "QueueName":"[AZURE-STORAGE-QUEUE-NAME]",
    "MyStorageConnection__endpoint": "https://[AZURE-STORAGE-ACCOUNT-NAME].queue.core.windows.net/"
  }
}

```

When starting the function, I notice this unsettling warning message in my console:

```bash
Warning: Cannot find value named 'MyStorageConnection' in local.settings.json that matches 'connection' property set on 'queueTrigger'
in 'C:\src\blog-az-func-managed-identity\bin\Debug\netcoreapp3.1\QueueTriggerCSharp1\function.json'.
You can run 'func azure functionapp fetch-app-settings <functionAppName>' or specify a connection string in local.settings.json.
```

![Azure Function Core Tools - Unable to find matching connection string setting](/images/azure-function-secretless-extensions-first-experience/az-func-no-matching-connection-string-setting.png)

It is true . . . there is no `MyStorageConnection` setting in my _local.settings.json_ file.  This seems to be a false warning message from the tooling.  Presumably because this identity-based connection feature is new, and still preview, the core tools have not yet be updated to handle identity-based connections.

### Access denied

With that connection string right, the next step is to try to debug my function locally from VS Code, connecting to an Azure Storage account (not the storage emulator).

```bash
[2021-03-07T15:43:32.183Z] An unhandled exception has occurred. Host is shutting down.
[2021-03-07T15:43:32.195Z] Azure.RequestFailedException: This request is not authorized to perform this operation using this permission.
RequestId:25904295-f003-0055-6368-137a75000000
Time:2021-03-07T15:43:31.8268526Z
[2021-03-07T15:43:32.198Z] Status: 403 (This request is not authorized to perform this operation using this permission.)
[2021-03-07T15:43:32.200Z] ErrorCode: AuthorizationPermissionMismatch
[2021-03-07T15:43:32.202Z] 
[2021-03-07T15:43:32.203Z] Content:
[2021-03-07T15:43:32.205Z] <?xml version="1.0" encoding="utf-8"?><Error><Code>AuthorizationPermissionMismatch</Code><Message>This request is not authorized to perform this operation using this permission.
RequestId:25904295-f003-0055-6368-137a75000000
Time:2021-03-07T15:43:31.8268526Z</Message></Error>
[2021-03-07T15:43:32.211Z] 
[2021-03-07T15:43:32.212Z] Headers:
[2021-03-07T15:43:32.214Z] Server: Windows-Azure-Queue/1.0,Microsoft-HTTPAPI/2.0
[2021-03-07T15:43:32.218Z] x-ms-request-id: 25904295-f003-0055-6368-137a75000000
[2021-03-07T15:43:32.225Z] x-ms-version: 2018-11-09
[2021-03-07T15:43:32.228Z] x-ms-error-code: AuthorizationPermissionMismatch
[2021-03-07T15:43:32.231Z] Date: Sun, 07 Mar 2021 15:43:31 GMT
[2021-03-07T15:43:32.233Z] Content-Length: 279
[2021-03-07T15:43:32.235Z] Content-Type: application/xml
[2021-03-07T15:43:32.242Z] 
[2021-03-07T15:43:32.243Z]    at Azure.Storage.Queues.QueueRestClient.GetPropertiesAsync(Nullable`1 timeout, CancellationToken cancellationToken)
[2021-03-07T15:43:32.245Z]    at Azure.Storage.Queues.QueueClient.GetPropertiesInternal(Boolean async, CancellationToken cancellationToken, String operationName)
[2021-03-07T15:43:32.249Z]    at Azure.Storage.Queues.QueueClient.ExistsInternal(Boolean async, CancellationToken cancellationToken)
[2021-03-07T15:43:32.253Z]    at Azure.Storage.Queues.QueueClient.ExistsAsync(CancellationToken cancellationToken)
[2021-03-07T15:43:32.257Z]    at Microsoft.Azure.WebJobs.Extensions.Storage.Common.Listeners.QueueListener.ExecuteAsync(CancellationToken cancellationToken)
[2021-03-07T15:43:32.259Z]    at Microsoft.Azure.WebJobs.Extensions.Storage.Common.Timers.TaskSeriesTimer.RunAsync(CancellationToken cancellationToken)
```

I can work with that.  I need to set the right permissions for my [local identity](https://docs.microsoft.com/azure/azure-functions/functions-reference#local-development) to work with the Azure Storage queue.  I assumed since I am the Service Administrator of my Azure subscription, that I would already have the right permissions.  Apparently that was an incorrect assumption.

Through a bit of trial and error, I learn that I need to put myself in the [Storage Queue Data Contributor](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-queue-data-contributor) role.  I do that [via the Azure portal](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac-portal).

![Azure portal - view assigned Azure Storage RBAC roles](/images/azure-function-secretless-extensions-first-experience/blob-rbac-role-assignment.png)

## Key Considerations

### No Consumption plan . . . yet

Identity-based connections are new, and thus there is not yet full support across all Azure Functions hosting options.  In general, identity-based connections are not supported on the Azure Functions Consumption plan. As of this writing, support is for Azure Functions Premium plan and an App Service/Dedicated plan. It'd be good to keep an eye on the [official docs](https://docs.microsoft.com/azure/azure-functions/functions-reference#configure-an-identity-based-connection) if this should change (overall, or for specific extensions).

### Not all extensions . . . yet

Not all Azure Functions extensions support identity-based connections.  As of this writing, support is limited to [Azure Storage (blobs and queues)](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage/5.0.0-beta.2) and [Event Hubs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventHubs/5.0.0-beta.1).  I assume Service Bus will be coming soon, as Service Bus is in the same (generally speaking) family of messaging/eventing services.

## Conclusion

I'm excited to see how the new extensions develop and to be able to use the extensions, along with managed identity, for Azure Functions triggers and bindings.  Fewer secrets floating is good.  Using Azure AD-based authentication (via managed identity), combined with virtual network-based restrictions such as private endpoints, should provide for two avenues to control access to valuable resources.

### References

- [General Azure Functions developer guidance on using managed identity connections](https://docs.microsoft.com/azure/azure-functions/functions-reference#connections)
- [Azure Web Jobs Storage Queues client library](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Microsoft.Azure.WebJobs.Extensions.Storage.Queues)
- [Azure Web Jobs Event Hubs client library](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.WebJobs.Extensions.EventHubs)
- [Azure Functions Storage Queue Trigger (additional v5.0 info)](https://docs.microsoft.com/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp#additional-types)
- [Azure RBAC - Built-in Roles](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)
- [NuGet Package for Azure Storage Extension](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage/5.0.0-beta.2)
- [NuGet Package for Azure Event Hub Extension](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventHubs/5.0.0-beta.1)
- [Sample on using managed identity from within an Azure Function (via Azure SDK, not the extension)](https://docs.microsoft.com/samples/azure-samples/functions-storage-managed-identity/using-managed-identity-between-azure-functions-and-azure-storage/)
