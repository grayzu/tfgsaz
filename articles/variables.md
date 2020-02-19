---
id: variables_az
level: Getting Started
products_used:
  - Terraform
layout: content_layout
name: 'Input Variables'
description: |-
  Parameterize your configuration with input variables.
---

You now have enough Terraform knowledge to create useful configurations, but we're still hard-coding most values. To become truly shareable and version-controlled, we need to parameterize the configurations. This lesson introduces input variables as a way to do this.

## Defining Variables

Up to now we have embedded all necessary values as literals. We're going to add a few simple variables to our configuration:

- Prefix - the string to use as a prefix on all the resources created by this configuration
- Location - the Azure region where the resources will be created
- VM Administrator user name
- VM Administrator password, which must meet Azure complexity standards
- Tags to add metadata to the resources created by this configuration


Create another file `variables.tf` with the following contents:

```hcl
variable "location" {}

variable "admin_username" {
    type = string
    description = "Administrator user name for virtual machine"
}

variable "admin_password" {
    type = string
    description = "Password must meet Azure complexity requirements"
}

variable "prefix" {
    type = string
    default = "my"
}

variable "tags" {
    type = map

    default = {
        Environment = "Terraform Getting Started"
        Dept = "DevOps"
  }
}

variable "sku" {
    default = {
        westus2 = "16.04-LTS"
        eastus = "18.04-LTS"
    }
}
```

The declarations define six new variables within your Terraform configuration, three of which are required. The _location_ has empty brackets, which tells you that the variable is required and the type of the variable will be determined by the input value. The _admin_username_ defines the type and a contextual description that is displayed when Terraform requests input. The other types are discussed below. 

## Using Variables in Configuration

Next, we need to modify the configuration to include these variables. The [complete configuration](#complete-configuration) is at the end of this page. Copy it to `main.tf`. Upload `main.tf` and `variables.tf` to your folder on Cloud Shell, overwriting the previous `main.tf`.

## Assigning Variables

There are multiple ways to assign values to variables. The following is the descending order of precedence in which variables are considered.

### Command-line flags

You can set variables directly on the command-line with the
`-var` flag. Any command in Terraform that inspects the configuration
accepts this flag, such as `apply`, `plan`, and `refresh`:

```shell
$ terraform apply \
>> -var 'prefix=tf' \
>> -var 'location=eastus'\
>> -var 'admin_username=y'\
>> -var 'admin_password=z'
```

### From a file

Terraform can populate variables using values from a file. For all files which match `terraform.tfvars` or `*.auto.tfvars` present in the
current directory, Terraform automatically loads them to populate variables. If
the file is named something else, you can use the `-var-file` flag directly to
specify a file. These files are the same syntax as Terraform
configuration files. And like Terraform configuration files, these files
can also be JSON.

To persist variable values, create a file named `terraform.tfvars` and assign variables within this file.

```hcl
location = "westus"
prefix = "tf"
admin_username = "plankton"
admin_password = "Password1234!"
```

We don't recommend saving sensitive information such as passwords, certificates, connection strings, etc. to version control. 

You can use multiple `-var-file` arguments in a single command, with some
checked in to version control and others not checked in. For example:

```shell
$ terraform apply \
  -var-file='secret.tfvars' \
  -var-file='production.tfvars'
```

### From environment variables

Terraform will read environment variables in the form of `TF_VAR_name`
to find the value for a variable. For example, the `TF_VAR_location`
variable can be set to set the `location` variable.

Environment variables are commonly used to [configure and customize Terraform](https://www.terraform.io/docs/commands/environment-variables.html). Azure Provider has its own set of [arguments and environment variables](https://www.terraform.io/docs/providers/azurerm/index.html#argument-reference).


-> **Note**: Environment variables can only populate string-type variables.
List and map type variables must be populated via one of the other mechanisms.

### UI Input

If you execute `terraform apply` with certain variables unspecified,
Terraform will ask you to input their values interactively. These
values are not saved, but this provides a convenient workflow when getting
started with Terraform. UI input is not recommended for everyday use of
Terraform.

-> **Note**: In Terraform versions 0.11 and earlier, UI input is only supported for string variables. List and map variables must be populated via one of the other mechanisms. Terraform 0.12 introduces the ability to populate complex variable types from the UI prompt.


### Variable Defaults

If no value is assigned to a variable via any of these methods and the
variable has a `default` key in its declaration, that value will be used
for the variable.

## Lists

Lists are defined either explicitly or implicitly

```hcl
# implicitly by using brackets [...]
variable "cidrs" { default = [] }

# explicitly
variable "cidrs" { type = "list" }
```

You can specify lists in a `terraform.tfvars` file:

```hcl
cidrs = [ "10.0.0.0/16", "10.1.0.0/16" ]
```

When you use a list expression as an argument, enclose the expression in square brackets:

```hcl
resource "azurerm_virtual_machine" "vm" {
    <...snip...>
    network_interface_ids = [azurerm_network_interface.nic.id]
```


## Maps

A _map_ value is a lookup table of string name = value pairs. We are going to use a map variable to specify tags for the resources we create in Azure. [Resource tags](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags) store metadata for the resource. Our `variables.tf` contains two maps, one for tags, and one for sku.

```hcl
variable "tags" {
    type = map
    default = {
        Environment = "Terraform Getting Started"
        Dept = "DevOps"
    }
}

variable "sku" {
    default = {
        westus2 = "16.04-LTS"
        eastus = "18.04-LTS"
    }
```

A variable can have a `map` type assigned explicitly, or it can be implicitly
declared as a map by specifying a default value that is a map. The above
demonstrates both.

A map is a collection of string values grouped together. When it is necessary to group different kinds of values, for example strings, bool values, and/or numbers, you will need to use an [object](https://www.terraform.io/docs/configuration/types.html#object-) type. 

Let us imagine that you have a need to vary the virtual machine sku based on the region where the vm will be created. Modify the virtual machine block as follows:

```hcl
storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = lookup(var.sku, var.location)
    version   = "latest"
}
```

This introduces a built-in function call. The
`lookup` function does a dynamic lookup in a map for a key. The
key is `var.location`, which specifies that the value of the location
variable is the key to look up the corresponding sku.

While we don't use it in our example, it is worth noting that you
can also do a static lookup of a map directly with
var.sku["westus"]. For more information, see [Indices and Attributes](https://www.terraform.io/docs/configuration/expressions.html#indices-and-attributes).

A complete list of Terrform built-in functions is [here](https://www.terraform.io/docs/configuration/interpolation.html#supported-built-in-functions).

### Assigning Maps

We set defaults above, but maps can also be set using the `-var` and
`-var-file` values. For example:

```shell
$ terraform apply -var 'tags={ Environment = "Terraform Getting Started", Dept = "DevOps" }'
# ...
```

-> **Note**: Even if every key will be assigned as input, the variable must be
established as a map by setting its default to `[]`.

# Complete configuration

The complete Terraform configuration now consists of three files: `main.tf`, `variables.tf`, and `terraform.tfvars`. The script for each file is given below.

## main.tf

```hcl
# Configure the Microsoft Azure Provider.
provider "azurerm" {
    version = ">= 1.32"
}

# Create a resource group
resource "azurerm_resource_group" "rg" {
    name     = "${var.prefix}TFRG"
    location = var.location
    tags     = var.tags
}

# Create virtual network
resource "azurerm_virtual_network" "vnet" {
    name                = "${var.prefix}TFVnet"
    address_space       = ["10.0.0.0/16"]
    location            = var.location
    resource_group_name = azurerm_resource_group.rg.name
    tags                = var.tags
}

# Create subnet
resource "azurerm_subnet" "subnet" {
    name                 = "${var.prefix}TFSubnet"
    resource_group_name  = azurerm_resource_group.rg.name
    virtual_network_name = azurerm_virtual_network.vnet.name
    address_prefix       = "10.0.1.0/24"
}

# Create public IP
resource "azurerm_public_ip" "publicip" {
    name                         = "${var.prefix}TFPublicIP"
    location                     = var.location
    resource_group_name          = azurerm_resource_group.rg.name
    allocation_method            = "Dynamic"
    tags                         = var.tags
}

# Create Network Security Group and rule
resource "azurerm_network_security_group" "nsg" {
    name                = "${var.prefix}TFNSG"
    location            = var.location
    resource_group_name = azurerm_resource_group.rg.name
    tags                = var.tags

    security_rule {
        name                       = "SSH"
        priority                   = 1001
        direction                  = "Inbound"
        access                     = "Allow"
        protocol                   = "Tcp"
        source_port_range          = "*"
        destination_port_range     = "22"
        source_address_prefix      = "*"
        destination_address_prefix = "*"
    }
}

# Create network interface
resource "azurerm_network_interface" "nic" {
    name                      = "${var.prefix}NIC"
    location                  = var.location
    resource_group_name       = azurerm_resource_group.rg.name
    network_security_group_id = azurerm_network_security_group.nsg.id
    tags                      = var.tags

    ip_configuration {
        name                          = "${var.prefix}NICConfg"
        subnet_id                     = azurerm_subnet.subnet.id
        private_ip_address_allocation  = "dynamic"
        public_ip_address_id          = azurerm_public_ip.publicip.id
    }
}

# Create a Linux virtual machine
resource "azurerm_virtual_machine" "vm" {
    name                  = "${var.prefix}TFVM"
    location              = var.location
    resource_group_name   = azurerm_resource_group.rg.name
    network_interface_ids = [azurerm_network_interface.nic.id]
    vm_size               = "Standard_DS1_v2"
    tags                  = var.tags

    storage_os_disk {
        name              = "${var.prefix}OsDisk"
        caching           = "ReadWrite"
        create_option     = "FromImage"
        managed_disk_type = "Premium_LRS"
    }

    storage_image_reference {
        publisher = "Canonical"
        offer     = "UbuntuServer"
        sku       = lookup(var.sku, var.location)
        version   = "latest"
    }

    os_profile {
        computer_name  = "${var.prefix}TFVM"
        admin_username = var.admin_username
        admin_password = var.admin_password
    }

    os_profile_linux_config {
        disable_password_authentication = false
    }

}

output "ip" {
    value = azurerm_public_ip.publicip.ip_address
}

output "os_sku" {
    value = lookup(var.sku, var.location)
}
```

## variables.tf

```hcl
variable "location" {}

variable "admin_username" {
    type = string
    description = "Administrator user name for virtual machine"
}

variable "admin_password" {
    type = string
    description = "Password must meet Azure complexity requirements"
}

variable "prefix" {
    type = string
    default = "my"
}

variable "tags" {
    type = map

    default = {
        Environment = "Terraform Getting Started"
        Dept = "DevOps"
  }
}

variable "sku" {
    default = {
        westus2 = "16.04-LTS"
        eastus = "18.04-LTS"
    }
}

```

## terraform.tfvars

```hcl
location = "westus"
prefix = "tf"
admin_username = "plankton"
admin_password = "Password1234!"
```
