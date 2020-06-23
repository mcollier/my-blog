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

## Azure Virtual Network

A private endpoint gets its IP address from a virtual network.  So, the first Azure resource we'll need to create is a virtual network.  

> I'm not going to go into a lot of details on creating a virtual network in this post.  If you'd like to learn more, please refer to the [official documentation](https://docs.microsoft.com/azure/virtual-network/quick-create-portal).

The virtual network will need at least one subnet.  The subnet will be from where the private endpoint obtains its IP address.  In this post I'm going to create an Azure VM from which I'll connect to the private endpoint of the Azure Function.  I'll connect to the VM using the Azure Bastion service.  Azure Bastion requires a subnet named "AzureBastionSubnet".  While creating the virtual network, I'll create two subnets: "default" and "AzureBastionSubnet".

![Azure Virtual Network with two subnets]()

## Azure Functions Premium plan

## Azure DNS Zone

## Azure Virtual Machine

### Azure Bastion

## Resources

- [Azure Private Link FAQ](https://docs.microsoft.com/azure/private-link/private-link-faq)
- [Azure Virtual Network Service Endpoint Overview](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)
- [Azure Virtual Network Service Endpoint FAQ](https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq#virtual-network-service-endpoints)