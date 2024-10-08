---
title: Application Gateway classic to Resource Manager
description: Learn about moving Application Gateway resources from the classic deployment model to the Resource Manager deployment model.
services: application-gateway
author: greg-lindsay
ms.service: azure-application-gateway
ms.topic: how-to
ms.date: 06/27/2024
ms.author: greglin
---

# Application Gateway classic to Resource Manager migration

Resource Manager enables deploying complex applications through templates, configures virtual machines by using VM extensions, and incorporates access management and tagging. Azure Resource Manager includes scalable, parallel deployment for virtual machines into availability sets. The new deployment model also provides lifecycle management of compute, network, and storage independently.
You can read more about Azure Resource Manager [features and benefits](../azure-resource-manager/management/overview.md).

Application Gateway resources will **not** be migrated automatically as part of VNet migration from classic to Resource Manager.
As part of VNet migration process as documented at [IaaS resources migration page](/azure/virtual-machines/migration-classic-resource-manager-ps), if you have an Application Gateway resource present on the VNet that you're trying to migrate to Resource Manager deployment model, the automatic migration wouldn't be successful. 

In order to migrate your Application Gateway resource to Resource Manager deployment model, you'll have to remove the Application Resource from the VNet before beginning migration and then recreate the Application Gateway resource once migration is complete.

## Creating a new Application Gateway resource 

For more information on how to set up an Application Gateway resource after VNet migration, you can refer:

* [Deployment via portal](quick-create-portal.md)
* [Deployment via PowerShell](quick-create-powershell.md)
* [Deployment via Azure CLI](quick-create-cli.md)
* [Deployment via ARM template](quick-create-template.md)

## Common questions

### What is Azure Service Manager and what does it mean by classic?

The word "classic" in classic networking service refers to networking resources managed by Azure Service Manager (ASM). Azure Service Manager (ASM) is the old control plane of Azure responsible for creating, managing, deleting VMs and performing other control plane operations.

> [!NOTE]
> To view all the classic resources in your subscription, Open the **All Resources** blade and look for a **(Classic)** suffix after the resource name.

### What is Azure Resource Manager?

Azure Resource Manager is the latest control plane of Azure responsible for creating, managing, deleting VMs and performing other control plane operations.

### Where can I find more information regarding classic to Azure Resource Manager migration?

Please refer to [Frequently asked questions about classic to Azure Resource Manager migration](/azure/virtual-machines/migration-classic-resource-manager-faq)

### How do I report an issue?

Post your issues and questions about migration to our [Microsoft Q&A page](/answers/topics/azure-virtual-network.html). We recommend posting all your questions on this forum. If you have a support contract, you're welcome to log a support ticket as well.

## Next steps
To get started see: [platform-supported migration of IaaS resources from classic to Resource Manager](/azure/virtual-machines/migration-classic-resource-manager-ps)

For any concerns around migration, you can contact Azure Support. Learn more about [Azure support here](https://azure.microsoft.com/support/options/).