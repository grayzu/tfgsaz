---
id: install_az
level: Getting Started
products_used:
  - Terraform
layout: content_layout
name: 'Installing Terraform'
content_length: 5
description: |-
  How to install Terraform
---

HashiCorp Terraform is installed by default in the [Azure Cloud Shell][az-cs]. Cloud shell can be run [standalone](https://shell.azure.com/), or as an integrated command-line terminal from the Azure [portal](https://portal.azure.com). 

We recommend that you use Cloud Shell and the Azure portal to complete this tutorial because it is the easiest way to get started using Terraform on Azure. Besides having latest release of Terraform already installed, Cloud Shell is configured to use your identity to authenticate to Azure. That simplifies the examples because it isn't necessary to include authentication credentials in the configuration. 

If you choose not to use the Azure cloud Shell, you need to install Terraform on your target system. To install Terraform on any supported system:

1. Find the appropriate [Terraform distribution package][tf-binaries] for your system and download it. Terraform is distributed as a single `.zip` file.
1. After downloading Terraform, unzip the package to a directory of your choosing. Terraform runs as a single binary named `terraform`. Any other files in the package can be safely removed and Terraform will still function.
1. Optional but highly recommended: modify the path to include the directory that contains the Terraform binary.
   - [How to set the \$PATH on Linux and Mac][lin-path]
   - [How to set the PATH on Windows][win-path]

    An alternative to modifying the path is to move the Terraform executable to a directory that is normally included in the path by default, for example:
    `sudo mv terraform /user/local/bin`

In addition to the basic installation just described, you also have the option to use a package manager to install Terraform:
 * [Chocolatey (Windows)](https://chocolatey.org/packages/terraform) 
 * [Homebrew (Mac)](https://formulae.brew.sh/formula/terraform)

## Compile from source

If you want to be certain that the `terraform` binary is from a trusted source, you can compile it yourself. This guide will not cover how to compile Terraform from source. The Terraform core and instructions are available from [HashiCorp's GitHub repository][gh-terraform].

## Verifying the installation

After installing Terraform, verify the installation by opening a new
terminal session and checking that Terraform is available. Execute `terraform` at the prompt, and you should see output similar to this (truncated here for brevity):

```text
$ terraform
Usage: terraform [-version] [-help] <command> [args]

The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you're just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.

Common commands:
    apply              Builds or changes infrastructure
    console            Interactive console for Terraform interpolations
    destroy            Destroy Terraform-managed infrastructure
...

```

If you get an error that `terraform` could not be found, your `PATH` environment
variable was not set up properly. Please go back and ensure that your `PATH`
variable contains the directory where the Terraform binary was installed.

<!-- links -->

[az-cs]: https://docs.microsoft.com/en-us/azure/cloud-shell/overview
[tf-binaries]: https://www.terraform.io/downloads.html
[tf-vmtemplate]: https://azuremarketplace.microsoft.com/en-us/marketplace/apps/azure-oss.terraform?tab=Overview
[tf-docs]: https://www.terraform.io/docs/index.html
[lin-path]: https://stackoverflow.com/questions/14637979/how-to-permanently-set-path-on-linux
[win-path]: https://stackoverflow.com/questions/1618280/where-can-i-set-path-to-make-exe-on-windows
[gh-terraform]: https://github.com/hashicorp/terraform
