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
- New storage account
- New premium plan
- Enabled Application Insights

In the end, my premium plan looks like this:
![Summary of a new Azure Functions Premium plan](../../images/inbound-private-endpoints-with-azure-functions/create-function-summary.png)

## Azure DNS Zone

## Resources

- [Azure Private Link FAQ](https://docs.microsoft.com/azure/private-link/private-link-faq)
- [Azure Virtual Network Service Endpoint Overview](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)
- [Azure Virtual Network Service Endpoint FAQ](https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq#virtual-network-service-endpoints)