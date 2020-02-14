---
id: change_az
level: Getting Started
products_used:
  - Terraform
layout: content_layout
name: 'Change Infrastructure'
content_length: 7
description: |-
  Modify existing resources.
---

In the previous page, you created your first infrastructure with
Terraform: a resource group. In this lesson, you'll modify that resource and see how Terraform handles change.

Infrastructure is continuously evolving, and Terraform was built
to help manage and enact that change. As you change Terraform
configurations, Terraform builds an execution plan that only
modifies what is necessary to reach your desired state. Terraform builds an execution plan by comparing your desired state as described in the configuration to the current state, saved in the `terraform.tfstate` file or in a remote state backend.

## Configuration

Let's modify the resource group of our instance by adding metadata (tags).

In Cloud Shell, in the directory that contains the configuration file, type
`code main.tf`. This will open the configuration file in the [Cloud Shell editor](https://docs.microsoft.com/en-us/azure/cloud-shell/using-cloud-shell-editor).

Edit the `azurerm_resource_group` resource in your configuration and add the tags block as shown below:

```hcl
resource "azurerm_resource_group" "rg" {
    name     = "myTFResourceGroup"
    location = "westus"

    tags = {
        Environment = "Terraform Getting Started"
        Team = "DevOps"   
    }
}
```

Use the **...** menu on the top right of the editor window to save your changes and close the editor.

Unlike the change that you just made to the resource group by adding a tag, some changes will force Terraform to destroy and then recreate the resource. For example, changing the name or location of a resource group will force Terraform to recreate the resource. Recreating the resource group will force the destruction of _all_ the resources in the group. Use caution when making changes that force Terraform to destroy and recreate a resource. 

## Plan

When a configuration is changed, the execution plan shows what actions Terraform will take to effect the change. Let's update the plan, and this time, save the plan to apply in the next step:

`terraform plan -out=newplan`

The **-out** argument tells Terraform to save the plan in a file named `newplan` in the current directory.

```shell
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # azurerm_resource_group.rg will be updated in-place
  ~ resource "azurerm_resource_group" "rg" {
        id       = "/subscriptions/<subscription-id>/resourceGroups/myTFResourceGroup"
        location = "westus"
        name     = "myTFResourceGroup"
      ~ tags     = {
          + "Environment" = "Terraform Getting Started"
          + "Team" = "DevOps" 
        }
    }

Plan: 0 to add, 1 to change, 0 to destroy.

------------------------------------------------------------------------

This plan was saved to: newplan

To perform exactly these actions, run the following command to apply:
    terraform apply "newplan"

```

Notice the symbols that precede the resource and arguments: **~** indicates an update in-place change for the resource and the tags argument. The **+** next to "environment" indicates that the environment tag will be created. 


## Apply Changes

After changing the configuration, run `terraform apply "newplan"` to apply the changes:

```shell
$ terraform apply "newplan"
azurerm_resource_group.rg: Modifying... [id=/subscriptions/<subscription id>/resourceGroups/myTFResourceGroup]
azurerm_resource_group.rg: Modifications complete after 4s [id=/subscriptions/<subscription id>/resourceGroups/myTFResourceGroup]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.

The state of your infrastructure has been saved to the path
below. This state is required to modify and destroy your
infrastructure, so keep it safe. To inspect the complete state
use the `terraform show` command.

State path: terraform.tfstate

```

You can use `terraform show` again to see the new values associated with this resource group:

```shell
$ terraform show
# azurerm_resource_group.rg:
resource "azurerm_resource_group" "rg" {
    id       = "/subscriptions/<subscription-id>/resourceGroups/myTFResourceGroup"
    location = "westus"
    name     = "myTFResourceGroup"
    tags     = {
        "Environment" = "Terraform Getting Started"
        "Team"        = "DevOps"
    }
}
```
