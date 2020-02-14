---
id: destroy_az
level: Getting Started
products_used:
  - Terraform
layout: content_layout
name: 'Destroy Infrastructure'
content_length: 5
description: |-
  How to completely destroy the Terraform-managed infrastructure.
---

You have now seen how to build and change infrastructure. Before you
move on to creating multiple resources and showing resource
dependencies, you should know how to completely destroy
the Terraform-managed infrastructure.

## Destroy

Resources can be destroyed using the [**terraform destroy**](https://www.terraform.io/docs/commands/destroy.html) command. You'll use `terraform destroy` in this tutorial to remove infrastructure between lessons, and when you're finished with the guide.

In Cloud Shell, navigate to the directory that contains the configuration used in the previous lessons. 

First, let's look at the actions Terraform will take to destroy the infrastructure. Run the following command to see the execution plan for destroying the infrastructure:

`terraform plan -destroy` 

```shell
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # azurerm_resource_group.rg will be destroyed
  - resource "azurerm_resource_group" "rg" {
      - id       = "/subscriptions/c5a42fec-6a86-43df-866b-7a6845daaa98/resourceGroups/myTFResourceGroup" -> null
      - location = "westus" -> null
      - name     = "myTFResourceGroup" -> null
      - tags     = {
          - "Environment" = "Terraform Getting Started"
          - "Team"        = "DevOps"
        } -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.
```

Just like with `apply`, Terraform determines the order in which
things must be destroyed. In this case there was only one resource, so no
ordering was necessary. In more complicated cases with multiple resources,
Terraform will destroy them in a suitable order to respect dependencies,
as you'll see later in this guide.

Now run `terraform destroy` to remove the resource group.
