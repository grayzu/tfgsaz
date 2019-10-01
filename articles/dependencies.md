---
id: dependencies_az
level: Getting Started
products_used:
  - Terraform
layout: content_layout
name: 'Resource Dependencies'
content_length: 20
description: |-
  Understanding and working with multiple resources and dependencies.
---
<!-- 08/06/19, review for v0.12 -->

In this lesson we are going to introduce resource dependencies. Up to this point, the example configuration has only contained a single resource. Real infrastructure is a diverse collection of interdependent resources. Terraform configurations usually contain many resources.

When Terraform changes infrastructure, many of the changes have to be made in a specific order. This order is determined by resource dependencies. There are two types of dependencies:

* Implicit dependencies, which Terraform and the Azure provider determine automatically for you based on the configuration. For example, you can't create a new virtual machine without first having a network interface (NIC). Azurerm "knows" to create the NIC before attempting to create the VM.

* Explicit dependencies, which you define using the `depends_on` meta-argument. Explicit dependencies are used when Terraform can't "see" an implicit dependency, and when you want to override the default execution plan.

These dependency types are discussed and illustrated in more detail below. 

## Configuration

The configuration for this lesson is a basic example of multiple, interdependent resources needed to deploy a Linux virtual machine. Before a virtual machine can be deployed on Azure, several other resources need to be created (or already present). The following are the minimum set of resources that need to exist before a VM can be be created:

* Resource group
* Virtual network
* Subnet
* Network security group
* Network interface

In addition to this minimal set of resources, the sample configuration includes a public IP resource to make it easier to access the VM, and a network security rule that opens port 22 for ssh.

We'll start by adding a [azurerm_virtual_network][az-vnet] resource to our existing configuration. Add the following new resource to the `main.tf` file that you create in the cloud console:

```hcl
# Create a virtual network
resource "azurerm_virtual_network" "vnet" {
    name                = "myTFVnet"
    address_space       = ["10.0.0.0/16"]
    location            = "eastus"
    resource_group_name = azurerm_resource_group.rg.name
}
```
The azurerm_virtual_network block introduces a few new features: expressions, input variables (identified by the **var.** prefix), and lists. They are described briefly here, and again in more detail in [Input Variables](./variables.md).


### Expressions

To create a new Azure VNet, you have to specify the name of the resource group to contain the vnet. Notice that the value of `resource_group_name` is an [expression](https://www.terraform.io/docs/configuration/expressions.html), `azurerm_resource_group.rg.name`. This expression returns the Azure name of the resource group that is referenced by the terraform `azurerm_resource_group` resource with the name `rg`. In our example this will resolve to "_myTFResourceGroup_". The general syntax of expressions is TYPE.NAME.ATTRIBUTE. Expressions are used throughout the configuration to obtain values from other resources. 

Previous versions of Terraform required expressions only within interpolation syntax `${ ... }`. Beginning with Terraform 0.12, expressions can be used directly as shown in the example. 

The [interpolation](https://www.terraform.io/docs/configuration/expressions.html#interpolation) syntax isn't gone, however, it's still important for working with [string templates](https://www.terraform.io/docs/configuration/expressions.html#string-templates). The following code fragment shows how an expression is used with interpolation syntax to assemble a command line string from literals, variables, and resource attributes:

```hcl
provisioner "local-exec" {
        command = "ssh-keygen -f ${var.file-path}${var.vm-name} -t rsa -b 4096 -N ${random_string.passphrase.result}"
}
```

The sample above generates an ssh key pair with a password-protected private key, on the local Terraform host system. [Provisioners](https://www.terraform.io/docs/provisioners/index.html) will be discussed in more detail [later in the guide](./provision.md).

### Lists

Next, look at the value for `address_space`. The address is surrounded by square brackets []. Square brackets define lists. The list is a sequence of comma-separated values. When you see square brackets around an argument, it tells you that the argument accepts more than one value. In this example, we have specified one address space, but you can supply more than one.

## Implicit dependencies

Terraform can infer when one resource depends on another by analyzing the resource attributes used in interpolation expressions. The act of referencing an attribute from another resource in the argument value of a resource results in an implicit dependency. From this Terraform builds a dependency tree to determine the correct order in which to create the each resource. Use implicit dependencies wherever possible.

In the following example of an implicit dependency, Terraform knows that the `azurerm_resource_group` has to be created before the `azurerm_virtual_network` because the virtual network references the resource group in its resource_group_name argument. The expression `azurerm_resource_group.rg.name` creates the implicit dependency on the `azurerm_resource_group` object named `rg`.

```hcl
# Create a resource group
resource "azurerm_resource_group" "rg" {
    name     = "myTFResourceGroup"
    location = "westus2"
}
# Create virtual network
resource "azurerm_virtual_network" "vnet" {
    name                = "myTFVnet"
    address_space       = ["10.0.0.0/16"]
    location            = "westus2"
    resource_group_name = azurerm_resource_group.rg.name
}
```

Because of the dependency, the order of blocks and expressions in the configuration file doesn't matter. This example works just like the one above:

```hcl
# Create virtual network
resource "azurerm_virtual_network" "vnet" {
    name                = "myTFVnet"
    address_space       = ["10.0.0.0/16"]
    location            = "westus2"
    resource_group_name = azurerm_resource_group.rg.name
}

# ...more blocks here...

# Create a resource group
resource "azurerm_resource_group" "rg" {
    name     = "myTFResourceGroup"
    location = "westus2"
}
```

## Explicit dependencies

Sometimes there are dependencies between resources that are not *visible* to Terraform. In these scenarios, use the **[depends_on](https://www.terraform.io/docs/configuration/resources.html#depends_on-explicit-resource-dependencies)** [meta-argument](https://www.terraform.io/docs/configuration/resources.html#meta-arguments) to explicitly define these dependencies. The `depends_on` meta-argument accepts a list of resources. 

Explicit dependencies:
* Control of the order in which resources are created 
* Can be used to create parallel flows in the configuration 
* May be needed when a configuration depends on processes outside the domain of Terraform and Azure provider

The following code snippet uses the **[Local](https://www.terraform.io/docs/providers/local/index.html)**, **[Null](https://www.terraform.io/docs/providers/null/index.html)**, and **[Random](https://www.terraform.io/docs/providers/random/index.html)** providers. In this example, Terraform has no way of knowing that the local_file data resources depend on the result of the **[local-exec](https://www.terraform.io/docs/provisioners/local-exec.html)** provisioner. (We will discuss provisioners in a later lesson.)

By default, local-exec signals success in _starting_ the host process. Using `depends_on` causes local-exec to wait until the host process is _complete_. The same pattern applies to Azure resources. When a resource is included in the `depends_on = [<resource list...>]` list, Terraform will wait for the listed resource to be provisioned before creating the resource containing the meta-argument. 

```hcl
# get a passphrase that contains alpha, numeric, and some special characters, but no spaces
resource "random_string" "passphrase" {
    length = 12
    special = true
    override_special = "!@#$%&*-_=?"
    number = true  
}

# create a new ssh key pair, with a password on the private key
resource "null_resource" "sshkg" {
    # generate the key pair
    provisioner "local-exec" {
        command = "ssh-keygen -f ${var.file-path}${var.vm-name} -t rsa -b 4096 -N ${random_string.passphrase.result}"
    }
}

# get the ssh keys
data "local_file" "sshpk" {
    # private key
    depends_on = [null_resource.sshkg]  
    filename = "${var.file-path}${var.vm-name}"
}
data "local_file" "sshpub" {
    # public key
    depends_on = [null_resource.sshkg]
    filename = "${var.file-path}${var.vm-name}.pub"
}

# Write ssh keys and passphrase to key vault, and then provision VM with public key
...
```

Non-Dependent Resources

Configurations sometimes contain resources that do not depend on any other resource. For example, if you were to add another azurerm_virtual_machine block to create a second VM, there would be no dependency between the first and second VMs. Because there is no dependency between the VMs, they can be created in parallel. 

Whenever possible, Terraform will perform operations concurrently. By default, Terraform will process up to 10 nodes concurrently while [walking the graph](https://www.terraform.io/docs/internals/graph.html#walking-the-graph). 



[COMMENT]: # ( Add an transition paragraph here saying sometimelike "let's jump into our example configuration now.  )

[COMMENT]: # ( Please include the ssh stuff into this example to combine the implicit and explicit dependencies. Also be sure to update the configuration description section to include ssh stuff as well.)

## Plan



## Apply

The [complete configuration](#complete-configuration) is at the bottom of this page. Copy it to a file named `main.tf`, and then upload it to your testing directory in Cloud Shell. The example assumes the directory is named `tftest`.

1. If necessary, modify values in main.tf for your environment. For example, you'll probably want to change the region from uswest2 to the region closest to you. If you're using a free Azure account, you might need to change the instance size to one that is covered under the free tier.

2. Run `terraform init` to initialize the working directory. Terraform will download plugins.

3. Run `terraform apply` to see how Terraform plans to apply this change. 

    The output will look similar to the following (truncated to save space):

    ```shell
    $ terraform apply

    An execution plan has been generated and is shown below.
    Resource actions are indicated with the following symbols:
    + create

    Terraform will perform the following actions:

    # azurerm_network_interface.nic will be created
        < ... snip ...>

    Plan: 7 to add, 0 to change, 0 to destroy.

    Do you want to perform these actions?
    Terraform will perform the actions described above.
    Only 'yes' will be accepted to approve.

    Enter a value:    
    ```

Terraform will create seven resources. As usual, Terraform prompts for confirmation before making any changes. Answer `yes` to apply. It will take several minutes to complete; it takes some time to create and deploy the virtual machine. The continued output will look similar to the
following:

```shell
 Enter a value: yes

azurerm_resource_group.rg: Creating...
azurerm_resource_group.rg: Creation complete after 1s [id=/subscriptions/ 
    <... snip ...>

azurerm_virtual_machine.vm: Creation complete after 1m31s [id=/subscriptions/159f2485-...7/resourceGroups/myTFResourceGroup/providers/Microsoft.Compute/virtualMachines/myTFVM]
```

[COMMENT]: # ( The above part should go away when you add the plan section. )

Congratulations, you have just deployed an Azure virtual machine using infrastructure as code!

---

**NOTE**
Do not forget to stop the virtual machine or use `terraform destroy` to remove it when you are done with this lesson, to avoid being charged for the instance while you're not using it. Recall that Terraform relies on state to know what to destroy, so if you plan to come back to this later you'll want to preserve the entire contents of your working directory until you are ready to destroy this infrastructure.

---



## Complete configuration

```hcl



```

<!-- links -->

[az-auth]: https://www.terraform.io/docs/providers/azurerm/auth/azure_cli.html
[az-vnet]: https://www.terraform.io/docs/providers/azurerm/r/virtual_network.html
[tf-interpolation]: https://www.terraform.io/docs/configuration/interpolation.html
