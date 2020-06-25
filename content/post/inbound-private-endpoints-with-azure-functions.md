---
layout: post
title: "Inbound Private Endpoints With Azure Functions"
date: 2020-06-22T22:05:19-04:00
author: "Michael S. Collier"
tags: [azure-functions, networking, virtual-network, azure-bastion, private-endpoints]
draft: true
---

Earlier this year I wrote a post showing how to set up private site access for Azure Functions.  In a nutshell, private site access is setting up a virtual network service endpoint to restrict access to the function to only traffic from the designated virtual network.  Service endpoints are great, but they are not without some drawbacks (public IP address, doesn't work with connections from on-premises resources (i.e. ExpressRoute), limited RBAC features, etc.)

In my opinion, [Azure Private Endpoint](https://docs.microsoft.com/azure/private-link/private-endpoint-overview) offer more granular control of network based access to specific Azure resources.  One of the major advantages of using a private endpoint is IP address associated with the designated resource is a private IP address from the virtual network's address space.  Meaning, if you want to access the Azure Function, you would need to be on the virtual network (e.g. a virtual machine connected to the virtual network, an AKS cluster, etc.) and you would connect via the private IP address, such as 10.1.2.3.  Azure DNS private zones will allow you to continue to use [https://contoso.azurewebsites.net](https://contoso.azurewebsites.net) as the DNS-friendly fully qualified domain name.  More on that later.

I recently wrote a blog post about how to use Azure Functions to interact with Azure resources via private endpoints.  The blog post demonstrates using private endpoints of an Azure Storage account and CosmosDB (the SQL API).  From an Azure Functions perspective, these are outbound private endpoints - the function is communicating out to the specific resource via the resource's private endpoint.  It didn't show how to set up an _inbound_ private endpoint.  Private endpoint (inbound) support for Azure App Service, and by relation Azure Functions, recently expanded the preview offering to be available in nearly all Azure regions.  It's time to learn about using Azure Functions with inbound private endpoints!

## Let's get started

Setting up an inbound private endpoint for an Azure Function requires the following Azure resources:

- A virtual network
- An Azure Functions Premium plan
- An Azure DNS Zone (optional)
- An Azure VM (optional)

## Resource Group

The first thing I'm going to do is create a new resource group.  All resources for this demo will be placed within the resource group.  This makes it easy for me to delete everything when I'm done.

## Azure Virtual Network

A private endpoint gets its IP address from a virtual network.  So, the first Azure resource we'll need to create is a virtual network.  

> I'm not going to go into a lot of details on creating a virtual network in this post.  If you'd like to learn more, please refer to the [official documentation](https://docs.microsoft.com/azure/virtual-network/quick-create-portal).

The virtual network will need at least one subnet.  The subnet will be from where the private endpoint obtains its IP address.  Later in this post I'm going to create an Azure VM from which I'll connect to the private endpoint of the Azure Function.  I'll connect to the VM using the [Azure Bastion](https://azure.microsoft.com/services/azure-bastion/) service.  

During the virtual network creation steps in the Azure Portal, you are prompted to enable the Azure Bastion host.  This creates the necessary subnet (AzureBastionSubnet) and the Azure Bastion service.

![Create a virtual network with the Azure Bastion service](../../images/inbound-private-endpoints-with-azure-functions/create-virtual-network-azure-bastion.png)

In the end I'll have a virtual network with default subnet, and the Azure Bastion host.

![Azure Virtual Network summary](../../images/inbound-private-endpoints-with-azure-functions/create-virtual-network-summary.png)

## Azure Virtual Machine

Next I'm going to create a Azure VM within the newly created virtual network.  I'll use this VM as a way to validate that I'm able to access the function endpoint from within the virtual network.  I'm not going to go into too many details here, as I feel there is already a good tutorial in the [official documentation](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal).

There are a few things I will call out though.

1. Since I'm using Azure Bastion, I don't want any public inbound ports enabled for this VM.
![No public inbound ports on the VM](../../images/inbound-private-endpoints-with-azure-functions/create-vm-basics.png)
1. There is no need for Public IP for this VM as connection to the VM will be handled via Azure Bastion.
![No public IP for the VM](../../images/inbound-private-endpoints-with-azure-functions/create-vm-networking.png)
1. On the Management section, leaving all the defaults _except_ for enabling auto-shutdown.  It's not strictly required, but I think it's a good practice, especially for dev/test VMs.

## Azure Functions Premium plan

It's time to get to the fun stuff now!  

I'm going to create an Azure Functions Premium plan to host my function app.  The function will be a basic HTTP-triggered function.  This part is pretty straight forward.  Nothing special.  You can find instructions on creating an Azure Functions Premium plan in the [official documentation](https://docs.microsoft.com/azure/azure-functions/functions-premium-plan).

To summarize, my Azure Functions Premium plan is set up as follows:

- Runtime stack is .NET Core 3.1
- Create a new storage account
- Create a new premium plan
- Enable Application Insights

In the end, my premium plan looks like this:
![Summary of a new Azure Functions Premium plan](../../images/inbound-private-endpoints-with-azure-functions/create-function-summary.png)

## Enable the Private Endpoint (Inbound)

Azure Functions can interact with Azure resources which are set up to use a private endpoint, such as Azure Storage, Service Bus, etc.  From the function's perspective, this is an _outbound_ private endpoint.  The function is reaching out to a specific resource via a private endpoint connection.  What I'm going to set up next is an _inbound_ private endpoint.  The function will have a private endpoint by which other clients can interact with the function.  Meaning, the function can be triggered via a private IP address such as [https://10.1.2.3/api/HelloWorld](https://10.1.2.3/api/HelloWorld).  Don't worry about that IP address too much . . . we'll fix that up with some DNS goodness soon.

### Warning - bug ahead

I believe there is currently a bug in the Azure portal's private endpoint create experience.  I would expect to create a private endpoint from the portal, and in doing so, an Azure Private DNS Zone related to <your-function-name>.privatelink.azurewebsites.net is automatically created on my behalf.  Currently, the portal experience is not creating the DNS zone.

Thes [DNS zone](https://docs.microsoft.com/azure/private-link/private-endpoint-dns#azure-services-dns-zone-configuration) is important as it allows me to invoke the function app using the FQDN of https://<your-function-name>.azurewebsites.net, instead of [https://10.1.2.3](https://10.1.2.3).  But fear not, there is a workaround!  Like most workarounds, it isn't ideal, but it works.  I'm going to create a private endpoint and then link it to the Azure Function.  What . . .

1. Add a new Private Endpoint resource.
    ![Create a new private endpoint](../../images/inbound-private-endpoints-with-azure-functions/new-private-endpoint-search.png)
1. On the _Basics_ section, give the private endpoint a **Name** and the desired **Region**.
    ![Give the private endpoint a name](../../images/inbound-private-endpoints-with-azure-functions/create-private-endpoint-basics.png)
1. On the _Resource_ section, select the Azure resource for which to associate with the private endpoint.  Azure Functions share much of the same infrastructure as Azure Web Apps, so the _Resource type_ to select will be **Microsoft.Web/sites** and the _Target sub-resource_ is a **site** ("site" is the only option).  I'll then select the desired function.
    ![Select the Azure resource for which to create the private endpoint](../../images/inbound-private-endpoints-with-azure-functions/create-private-endpoint-resource.png)
1. It is on the _Configuration_ section where I'll set up the networking and DNS integration for the private endpoint.  

Select the desired virtual network and subnet for the private endpoint. Remember, it is from this subnet that the private endpoint's IP address will be allocated.

I can also choose to integrate with Azure Private DNS Zones. It is not necessary to do so.  It is possible to use my own DNS server, or create DNS records on the necessary VM(s).  I'm lazy, so I'm going to let Azure Private DNS zones handle the DNS magic for me.  Notice that the DNS zone to be created is named _privatelink.azurewebsites.net_.
    ![Set up the network and DNS for the private endpoint](../../images/inbound-private-endpoints-with-azure-functions/create-private-endpoint-configuration.png)

1. This is how I've configured my private endpoint configuration.  The next step is to select **Create** and let the magic happen!
    ![Review the private endpoint configuration](../../images/inbound-private-endpoints-with-azure-functions/create-private-endpoint-review.png)

<!-- 1. Navigate to the newly created Azure Function app, and select **Networking** within the _Settings_ section.
    ![Select the Networking option within the Settings section](../../images/inbound-private-endpoints-with-azure-functions/)
1. On the _Networking_ page, select the link to **Configure your private endpoint connections** under the _Private Endpoint connections_ section.
    ![Select Configure your private endpoints connection](../../images/inbound-private-endpoints-with-azure-functions/)
1. On the _Private Endpoint connections_ page, select the **Add** button to create a new connection. -->

<!-- As the popular saying goes,

> "There are only two hard things in Computer Science: cache invalidation and naming things." — Phil Karlton from Martin Fowler -->

## Behold . . . the Private Endpoint

It'll take a few minutes for the private endpoint (and DNS Zone) to be created.  Once complete, my resource group contains the resources shown in the below screenshot.  Notice that there is now a new private endpoint, network interface, and private DNS zone.

![Resource group with private endpoint](./../images/inbound-private-endpoints-with-azure-functions/resource-group-after-private-endpoint-created.png)

## Deploy the code

You may recall from my [previous post](2020-01-22-azure-functions-private-site-access) on private site access (using a service endpoint) that each Azure Function includes an advanced tooling site ("Kudu") at [https://<your-function-name>.scm.azurewebsites.net>](https://<your-function-name>.scm.azurewebsites.net).  When using private site access / service endpoints, it is optional to apply the service endpoint to the Kudo site.  The option is gone when using a private endpoint - both the main site and the Kudu/SCM site are both private!

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

## Summary

## Resources

- [Azure Private Link FAQ](https://docs.microsoft.com/azure/private-link/private-link-faq)
- [Azure Virtual Network Service Endpoint Overview](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)
- [Azure Virtual Network Service Endpoint FAQ](https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq#virtual-network-service-endpoints)