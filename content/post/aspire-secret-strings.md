---
layout: post
title: "Removing Secret Strings from Your .NET Aspire Project"
author: "Michael S. Collier"
date: 2025-08-07T03:28:59Z
tags: [dotnet, aspire]
draft: true
---

If you're building modern .NET Aspire apps, you're probably familiar with how service names and resource identifiers are often passed around as string literals—things like `"apiservice"` or `"storage"`. But these **magic strings** can lead to headaches: typos, duplication, poor discoverability, and painful refactoring.

Thankfully, there's a clean and safe way to centralize and manage these identifiers using a shared constants class, **removing "secret strings" from your Aspire project.** :boom:

## :sparkles: Credit Where Credit’s Due

First, big thanks to [Jeff Fritz](https://www.youtube.com/@csharpfritz) for showcasing this pattern in his excellent video: [“Organize your .NET Aspire Projects like a Pro!”](https://www.youtube.com/watch?v=Jt39GzYCRgo). This post builds directly on that idea and shows how you can apply it to both project references and cloud resources like Azure Storage.

## :dart: The Problem: Magic Strings in AppHost.cs

Here’s what the typical Aspire AppHost.cs might look like:

```csharp
var apiService = builder.AddProject<Projects.AgentFunction_ApiService>("apiservice");
```

Or when registering Azure Storage resources:

```csharp
var storage = builder.AddAzureStorage("storage")
                     .RunAsEmulator();
var blobs = storage.AddBlobs("blobs");
var queues = storage.AddQueues("queues");
```

These string literals (`"apiservice"`, `"storage"`, `"blobs"`, etc.) are fragile and hard to reuse safely across your codebase.

## 🛠 Step 1: Create a New Shared Project

Create a new class library in your solution:

```bash
dotnet new classlib -n Shared
```

This project will house your service and resource names.

## :computer: Step 2: Define Constants in Shared/Services.cs

Create a static class in the Shared project:

```csharp
namespace Shared;

public static class Services
{
    public const string ApiService = "apiservice";
    public const string AzureStorage = "storage";
    public const string AzureStorageBlobs = "blobs";
    public const string AzureStorageQueues = "queues";
}
```

Now you have a single source of truth for all your Aspire identifiers. :dart:

## :link: Step 3: Reference Shared from AppHost

Edit the `AppHost.csproj` to include a reference to `Shared`, but be sure to opt out of Aspire’s resource detection using the `IsAspireProjectResource="false"` attribute:

```xml
<ProjectReference Include="..\Shared\Shared.csproj" IsAspireProjectResource="false" />
```

This ensures Aspire doesn't treat `Shared` as a resource provider.

## :hammer: Step 4: Update AppHost.cs to Use Constants

Now refactor your code to use those constants:

### :hourglass_flowing_sand: Old

```csharp
var apiService = builder.AddProject<Projects.AgentFunction_ApiService>("apiservice");
```

### :white_check_mark: New

```csharp
var apiService = builder.AddProject<Projects.ApiService>(Shared.Services.ApiService);
```

And for Azure Storage:

### :hourglass_flowing_sand: Old

```csharp
var storage = builder.AddAzureStorage("storage")
                     .RunAsEmulator();
var blobs = storage.AddBlobs("blobs");
var queues = storage.AddQueues("queues");

```

### :white_check_mark: New

```csharp
var storage = builder.AddAzureStorage(Shared.Services.AzureStorage)
                     .RunAsEmulator();
var blobs = storage.AddBlobs(Shared.Services.AzureStorageBlobs);
var queues = storage.AddQueues(Shared.Services.AzureStorageQueues);
```

Cleaner. Safer. Easier to manage. :bulb:

## :balloon: Step 5: Reference Shared in Client Projects (Optional)

If you have other Aspire projects (e.g., frontend apps or background workers) that also use these service names, add a project reference to `Shared` there as well. 

```xml
<ProjectReference Include="..\Shared\Shared.csproj" IsAspireProjectResource="false" />
```

Now your whole solution can rely on centralized, strongly-typed service identifiers.

## :bell: Bonus: Shorten Project Names with AspireProjectMetadataTypeName

In addition to removing magic strings, you can simplify long project names in Aspire by explicitly specifying the metadata type name in your project reference.

### :hourglass_flowing_sand: Old

```xml
<ProjectReference Include="..\AgentFunction.ApiService\AgentFunction.ApiService.csproj" />
```

This results in awkward long names like:

```csharp
builder.AddProject<Projects.AgentFunction_ApiService>("apiservice");
```

### :white_check_mark: New

```xml
<ProjectReference Include="..\AgentFunction.ApiService\AgentFunction.ApiService.csproj"
                  AspireProjectMetadataTypeName="ApiService" />
```

Now you can shorten the reference in AppHost.cs to:

```csharp
builder.AddProject<Projects.ApiService>(Shared.Services.ApiService);
```

This makes your code cleaner and more readable, especially in solutions with deeply nested or namespaced projects.

## :revolving_hearts: Benefits Recap

:white_check_mark: Eliminate fragile magic strings

:white_check_mark: Enable compiler support and refactoring

:white_check_mark: Improve discoverability of services and resources

:white_check_mark: Keep your solution organized and maintainable

## :tv: Learn More

:point_right: Watch [Jeff Fritz’s video](https://www.youtube.com/watch?v=Jt39GzYCRgo) for a great walkthrough of this and other .NET Aspire tips.

## :raised_hands: Final Thoughts

Removing magic strings is a small refactor that pays big dividends in maintainability and clarity. Give it a try in your .NET Aspire project—and say goodbye to those “secret strings” once and for all. :unlock:
