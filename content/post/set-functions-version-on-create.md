---
title: "Set Functions Version on Create"
author: "Michael S. Collier"
tags: [azure-functions]
date: 2020-11-20T16:13:53-05:00
draft: true
comments: true
---

I recently ran into a situation using the Azure Functions default host key where I didn't understand the behavior I was observing.  Thanks to the help of some fantastic colleagues, we figured out what was going on.  I understand what is happening now.  I want to share here in hopes that my experience will help others that may run into a similar challenge.

## Scenario

I needed to create an Azure Function app via an ARM template.  The ARM template should create the Function app resource, and set the necessary application settings.  One of those app settings should be a Key vault reference to a secret which contains the default host key for the function app.  My function's application code would retrieve that and invoke another function in the app.

It all seems straight-forward enough.  I'm very comfortable with Azure Functions, and feel that I've got a decent grasp on ARM templates.

## How it started

The snippet below shows my starting ARM template.

Did you spot the error?

## The problem

After running the template, I noticed that the Key Vault secret value for the default host key was _not_ the same value the Azure Function portal showed.  

I had two different host keys!  But how?

The problem is where the FUNCTIONS_EXTENSION_VERSION is set.  By default, Azure Functions apps are set to use the v1 runtime.  The Azure Functions v1 runtime persists its key in Azure Files.

I was creating the Function app, and in doing so, the runtime was set to v1.  When retrieving the key in the ARM template (setting to a Key Vault secret), the Azure Functions runtime returned the key from Azure Files.

Another ARM resource block set the Function app's application settings, and it is there where I set the FUNCTIONS_EXTENSION_VERSION to ~3 (v3).  By setting the application setting in a separate step, that forced a restart of the function.  The restart is necessary to change the Azure Function's runtime from v1 to v3.

The v3 runtime uses Azure Blob storage for persisting the keys.  Thus, when looking at the key via the portal after the template finished executing, the key returned was the key from Azure Blob storage.

To summarize, the Key Vault secret value was set, and _then_ the function app restarted.  Meaning, the default host key that was retrieved was a v1 key, not a v3 key.  The process sequence was as follows:

1. Create a Function app with the default runtime (~1).
1. Retrieve the function's default host key and set as a Key Vault secret.
1. Set the Function app's application settings.
1. Function app restarts.
1. Function is running a v3 (~3) runtime.

## How it ended

The "fix" is to set the FUNCTIONS_EXTENSION_VERSION at the same time as the Function app is created.  Which, is exactly what [this documentation](https://docs.microsoft.com/azure/azure-functions/functions-infrastructure-as-code#create-a-function-app) indicates.

![embarrassing]()

If I set the FUNCTIONS_EXTENSION_VERSION to ~3 at app creation time, I get the expected default host key value in Key Vault, and in the portal.  They keys are the same!

It all seems so simple.
