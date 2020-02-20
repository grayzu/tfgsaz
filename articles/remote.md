---
id: remote_az
level: Beginner
products_used:
  - Terraform
layout: content_layout
name: 'Remote State Storage'
description: |-
  A brief introduction to remote backends for remote state management.
---

Terraform [state](https://www.terraform.io/docs/state/index.html) includes the settings for all of the resources maintained in the configuration. Previous lessons have mentioned state, now we'll get into the details.

It is helpful to understand how configuration files, execution plan, and state are related: 
* Configuration files define the _desired state_ of your managed infrastructure.
* An execution plan is a map of actions Terraform will perform so that the actual state matches the desired state in the configuration files. 
* State is a map of the _actual state_ of Azure infrastructure configuration, along with metadata that Terraform uses to track changes. 

By default, Terraform saves state in the project working directory in the `terraform.tfstate` file. Terraform manages state on local filesystem using the [local](https://www.terraform.io/docs/backends/types/local.html) backend. 

Managing state on the local filesystem works well for development and testing, as part of the "inner loop" development cycle. Although it works well on a single workstation, local state doesn't scale easily to distributed workflows. 

State presents a special challenge because team workflows often require access to state at more than one stage, and by more than one entity. A long-running CD pipeline may persist for weeks and involve multiple automation hosts and/or human collaborators. Production configurations change gradually over long periods of time. State has to be stored for the entire life cycle of the managed infrastructure.

Terraform state has to be protected just like a security credential, because in many cases it contains [sensitive data](https://www.terraform.io/docs/state/sensitive-data.html) in plain text. Passwords, connection strings, details of network security rules, etc. -- information you probably don't want to share with _everyone_. Users and automation hosts with access to state also have access to all of the secrets contained in the state files. 

Terraform supports team-based workflows with a feature known as [remote
backends](https://www.terraform.io/docs/backends/index.html). Remote backends allow Terraform to use a shared storage space for state data. A remote backend can solve many of the problems described above. It is strongly recommended to configure a remote backend for state. 

You have a choice of remote backend providers to match your needs:

* [Azurerm](#azurerm) backend stores remote state in Azure Storage. Azurerm provides a standard remote backend that is entirely native to the Azure platform.
* [Terraform Cloud](#terraform-cloud) is HashiCorp's commercial SaaS solution that includes a remote backend. Terraform Cloud combines a predictable and reliable shared run environment with tools to help you work together on Terraform configurations and modules. 
* Other providers, including Terraform Enterprise and Consul; for details, see [Backend Types](https://www.terraform.io/docs/backends/types/index.html).


## Azurerm

[Azurerm](https://www.terraform.io/docs/backends/types/azurerm.html) backend stores remote state on [Azure Blob storage](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction) and supports all of the [state storage and locking](https://www.terraform.io/docs/backends/state.html) features of a standard backend. Blob storage provides [RBAC](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac-cli?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) security, declarative access policy, data encryption, low cost, and high availability on the Azure platform.

Managed identities for Azure resources is an Azure AD feature that allows Terraform to [access storage without a separate credential](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-msi?toc=%2fazure%2fstorage%2fblobs%2ftoc.json), such as an access key or SAS token. Terraform needs to be running on an Azure VM or container with a managed identity, and the storage account and Blob container need to be configured for access. For more information, see [Authenticating using managed identities for Azure resources](https://www.terraform.io/docs/providers/azurerm/auth/managed_service_identity.html). 

Here are a few tips for working with remote state:

* You have to enable remote state on every Terraform project. You can use the **[TF_CLI_ARGS_name](https://www.terraform.io/docs/commands/environment-variables.html#tf_cli_args-and-tf_cli_args_name)** to add configuration details to `terraform init`. 

    ```bash
    export TF_CLI_ARGS_init="-backend-config=<partial_config>"
    ```
    `<partial_config>` is replaced with the path and file to a [partial configuration](https://www.terraform.io/docs/backends/config.html#partial-configuration). Whenever `terraform init` is run, the command expands to automatically configure a remote backend.

* You can override the default environment setting and control backend initialization as described in [terraform init](https://www.terraform.io/docs/commands/init.html#backend-initialization). 
* A note about the `key=` field in the provider block: nearly every azurerm example everywhere shows **prod.terraform.tfstate**, but there is nothing magic about that value. The key can be any valid Azure Blob name.


## Terraform Cloud

[Terraform Cloud](https://www.hashicorp.com/products/terraform/?utm_source=oss&utm_medium=getting-started&utm_campaign=terraform) includes a remote backend, and tools that allow teams to easily version, audit, and collaborate
on infrastructure changes. Each proposed change generates
a Terraform plan which can be reviewed and collaborated on as a team.
When a proposed change is accepted, the Terraform logs are stored,
resulting in a linear history of infrastructure states to
help with auditing and policy enforcement. Additional benefits to
running Terraform remotely include moving access
credentials off of developer machines and freeing local machines
from long-running Terraform processes.

Although Terraform Cloud can act as a standard remote backend to support Terraform runs on local machines, it works even better as a remote run environment. It supports two main workflows for performing Terraform runs:

- A VCS-driven workflow, in which it automatically queues plans whenever changes are committed to your configuration's VCS repo.
- An API-driven workflow, in which a CI pipeline or other automated tool can upload configurations directly.

For a hands-on introduction to Terraform Cloud, [follow the Terraform Cloud getting started guide](https://learn.hashicorp.com/terraform/cloud-gettingstarted/tfc_overview).


### How to Store State Remotely in Terraform Cloud

First, we'll use Terraform Cloud as our backend. Terraform Cloud
offers free remote state management. Terraform Cloud is the recommended best practice for remote state storage.

If you don't have an account, please [sign up here](https://app.terraform.io/signup) for this guide.
For more information on Terraform Cloud, [view our getting started guide](https://learn.hashicorp.com/terraform/cloud-gettingstarted/tfc_overview)
First, configure the backend in your configuration with your own organization and workspace names:

```hcl
terraform {
  backend "remote" {
    organization = "Cloud-Org"

    workspaces {
      name = "Dev-QA"
    }
  }
}
```

The `backend` section configures the backend you want to use. After
configuring a backend, run `terraform init` to setup Terraform. It should
ask if you want to migrate your state to Terraform Cloud. Say "yes" and Terraform
will copy your state.

Now, if you run `terraform apply`, Terraform should state that there are
no changes:

```shell
$ terraform apply
# ...

No changes. Infrastructure is up-to-date.

This means that Terraform did not detect any differences between your
configuration and real physical resources that exist. As a result, Terraform
doesn't need to do anything.
```

Terraform is now storing your state remotely in Terraform Cloud. Remote state
storage makes collaboration easier and keeps state and secret information
off your local disk. Remote state is loaded only in memory when it is used.

If you want to move back to local state, you can remove the backend configuration
block from your configuration and run `terraform init` again. Terraform will
once again ask if you want to migrate your state back to local.
