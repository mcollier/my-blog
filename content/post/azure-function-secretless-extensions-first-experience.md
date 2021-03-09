---
title: "Azure Function Secretless Extensions - First Experience"
author: "Michael S. Collier"
date: 2021-03-08T09:00:00-05:00
draft: false
comments: true
---

I recently started experimenting with the beta versions of the new Azure Storage and Event Hub extensions for Azure Functions.  The new extensions use the [new Azure SDK](https://aka.ms/azsdk), and as a result, include support for using Azure AD to authenticate to specific Azure resources (a.k.a., [managed identities](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)).  I'm a fan of having fewer secrets to manage.  Less secrets . . . more better.  :wink:

This intent of this blog post is to share my initial experiences with the extensions.  It's still early, and thus I expect a few bumps in along the way.
{{< giphy aMh59aKR8vjdC >}}

## Getting Started

I'm going to start by describing how I was able to get a _very simple_ Azure Storage queue-triggered function to use an identity-based connection.  Later, I'll discuss how I was able to apply these learnings to work with Event Hubs.

> Eager to get to it? Skip to the code samples in my [GitHub repo](https://github.com/mcollier/azure-function-managed-identity-sample).

### Create the resources

I first need to [create an Azure Storage account](https://docs.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-cli).  This storage account will host the queue from which the function will receive messages. I'll also use this storage account when provisioning my function app in Azure.

I know I'm going to eventually deploy this function to Azure.  Thus, I'll create a new [Azure Functions Premium plan](https://docs.microsoft.com/azure/azure-functions/functions-premium-plan?tabs=portal#create-a-premium-plan) (an App Service plan should also work, but I haven't tried yet).  At the time I'm writing this, [Azure Function Consumption plans are not yet supported for use with identity-based connections](https://docs.microsoft.com/azure/azure-functions/functions-reference#configure-an-identity-based-connection).  Since the function will connect to Azure Storage using an identity-based connection, the Azure Function will need to be [set up with a managed identity](https://docs.microsoft.com/azure/app-service/overview-managed-identity?tabs=dotnet).

![Azure Portal - enable managed identity](/images/azure-function-secretless-extensions-first-experience/azure-portal-enable-func-managed-identity.png)

While creating the Azure Function app, I'm also going to create an Application Insights instance.  I like Application Insights available with my Azure Functions.  The [Live Metrics and Logs](https://docs.microsoft.com/azure/azure-functions/functions-monitoring) are incredibly helpful in figuring out what may be going wrong.

<!-- and/or an [Event Hub namespace and event hub](https://docs.microsoft.com/azure/event-hubs/event-hubs-create), depending on which resources your Azure Function will use.  You'll also want to [create an Azure Functions Premium plan](https://docs.microsoft.com/azure/azure-functions/functions-premium-plan?tabs=portal#create-a-premium-plan) (an App Service plan should also work). Since the function will connect to Azure Storage or Event Hubs using an identity-based connection, the Azure Function will need to be [set up with a managed identity](https://docs.microsoft.com/azure/app-service/overview-managed-identity?tabs=dotnet). -->
<!-- 
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
``` -->

## My first function using managed identity

As previously mentioned, my first function to use an identity-based connection is an Azure Storage queue-triggered function.  My primary objective is to establish the connection, using an Azure AD identity, to an Azure Storage queue so that the queue trigger executes.  Since I'm starting locally, I want to use my local identity (on my dev laptop).  Once I deploy to Azure, I want the function to use the identity of the function app.

I start by creating an [Azure Storage queue-triggered function](https://docs.microsoft.com/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp). In order to use the new Storage extension, I need to add the new extension to my project.  This upgrades the extension to use the preview version of the v5.x extension.

```dotnetcli
dotnet add package Microsoft.Azure.WebJobs.Extensions.Storage --version 5.0.0-beta.2
```

Additionally, instead of using a `string` or `CloudQueueMessage` as the input type, I change to using the new `QueueMessage` type.  This change is [due to the use of the v5.x extension](https://docs.microsoft.com/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp#usage).  My code now looks like this:

```csharp
using Azure.Storage.Queues.Models;

using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;

namespace Company.Function
{
    public static class QueueTriggerCSharp1
    {
        [FunctionName("QueueTriggerCSharp1")]
        public static void Run(
          [QueueTrigger("%QueueName%", Connection="MyStorageConnection")] QueueMessage myQueueItem,
          ILogger log)
        {
            log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
        }
    }
}

```

{{< giphy QA1mexM96Rdf4ENJcD >}}

### Making a connection

With my initial Azure Storage queue-triggered function created locally in Visual Studio Code, I need to set up a connection to my Azure Storage account.  The [documentation](https://docs.microsoft.com/azure/azure-functions/functions-reference#connection-properties) mentions needing to set a "Service URI" property to the URI for the service to which I"m connecting.  I'm not sure what "Service URI" is, or how to set it.  Should there be a `ServiceURI` property on the trigger (like the `Connection` property)?  Should `serviceUri` be part of the app setting value?  It turns out neither.  

I need to set a local setting of name "MyConnectionString__endpoint" and value of the URI for my Azure Storage queue.  My function code set the `QueueTrigger` attribute's `Connection` property to "MyConnectionString".  Thus, my _local.settings.json_ file looks as follows:

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

\
When starting the function, I notice this unsettling warning message in my console:

```bash
Warning: Cannot find value named 'MyStorageConnection' in local.settings.json that matches 'connection' property set on 'queueTrigger'
in 'C:\src\blog-az-func-managed-identity\bin\Debug\netcoreapp3.1\QueueTriggerCSharp1\function.json'.
You can run 'func azure functionapp fetch-app-settings <functionAppName>' or specify a connection string in local.settings.json.
```

<!-- ![Azure Function Core Tools - Unable to find matching connection string setting](/images/azure-function-secretless-extensions-first-experience/az-func-no-matching-connection-string-setting.png) -->
\
It is true . . . there is no "MyStorageConnection" setting in my _local.settings.json_ file.  This seems to be a false warning message from the tooling.  Presumably because the identity-based connection feature is new, and still preview, the core tools have not yet been updated to handle identity-based connections.
{{< giphy KEXly2BwaldSlhY8BL >}}

### Access denied

With that connection string right, the next step is to try to debug my function locally from Visual Studio Code, connecting to my Azure Storage account (not the storage emulator).  I try and get this error:

```bash
This request is not authorized to perform this operation using this permission.
RequestId:[REDACTED]
Time:2021-03-08T22:57:07.9749740Z
Status: 403 (This request is not authorized to perform this operation using this permission.)
ErrorCode: AuthorizationPermissionMismatch

Content:
<?xml version="1.0" encoding="utf-8"?><Error><Code>AuthorizationPermissionMismatch</Code><Message>This request is not authorized to perform this operation using this permission.
RequestId:[REDACTED]
Time:2021-03-08T22:57:07.9749740Z</Message></Error>

Headers:
Server: Windows-Azure-Queue/1.0,Microsoft-HTTPAPI/2.0
x-ms-request-id: [REDACTED]
x-ms-version: 2018-11-09
x-ms-error-code: AuthorizationPermissionMismatch
Date: Mon, 08 Mar 2021 22:57:07 GMT
Content-Length: 279
Content-Type: application/xml
```

\
Right . . . I can work with that.  My local identity doesn't have the necessary permissions to work with the designated Azure Storage queue.  I need to set the right permissions for my [local identity](https://docs.microsoft.com/azure/azure-functions/functions-reference#local-development) to work with the Azure Storage queue. Through a bit of trial and error, I learn that I need to put myself in the [Storage Queue Data Contributor](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-queue-data-contributor) role.  I do that [via the Azure portal](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac-portal).

![Azure portal - view assigned Azure Storage RBAC roles](/images/azure-function-secretless-extensions-first-experience/blob-rbac-role-assignment.png)

Once my local identity is in the right Azure AD role, and thus has the correct permissions to work with the Azure Storage queue, my local function is able to process messages!!

![Local function processing messages from Azure Storage queue](/images/azure-function-secretless-extensions-first-experience/az-func-queue-local.png)

### Deploy to Azure

[Publishing my function to Azure is relatively straightforward with Visual Studio Code](https://docs.microsoft.com/azure/azure-functions/create-first-function-vs-code-csharp#5-publish-the-project-to-azure). Like I did for my personal identity, I also need to set the correct permissions for my Azure Function app.  I need to add the function's identity to the Storage Queue Data Contributor role
![Azure Portal - set the function identity to the needed role](/images/azure-function-secretless-extensions-first-experience/azure-portal-enable-func-managed-identity-function-app.png)
![Azure Portal - set the function identity to the needed role](/images/azure-function-secretless-extensions-first-experience/azure-portal-set-func-managed-id-role-function-app.png)

I use [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/) to add a few test messages to the Azure Storage queue to make sure the function is picking up the messages.  I can use the Application Insights Logs to see my highly verbose trace statements. :wink:

![Application Insights log statements](/images/azure-function-secretless-extensions-first-experience/app-insights-log-stroage-queue.png)

## Event Hubs

Now that I know out how to use an identity-based connection with an Azure Storage queue, I want to try doing the same with Event Hubs.  Before I get started with the code, I'm going to need to [create a new Event Hubs namespace and an event hub](https://docs.microsoft.com/azure/event-hubs/event-hubs-create).

I'm going to start with the default Event Hub-triggered function generated by the Visual Studio Code template.  Like with the Azure Storage extension, I need to update my project to use the new Event Hub extension.

```dotnetcli
dotnet add package Microsoft.Azure.WebJobs.Extensions.EventHubs --version 5.0.0-beta.1
```

The new Azure SDK uses a few different data types when working with Event Hubs.  The [sample GitHub code](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.WebJobs.Extensions.EventHubs) is very helpful in figuring out the needed changes.  I need to make a few minor changes:

- Instead of using the `Microsoft.Azure.EventHubs` namespace, use `Azure.Messaging.EventHubs`
- Instead of `eventData.Body.Array`, use `eventData.EventBody`

No other changes were needed, and thus my sample code looks very similar to that generated by the Visual Studio Code template.

```csharp
[FunctionName("EventHubTriggerCSharp1")]
public static async Task Run(
    [EventHubTrigger("%EventHubName%", Connection = "MyEventHubConnection")] EventData[] events,
    ILogger log)
{
    var exceptions = new List<Exception>();

    foreach (EventData eventData in events)
    {
        try
        {
            string messageBody = Encoding.UTF8.GetString(eventData.EventBody);

            // Replace these two lines with your processing logic.
            log.LogInformation($"C# Event Hub trigger function processed a message: {messageBody}");
            await Task.Yield();
        }
        catch (Exception e)
        {
            // We need to keep processing the rest of the batch - capture this exception and continue.
            // Also, consider capturing details of the message that failed processing so it can be processed again later.
            exceptions.Add(e);
        }
    }

    // Once processing of the batch is complete, if any messages in the batch failed processing throw an exception so that there is a record of the failure.

    if (exceptions.Count > 1)
        throw new AggregateException(exceptions);

    if (exceptions.Count == 1)
        throw exceptions.Single();
}
```

\
The [GitHub Azure SDK page for the new extension](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.WebJobs.Extensions.EventHubs#managed-identity-authentication) also shows me the connection string details.  For Event Hubs, the connection string suffix isn't "endpoint", but instead is "fullyQualifiedNamespace".

{{< giphy NEvPzZ8bd1V4Y >}}

Thus, my _local.settings.json_ now appears as follows:

```json
{
  "IsEncrypted": false,
  "Values": {
    "APPINSIGHTS_INSTRUMENTATIONKEY": "[APPLICATION-INSIGHTS-KEY]",
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "QueueName": "[AZURE-STORAGE-QUEUE-NAME]",
    "MyStorageConnection__endpoint": "https://[AZURE-STORAGE-ACCOUNT-NAME].queue.core.windows.net/",
    "EventHubName":"[EVENT-HUB-NAME]",
    "MyEventHubConnection__fullyQualifiedNamespace":"[EVENT-HUB-NAMESPACE].servicebus.windows.net"
  }
}
```

\
Trying to run the function locally now results in the following error:

```bash
EventProcessorHost error (Action='Retrieving list of partition identifiers from a Consumer Client.', HostName='[REDACTED]', PartitionId='').
System.UnauthorizedAccessException: Attempted to perform an unauthorized operation.
	at Azure.Messaging.EventHubs.AmqpError.ThrowIfErrorResponse(AmqpMessage response, String eventHubName)
	at Azure.Messaging.EventHubs.Amqp.AmqpClient.GetPropertiesAsync(EventHubsRetryPolicy retryPolicy, CancellationToken cancellationToken)
	at Azure.Messaging.EventHubs.Amqp.AmqpClient.GetPropertiesAsync(EventHubsRetryPolicy retryPolicy, CancellationToken cancellationToken)
EventProcessorHost error (Action='Retrieving list of partition identifiers from a Consumer Client.', HostName='[REDACTED]', PartitionId='').
	at Azure.Messaging.EventHubs.EventHubConnection.GetPropertiesAsync(EventHubsRetryPolicy retryPolicy, CancellationToken cancellationToken)
System.UnauthorizedAccessException: Attempted to perform an unauthorized operation.
	at Azure.Messaging.EventHubs.EventHubConnection.GetPartitionIdsAsync(EventHubsRetryPolicy retryPolicy, CancellationToken cancellationToken)
	at Azure.Messaging.EventHubs.AmqpError.ThrowIfErrorResponse(AmqpMessage response, String eventHubName)
	at Azure.Messaging.EventHubs.Primitives.EventProcessor`1.RunProcessingAsync(CancellationToken cancellationToken)
	at Azure.Messaging.EventHubs.Amqp.AmqpClient.GetPropertiesAsync(EventHubsRetryPolicy retryPolicy, CancellationToken cancellationToken)
	at Azure.Messaging.EventHubs.Amqp.AmqpClient.GetPropertiesAsync(EventHubsRetryPolicy retryPolicy, CancellationToken cancellationToken)
	at Azure.Messaging.EventHubs.EventHubConnection.GetPropertiesAsync(EventHubsRetryPolicy retryPolicy, CancellationToken cancellationToken)
	at Azure.Messaging.EventHubs.EventHubConnection.GetPartitionIdsAsync(EventHubsRetryPolicy retryPolicy, CancellationToken cancellationToken)
	at Azure.Messaging.EventHubs.Primitives.EventProcessor`1.RunProcessingAsync(CancellationToken cancellationToken)
EventProcessorHost error (Action='Retrieving list of partition identifiers from a Consumer Client.', HostName='[REDACTED]', PartitionId='').
```
\
Got it . . . I _again_ don't have the right permissions.  I need to get my local identity into the [Azure Event Hubs Data Receiver](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#azure-event-hubs-data-receiver) role.  While I'm getting myself right, I might as well do the same for my Azure Function.

![Azure Portal- setting RBAC assignment for Event Hub namespace](/images/azure-function-secretless-extensions-first-experience/event-hub-rbac-assignment.png)

I can use [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer) to add a few events to my newly created Event Hub. Run it again.  No errors!  Ship it!!

{{< giphy ta83CqOoRwfwQ >}}

## Important Considerations

Before fully adopting identity-based connections with Azure Functions, there a few important things to consider.  First and foremost, the extensions are in a preview state.  They may not work as expected ([submit feedback](https://github.com/Azure/azure-sdk-for-net/issues)).  Functionality may still change (expect it).

### No Consumption plan . . . yet

Identity-based connections are new, and thus there is not yet full support across all Azure Functions hosting options.  In general, identity-based connections are not supported on the Azure Functions Consumption plan. As of this writing, support is for Azure Functions Premium plan and an App Service/Dedicated plan. It'd be good to keep an eye on the [official docs](https://docs.microsoft.com/azure/azure-functions/functions-reference#configure-an-identity-based-connection) if this should change (overall, or for specific extensions).

### Not all extensions . . . yet

Not all Azure Functions extensions support identity-based connections.  As of this writing, support is limited to [Azure Storage (blobs and queues)](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage/5.0.0-beta.2) and [Event Hubs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventHubs/5.0.0-beta.1).  I assume Service Bus will be coming soon, as Service Bus is in the same (generally speaking) family of messaging/eventing services.

## Conclusion

I'm excited to see how the new extensions develop and to be able to use the extensions, along with managed identity, for Azure Functions triggers and bindings.  Fewer secrets floating is good.  Using identity-based authentication (via managed identity), combined with virtual network-based restrictions such as private endpoints, should provide for two avenues to control access to valuable resources.

As I keep working with Azure Functions and identity-based extensions, I'll keep my samples in my [GitHub repo](https://github.com/mcollier/azure-function-managed-identity-sample).

{{< giphy ziLadIVnOGCKk >}}

### References

- [General Azure Functions developer guidance on using managed identity connections](https://docs.microsoft.com/azure/azure-functions/functions-reference#connections)
- [Azure Web Jobs Storage Queues client library](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Microsoft.Azure.WebJobs.Extensions.Storage.Queues)
- [Azure Web Jobs Event Hubs client library](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.WebJobs.Extensions.EventHubs)
- [Azure Functions Storage Queue Trigger (additional v5.0 info)](https://docs.microsoft.com/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp#additional-types)
- [Azure RBAC - Built-in Roles](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)
- [NuGet Package for Azure Storage Extension](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage/5.0.0-beta.2)
- [NuGet Package for Azure Event Hub Extension](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventHubs/5.0.0-beta.1)
- [Sample on using managed identity from within an Azure Function (via Azure SDK, not the extension)](https://docs.microsoft.com/samples/azure-samples/functions-storage-managed-identity/using-managed-identity-between-azure-functions-and-azure-storage/)
