---
layout: post
title: "Inbound Private Endpoints With Azure Functions"
date: 2020-06-22T22:05:19-04:00
author: "Michael S. Collier"
tags: [azure-functions, networking, virtual-network, azure-bastion, private-endpoints]
draft: true
---

Earlier this year I wrote a [post showing how to set up private site access for Azure Functions](../azure-functions-private-site-access).  To briefly recap, private site access refers to setting up a virtual network service endpoint to restrict HTTP-based access to the function to be only traffic from the designated virtual network (i.e. inbound HTTP requests).  Attempts to access the public endpoint [e.g., https://contoso.azurewebsites.net](https://contoso.azurewebsites.net) result in an HTTP 403 Forbidden message.  Service endpoints are great, but they are not without some drawbacks (use a public IP address, doesn't work with connections from on-premises resources (i.e. ExpressRoute), limited RBAC features, etc.)

[Azure Private Endpoint](https://docs.microsoft.com/azure/private-link/private-endpoint-overview) offers more granular control of network-based access to specific Azure resources.  One of the major advantages of using a private endpoint is the IP address a private IP address from the virtual network's address space.  Meaning, if you want to access the Azure Function, you will need to be on the virtual network (e.g. a virtual machine connected to the virtual network, an AKS cluster, etc.) and you will connect via the private IP address, such as 10.1.2.3.  Azure DNS private zones will allow you to continue to use [https://contoso.azurewebsites.net](https://contoso.azurewebsites.net) as the DNS-friendly fully qualified domain name.  More on that later.

I recently wrote a [blog post about how to use Azure Functions to interact with Azure resources via private endpoints](../azure-functions-with-private-endpoints).  The blog post demonstrates interacting with private endpoints of an Azure Storage account and Cosmos DB (the SQL API).  From an Azure Functions perspective, these are outbound private endpoints - the function is communicating _out_ to the specific resource via the resource's private endpoint.  The post didn't show how to set up an _inbound_ private endpoint.  Private endpoint (inbound) support for Azure App Service, and by relation Azure Functions, recently expanded the preview offering to be available in nearly all Azure regions.  It's time to learn about using Azure Functions with inbound private endpoints!

## Getting started

Setting up an inbound private endpoint for an Azure Function requires the following Azure resources:

- An Azure Virtual Network
- An Azure Functions Premium plan
- A Private Endpoint
- An Azure DNS Zone (optional)
- An Azure VM (optional)

## Resource group

The first thing I'm going to do is create a new resource group.  All resources for this demo will be placed within the resource group.  This makes it easy for me to delete everything when I'm done.

## Azure Virtual Network

A private endpoint gets its IP address from a virtual network.  Therefore, the first Azure resource I'll need to create is a virtual network.  

> I'm not going to go into a lot of details on creating a virtual network in this post.  To learn more, please refer to the [official documentation](https://docs.microsoft.com/azure/virtual-network/quick-create-portal).

The virtual network will need at least one subnet.  The private endpoint obtains its IP address from the subnet.  The same subnet can be used for the VM, and also for supplying an IP address for the private endpoint.  Later in this post I’m going to create an Azure VM.  The VM will be used to connect to the private endpoint of the Azure Function, as well as serve as a private build agent from which the code will be deployed (more on that later).  I'll connect to the VM using the [Azure Bastion](https://azure.microsoft.com/services/azure-bastion/) service.  

During the virtual network creation steps in the Azure Portal, you are prompted to enable the Azure Bastion host.  This creates the necessary subnet (AzureBastionSubnet) and the Azure Bastion service.

![Create a virtual network with the Azure Bastion service](../../images/inbound-private-endpoints-with-azure-functions/create-virtual-network-azure-bastion.png)

In the end I'll have a virtual network with default subnet, and the Azure Bastion host.

![Azure Virtual Network summary](../../images/inbound-private-endpoints-with-azure-functions/create-virtual-network-summary.png)

## Azure Virtual Machine

Next, I'm going to create an Azure VM within the newly created virtual network.  I'll use this VM as a way to validate that I'm able to access the function endpoint from within the virtual network.  

> I'm not going to go into too many details here, as I feel there is already a good tutorial in the [official documentation](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal).

When creating the VM, there are a few things that I'll do which deviate from the tutorial.

1. Since I'm using Azure Bastion, I don't want any public inbound ports enabled for this VM.
![No public inbound ports on the VM](../../images/inbound-private-endpoints-with-azure-functions/create-vm-basics.png)
1. There is no need for a Public IP for this VM as connections to the VM will be handled via Azure Bastion.
![No public IP for the VM](../../images/inbound-private-endpoints-with-azure-functions/create-vm-networking.png)
1. On the Management section, I'm leaving all the defaults _except_ for enabling auto-shutdown.  It's not strictly required, but I think it's a good practice, especially for dev/test VMs.

## Azure Functions Premium plan

It's time to get to the fun stuff now!  

I'm going to create an Azure Functions Premium plan to host my function app.  The function will be a basic HTTP-triggered function.  This part is pretty straight forward.  

> Instructions on creating an Azure Functions Premium plan can be found in the [official documentation](https://docs.microsoft.com/azure/azure-functions/functions-premium-plan).

To summarize, my Azure Functions Premium plan is set up as follows:

- Runtime stack is .NET Core 3.1
- Create a new storage account
- Create a new premium plan
- Enable Application Insights

In the end, my premium plan looks like this:
![Summary of a new Azure Functions Premium plan](../../images/inbound-private-endpoints-with-azure-functions/create-function-summary.png)

## Enable the Private Endpoint (Inbound)

As mentioned earlier, [Azure Functions can interact with Azure resources which are set up to use a private endpoint](../azure-functions-with-private-endpoints), such as Azure Storage, Service Bus, etc.  From the function's perspective, this is an _outbound_ private endpoint.  The function is reaching out to a specific resource via a private endpoint connection.  What I'm going to set up next is an _inbound_ private endpoint.  The function will have a private endpoint by which other clients can interact with the function.  Meaning, the function can be triggered via a private IP address such as [https://10.1.2.3/api/HelloWorld](https://10.1.2.3/api/HelloWorld).  Don't worry about that IP address too much . . . we'll fix that up with some DNS goodness soon.

### Warning - bug ahead

I believe there is currently a bug in the Azure portal's private endpoint create experience.  I would expect to create a private endpoint from the portal, and in doing so, an Azure Private DNS Zone related to your-function-name.privatelink.azurewebsites.net is automatically created on my behalf.  

Once this bug is resolved, I expect to create a private endpoint from the portal, and in doing so, an Azure Private DNS Zone related to your-function-name.privatelink.azurewebsites.net is automatically created on my behalf.  Creating a private endpoint is a matter of selecting  **Networking** under the _Settings_ section of the function.  On the _Networking_ page there is a link to **Configure your private endpoint connections** under the _Private Endpoint connections_ section.
![Configuring a private endpoint](../../images/inbound-private-endpoints-with-azure-functions/azure-function-select-configure-private-endpoint.png)

From there I could add the private endpoint.  And that mostly works.  The problem is the DNS zone isn't created.  I don't believe it is possible to create a DNS zone myself that can handle requests for *.azurewebsites.net (as Microsoft owns the azurewebsites.net domain).
![Create a private endpoint](../../images/inbound-private-endpoints-with-azure-functions/create-private-endpoint.png)

### The Workaround

The [DNS zone](https://docs.microsoft.com/azure/private-link/private-endpoint-dns#azure-services-dns-zone-configuration) is important as it allows me to invoke the function app using [https://your-function-name.azurewebsites.net](https://your-function-name.azurewebsites.net), instead of [https://10.1.2.3](https://10.1.2.3).  But fear not, there is a workaround!  Like most workarounds, it isn't ideal, but it works.  I'm going to create a private endpoint and then link it to the Azure Function.  This is essentially what the Azure portal should be doing, but it is not currently.  Once the bug in the Azure portal experience is fixed, this workaround will not be necessary.

1. Add a new Private Endpoint resource.
    ![Create a new private endpoint](../../images/inbound-private-endpoints-with-azure-functions/new-private-endpoint-search.png)
1. On the _Basics_ section, give the private endpoint a **Name** and the desired **Region**.
    ![Give the private endpoint a name](../../images/inbound-private-endpoints-with-azure-functions/create-private-endpoint-basics.png)
1. On the _Resource_ section, select the Azure resource for which to associate with the private endpoint.  Azure Functions share much of the same infrastructure as Azure Web Apps, so the _Resource type_ to select will be **Microsoft.Web/sites** and the _Target sub-resource_ is a **site** ("site" is the only option).  I'll then select the desired _Resource_, which is my function.
    ![Select the Azure resource for which to create the private endpoint](../../images/inbound-private-endpoints-with-azure-functions/create-private-endpoint-resource.png)
1. It is on the _Configuration_ section where I'll set up the networking and DNS integration for the private endpoint.  

    Select the desired virtual network and subnet for the private endpoint. Remember, it is from this subnet that the private endpoint's IP address will be allocated.

    I can also choose to integrate with Azure Private DNS Zones. It is not necessary to do so.  It is possible to use my own DNS server, or create DNS records on the necessary VM(s).  I'm lazy, so I'm going to let Azure Private DNS zones handle the DNS magic for me.  Notice that the DNS zone to be created is named _privatelink.azurewebsites.net_.
    ![Set up the network and DNS for the private endpoint](../../images/inbound-private-endpoints-with-azure-functions/create-private-endpoint-configuration.png)

1. This is how I've configured my private endpoint configuration.  The next step is to select **Create** and let the magic happen!
    ![Review the private endpoint configuration](../../images/inbound-private-endpoints-with-azure-functions/create-private-endpoint-review.png)

## Behold . . . the Private Endpoint

It'll take a few minutes for the private endpoint (and DNS Zone) to be created.  Once complete, my resource group contains the resources shown in the below screenshot.  Notice that there is now a new private endpoint, network interface, and private DNS zone.

![Resource group with private endpoint](./../images/inbound-private-endpoints-with-azure-functions/resource-group-after-private-endpoint-created.png)

## Write the code

For the purposes of this post, a basic Azure Function is created by following the steps [Microsoft provides for a quickstart to create a C# function](https://docs.microsoft.com/azure/azure-functions/functions-create-first-function-vs-code?pivots=programming-language-csharp). The quickstart tutorial creates a C# HTTP-triggered Azure Function. I wrote the code and committed it to my Azure DevOps repository.

## Deploy the code

You may recall from my [previous post](../azure-functions-private-site-access) on private site access (using a service endpoint) that each Azure Function includes an advanced tooling site ("Kudu") at [https://your-function-name.scm.azurewebsites.net](https://<your-function-name>.scm.azurewebsites.net).  When using private site access / service endpoints, it is optional to apply the service endpoint to the Kudo site.  The option is gone when using a private endpoint - both the main site and the Kudu/SCM site are both private!

In order to deploy code to the (now private) function, it is necessary to create an agent within the virtual network.  In this case, the "agent" is a VM which is within the virtual network and thus capable of reaching the Kudu/SCM endpoint in order to deploy the code.

Since I already have a VM in the virtual network, I'm going to use that VM as an Azure DevOps self-hosted build agent.  I followed the instructions [here](https://docs.microsoft.com/azure/devops/pipelines/agents/v2-windows?view=azure-devops) to download the Azure DevOps agent to my VM and set it up to work with my Azure DevOps project.

Below is a copy of my Azure Pipelines file. An astute reader will notice that I let the Microsoft-hosted agent perform the build steps, but let the self-hosted agent (on my VM) perform the deployment steps.

```yml

trigger:
- master

variables:
  # Azure Resource Manager connection created during pipeline creation
  azureSubscription: ''

  # Function app name
  functionAppName: 'my-private-function'

  # Agent VM image name
  privateAgent-vmImageName: 'my-private-agent'
  hostedAgent-vmImageName: 'vs2017-win2016'

  # Working Directory
  workingDirectory: '$(System.DefaultWorkingDirectory)/'

stages:
- stage: Build
  displayName: Build stage

  jobs:
  - job: Build
    displayName: Build
    pool:
      # Use a Microsoft-hosted build agent
      vmImage: $(hostedAgent-vmImageName)

    steps:
    - task: DotNetCoreCLI@2
      displayName: Build
      inputs:
        command: 'build'
        projects: |
          $(workingDirectory)/*.csproj
        arguments: --output $(System.DefaultWorkingDirectory)/publish_output --configuration Release

    - task: ArchiveFiles@2
      displayName: 'Archive files'
      inputs:
        rootFolderOrFile: '$(System.DefaultWorkingDirectory)/publish_output'
        includeRootFolder: false
        archiveType: zip
        archiveFile: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
        replaceExistingArchive: true

    - publish: $(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip
      artifact: drop

- stage: Deploy
  displayName: Deploy stage
  dependsOn: Build
  condition: succeeded()

  jobs:
  - deployment: Deploy
    displayName: Deploy
    environment: 'development'
    pool:
      # Use a self-hosted build agent from the Default agent pool.
      name: Default
      vmImage: $(privateAgent-vmImageName)

    strategy:
      runOnce:
        deploy:

          steps:
          - task: AzureFunctionApp@1
            displayName: 'Azure functions app deploy'
            inputs:
              azureSubscription: '$(azureSubscription)'
              appType: functionApp
              appName: $(functionAppName)
              package: '$(Pipeline.Workspace)/drop/$(Build.BuildId).zip'
```

## Try it

Now that all the Azure resources have been provisioned and code deployed, it's time to try it!

Attempting to access the function from my local laptop should fail. Access to the function is forbidden since my laptop is is not on the virtual network.
![Access to function app is forbidden](../../images/inbound-private-endpoints-with-azure-functions/function-app-forbidden.png)

However, when I access function from the VM, there is much rejoicing!!
![Access to the function app is successful](../../images/inbound-private-endpoints-with-azure-functions/function-app-success.png)

In fact, performing a `nslookup` command from the VM shows that the private endpoint is being used.

```dos
>nslookup my-private-function.azurewebsites.net
Server:  UnKnown
Address:  168.63.129.16

Non-authoritative answer:
Name:    my-private-function.privatelink.azurewebsites.net
Address:  10.1.0.5
Aliases:  my-private-function.azurewebsites.net
```

## Summary

Inbound private endpoints with Azure Functions provide a private IP address for accessing a function.  Anything that needs to access the function must be on the virtual network, and access the function using the function's private IP address (or have DNS set up to route to the private address instead of the public address).  Setting up the private endpoint helps keep the function secure (from a network access perspective) by keeping inbound traffic within the virtual network.

It is possible to use private endpoints for both inbound and outbound connections.  I could have an HTTP-triggered function, which has a private endpoint, that writes out to Cosmos DB or Service Bus via an outbound private endpoint.  I could also set up Azure Storage to have a private endpoint, and use that for the function's AzureWebJobStorage setting. Doing so would enable me to _almost_ completely restrict access to the function and dependent resources.  The lone outstanding piece is the Azure Storage account used for the function's web content file share.  That storage account, currently, must not have any virtual network restrictions. That's getting fixed soon though!!!

I hope you found this post useful.  Please leave feedback or comments below, or reach out via [Twitter](https://www.twitter.com/michaelcollier).

## Resources

- [Connect privately to a Web App using Azure Private Endpoint](https://docs.microsoft.com/azure/private-link/create-private-endpoint-webapp-portal)
- [Using Private Endpoints for Azure Web App](https://docs.microsoft.com/azure/app-service/networking/private-endpoint)
- [Azure Private Link FAQ](https://docs.microsoft.com/azure/private-link/private-link-faq)
- [Azure Virtual Network Service Endpoint Overview](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)
- [Azure Virtual Network Service Endpoint FAQ](https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq#virtual-network-service-endpoints)