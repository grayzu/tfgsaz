---
id: intro_az
level: Getting Started
products_used:
  - Terraform
layout: content_layout
name: 'Introduction'
content_length: 3
description: |-
  Introduction to getting started with Terraform .12 and the Azure provider.
---

_Revised and expanded for Terraform v0.12_

This getting started guide is intended to help you quickly learn the fundamentals of Terraform and the [Azure Provider][az-rm]. After completing the tutorial, you'll know how to author a basic Terraform configuration for Azure.  

There are two different but closely related topics covered here: How to use Terraform, and how to use the Azure Provider. 

The tutorial begins with a introduction to Terraform basics. As you progress through the guide you'll modify a configuration step-by-step until you have explored all of the core parts of a Terraform configuration. You'll learn basic HCL (Hashicorp Configuration Language) syntax, and how to use modules, remote backends, outputs, and provisioners. 

Along the way we'll explore the Azure Provisioner (azurerm) for Terraform. Azurerm provides a declarative interface for managing Azure resources with Terraform. Azurerm lets you create, modify, and delete Azure resources in a Terraform configuration. The sample configuration creates a new resource group, VNET, subnet, NIC, etc, and then deploys a Linux VM.

At the end of this guide we'll list additional resources for using Terraform with Azure.

## Prerequisites

- **An Azure account.** If you don't have an Azure account, [create one now][az-create-account]. This tutorial can be completed using only the services included in an Azure [free account][az-free-faq]. You need to have at least Contributor role in the subscription to create and manage infrastructure for the tutorial. 

    **If you are using a paid subscription, you may be charged for the resources needed to complete the tutorial.** 

- **A system with Terraform 0.12.6 or later installed.** This guide includes instructions for installing Terraform on the platform of your choice.  

Azure Cloud Shell is the most convenient platform for this tutorial. Everything you need is preinstalled, and Cloud Shell uses the credentials of the signed-in Azure user. Most of the procedures assume that you are using Cloud Shell, either from the Azure Portal or standalone. Code samples are provided as-is.

<!-- links -->
[az-rm]: https://www.terraform.io/docs/providers/azurerm/index.html
[az-create-account]: https://azure.microsoft.com/en-us/free/
[az-free-faq]: https://azure.microsoft.com/en-us/free/free-account-faq/
