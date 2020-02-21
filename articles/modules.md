---
id: modules_az
level: Getting Started
products_used:
  - Terraform
layout: content_layout
name: 'Modules'
description: |-
  Refactor your existing configuration into a module for reusability.
---

Up to this point, we've been configuring Terraform by editing Terraform
configurations directly. Building configurations one at a time works well for learning and ad hoc testing, but it does not scale. As your infrastructure grows, demand for services will quickly overwhelm the development team and create a bottleneck. Lack of consistency and reusability will lead to management problems, and complicate troubleshooting. For these reasons, it is desirable to have a way to encapsulate common configuration elements for reuse, similar to an API.

Terraform [**modules**](https://www.terraform.io/docs/modules/index.html) are self-contained packages of Terraform configurations that are managed as a group. Modules are used to create reusable components, improve organization, and to treat pieces of infrastructure as a single entity. This section will cover the basics of using modules.

## Using Modules

If you have any instances running from prior steps in the getting
started guide, use `terraform destroy` to destroy them, and then remove all
configuration files.

The [Terraform Registry](https://registry.terraform.io/search?q=azure) includes a directory of ready-to-use Azure RM modules for various common purposes, which can serve as larger building-blocks for your infrastructure.

In this example, we're going to use two modules:

- [Azure RM Network Module](https://registry.terraform.io/modules/Azure/network/azurerm/2.0.0) to create a vnet and subnet.
- [Azure RM Compute Module](https://registry.terraform.io/modules/Azure/compute/azurerm/1.2.1) to create a Linux vm.

Create a configuration file with the following contents:

```hcl
# declare variables and defaults

variable "vm_name" {
  description = "VM name, up to 15 characters, numbers and letters, no special characters except hyphen -"
}

variable "admin_username"{
  description = "Admin user name for the virtual machine"
}

variable "location" {
  description = "Azure region"
}

variable "environment" {
  default = "dev"
}
variable "vm_size" {
  default = {
    "dev"  = "Standard_B2s"
    "prod" = "Standard_D2s_v3"
  }
}
# end vars

# Create a resource group
resource "azurerm_resource_group" "rg" {
  name     = "myTFModuleGroup"
  location = var.location
}

# Use the network module to create a vnet and subnet
module "network" {
  source              = "Azure/network/azurerm"
  version             = "~> 2.0"
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
  address_space       = "10.0.0.0/16"
  subnet_names        = ["mySubnet"]
  subnet_prefixes     = ["10.0.1.0/24"]
}

# Use the compute module to create the VM
module "compute" {
  source         = "Azure/compute/azurerm"
  version        = "~> 1.3"
  location       = var.location
  resource_group_name = azurerm_resource_group.rg.name
  vm_hostname    = var.vm_name
  vnet_subnet_id = element(module.network.vnet_subnets, 0)
  admin_username = var.admin_username
  remote_port    = "22"
  vm_os_simple   = "UbuntuServer"
  vm_size        = var.vm_size[var.environment]
  public_ip_dns  = ["zzdns123"]  #update this with a unique address if needed
}
```

The `module` block begins with the example given on the Terraform Registry
page for this module, telling Terraform to create and manage this module.
This is similar to a `resource` block: it has a name used within this
configuration and a set of input values that are listed in the documentation for compute module.

The `source` attribute is the only mandatory argument for modules. It tells
Terraform where the module can be retrieved. Terraform automatically
downloads and manages modules for you. Additionally, most modules will have at least a few required arguments.

In this instance, the modules are retrieved from the official Terraform Registry. Terraform can also retrieve modules from a variety of sources, including private module registries or directly from Git, Mercurial, HTTP, and local files.

The other attributes shown are inputs to our modules. The compute module supports many additional inputs, but most are optional and have reasonable values for experimentation.

After adding a new module to configuration, it is necessary to run (or re-run)
`terraform init` to obtain and install the new module's source code:

```shell
$ terraform init
Initializing modules...
Downloading Azure/compute/azurerm 1.3.0 for compute...
- compute in .terraform/modules/compute/Azure-terraform-azurerm-compute-fb014dd
- compute.os in .terraform/modules/compute/Azure-terraform-azurerm-compute-fb014dd/os
Downloading Azure/network/azurerm 2.0.0 for network...
- network in .terraform/modules/network/Azure-terraform-azurerm-network-564155f

Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "azurerm" (terraform-providers/azurerm) 1.32.1...
- Downloading plugin for provider "random" (terraform-providers/random) 2.2.0...

Terraform has been successfully initialized!
```

By default, this command does not check for new module versions that may be
available, so it is safe to run multiple times. The `-upgrade` option will
additionally check for any newer versions of existing modules and providers
that may be available.

[COMMENT]: # ( Please add plan here instead of directly runnly apply. )

## Apply Changes

With the network and compute modules and their dependencies (if any) installed, you can now apply these changes to create the virtual machine.

If you run `terraform apply`, you will see a list of all of the
resources encapsulated in the module. The output is similar to what we
saw when using resources directly, but the resource names now have
module paths prefixed to their names, like in the following example:

```shell
+ module.compute.azurerm_virtual_machine.vm-linux
      id:                                <computed>
      availability_set_id:               "${azurerm_availability_set.vm.id}"
      boot_diagnostics.#:                "1"
      boot_diagnostics.0.enabled:        "false"
      delete_data_disks_on_termination:  "false"
      delete_os_disk_on_termination:     "false"
      identity.#:                        <computed>

```

The `module.compute.azurerm_virtual_machine` prefix shown above indicates
that the resource is from the `module "compute"` block we wrote,
and that this module has its own `azurerm_virtual_machine` block
within it. Modules can be nested to decompose complex systems into
manageable components.

To proceed with the creation of the virtual machine, type `yes` at the confirmation prompt.

```shell

module.compute.azurerm_network_interface.vm: Creation complete after 1s ...
module.compute.azurerm_virtual_machine.vm-linux: Creating...
  availability_set_id:                                              "" => "/subscriptions/.../resourcegroups/terraform-compute/providers/microsoft.compute/availabilitysets/myvm-avset"
  boot_diagnostics.#:                                               "" => "1"
  boot_diagnostics.0.enabled:                                       "" => "false"
  delete_data_disks_on_termination:                                 "" => "false"
  delete_os_disk_on_termination:                                    "" => "false"6"
    <...snip...>

Plan: 11 to add, 0 to change, 0 to destroy.
```

After several minutes and many log messages about all of the resources
being created, you'll have a virtual machine up and running. These two modules encapsulate all of the resources we used in our previous configuration, plus they have the capability to create much more complex infrastructure than our simple example of creating one vm.

-> **Note**: You may need to generate an SSH key. You can follow instructions to do so [here.](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

## Module Outputs

Most modules also output key information about status, the names and IDs of resources created, etc. Unlike the output variables we discussed earlier, module outputs are not automatically echoed to the command line output. Module outputs are referenced as inputs to other modules or resources, and you can echo outputs to the UI or to a file using output variables in the configuration or the `local-exec` provisioner.

### Referencing module output

The syntax for referencing module outputs is `${module.NAME.OUTPUT}`, where
`NAME` is the module name given in the header of the `module` configuration
block and `OUTPUT` is the name of the output to reference.

```hcl
module "compute" {
    source            = "Azure/compute/azurerm"
    version           = "~> 1.3"
    location          = var.location
    vnet_subnet_id    = element(module.network.vnet_subnets, 0)
    <... snip ...>
```

In the example configuration above, the compute module references an output of the network module: `module.network.vnet_subnets`. The `vnet_subnets` outputs a list of subnet ids. We used the built-in `element` function to return the value at index 0 (first item) in the list. This gives us the subnet id where the virtual machine will be created.

To send module outputs to the UI or a file, add the following lines to the sample configuration at the end, after the `compute` block:

```hcl
output "public_ip" {
    value = module.compute.public_ip_address
}
```

```shell
# ...
Apply complete! Resources: 11 added, 0 changed, 0 destroyed.

Outputs:

public_ip = [40.118.146.187]
```

## Destroy

Just as with top-level resources, you can destroy the resources created by
the our configurations:

```shell
$ terraform destroy

Terraform will perform the following actions:

...

Plan: 0 to add, 0 to change, 11 to destroy.

Do you really want to destroy all resources?
```

As usual, Terraform describes all of the actions it will take. In this example,
Terraform will destroy all of the resources that were created by the module.
Type `yes` to confirm and, after a few minutes and even more log output,
all of the resources should be destroyed:

```shell
Destroy complete! Resources: 11 destroyed.
```

With all of the resources destroyed, you can delete the configuration file
you created above.
