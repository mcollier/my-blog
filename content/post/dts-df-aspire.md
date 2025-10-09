---
title: "Durable Task Scheduler + Durable Functions + .NET Aspire"
date: 2025-10-09T02:42:45Z
draft: true
categories: [azure-functions]
author: "Michael S. Collier"
tags: [azure-functions, aspire]
---

It's no secret that I love Azure Functions (just take a look at my prior posts).  That love extends to Azure Durable Functions.  As much as I love Durable Functions, there are certainly some challenges with using Azure Storage as a backend. The new Durable Task Scheduler service looks well-positioned to remedy many of those challenges.  The best things comes in threes, so I thought it would be fun to use .NET Aspire to help coordinate my Durable Functions project and Durable Task Scheduler (DTS).  Durable Functions + Durable Task Scheduler + .NET Aspire == Awesome!

I love this setup, especially for local development.  By using .NET Aspire, I've found it easy to set up my Durable Functions project, along with the Durable Task Scheduler emulator.  I let Aspire ensure the emulator is running - one less thing for me to worry about.  Aspire can spin up any container via the `AddContainer` method:

``` csharp
var dts = builder.AddContainer("dts", "mcr.microsoft.com/dts/dts-emulator:latest")
                    .WithLifetime(ContainerLifetime.Persistent)
                    .WithHttpEndpoint(port: 8080, targetPort: 8080, name: "dts-grpc")
                    .WithHttpEndpoint(port: 8082, targetPort: 8082, name: "dts-dashboard");
```

One dependency solved!  Next?

Azure Functions has a dependency on Azure Storage.  That dependency doesn't go away just because I'm using Durable Task Scheduler.  Azure Storage is still needed.  Even when DTS backs orchestration state, the Functions host still requires Azure Storage for triggers/bindings and host checkpoints, so Azurite remains part of the local stack.  Aspire helps here too, as I can use Aspire's existing Azure Storage integration, running the Azure Storage emulator locally.  

``` csharp
var storage = builder.AddAzureStorage(Shared.Services.AzureStorage)
                     .RunAsEmulator(azurite =>
                     {
                         azurite.WithBlobPort(27000)
                                .WithQueuePort(27001)
                                .WithTablePort(27002)
                                .WithLifetime(ContainerLifetime.Persistent);
                     });
```

Here I'm setting up Aspire to use the Azure Storage emulator, and setting consistent ports for the table, blob, and queue service. If I don't, Aspire assigns random ports.  The consistent ports make it easy for me to connect to the Azure Storage service emulator via tools such as Azure Storage Explorer.

Now I've got Aspire handling the following:

- Azure Durable Function project
- Azure Storage emulator
- Azure Durable Task Scheduler emulator

Winning!

Additionally, I flow two values from the AppHost’s appsettings: the DTS connection string and the task hub name. Aspire exposes them as parameters and injects them as environment variables consumed by the Functions app

- Azure Durable Task Scheduler connection string: the connection string for the local Durable Task Scheduler emulator
- Azure Durable Task Scheduler task hub name: the Durable Task hub name

``` json
"Parameters": {
    "azureDurableTaskSchedulerTaskHubName": "default",
    "azureDurableTaskSchedulerConnectionString": "Endpoint=http://localhost:8080;Authentication=None"
  }
```

These parameters get set as environment variables that are used by the Durable Functions project.

``` csharp
var builder = DistributedApplication.CreateBuilder(args);

// Pull in the connection string for the Durable Task Scheduler.
var azureDurableTaskSchedulerConnectionString = builder.AddParameter("azureDurableTaskSchedulerConnectionString");
var azureDurableTaskSchedulerTaskHubName = builder.AddParameter("azureDurableTaskSchedulerTaskHubName");

var storage = builder.AddAzureStorage(Shared.Services.AzureStorage)
                     .RunAsEmulator(azurite =>
                     {
                         azurite.WithBlobPort(27000)
                                .WithQueuePort(27001)
                                .WithTablePort(27002)
                                .WithLifetime(ContainerLifetime.Persistent);
                     });
var blobs = storage.AddBlobs(Shared.Services.AzureStorageBlobs);
var queues = storage.AddQueues(Shared.Services.AzureStorageQueues);
var tables = storage.AddTables(Shared.Services.AzureStorageTables);

var function = builder.AddAzureFunctionsProject<Projects.FunctionsService>(Shared.Services.FunctionsService)
                        .WithHostStorage(storage)
                        .WithEnvironment("DURABLE_TASK_SCHEDULER_CONNECTION_STRING", azureDurableTaskSchedulerConnectionString)
                        .WithEnvironment("TASKHUB_NAME", azureDurableTaskSchedulerTaskHubName);

// Only use the Durable Task Scheduler in run mode.
// In local development, we use the DTS emulator.
if (builder.ExecutionContext.IsRunMode)
{
    var dts = builder.AddContainer("dts", "mcr.microsoft.com/dts/dts-emulator:latest")
                    .WithLifetime(ContainerLifetime.Persistent)
                    .WithHttpEndpoint(port: 8080, targetPort: 8080, name: "dts-grpc")
                    .WithHttpEndpoint(port: 8082, targetPort: 8082, name: "dts-dashboard");

    function.WaitFor(dts);
}

builder.Build().Run();

```

### References

- [Configure a Durable Functions app to use Azure Functions Durable Task Scheduler](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-task-scheduler/quickstart-durable-task-scheduler?pivots=csharp)
