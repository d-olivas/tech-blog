---
title: "How to Sync Terraform State with Existing Azure Infrastructure Using the Terraform Import Command"
date: 2021-04-15T10:00:00-06:00
draft: false
tags: ["azure", "terraform", "devops", "state import"]
categories: ["Azure", "DevOps", "Terraform"]
description: "A detailed guide with two real-life examples."
cover:
  image: cover.jpg
  alt: alt_text
  relative: true
  caption: "[Image](https://unsplash.com/photos/time-lapse-photo-of-concrete-highway-with-cars-vdBE638sszE) from [Unsplash](https://unsplash.com/) by [@shawnanggg](https://unsplash.com/@shawnanggg)."
showToc: true # show Table of Contents
---

## Introduction

If you are reading this post, you probably are in a situation where you have to use Terraform to manage your cloud infrastructure. But there are existing resources, not created using Terraform, that need to be managed by it.

I will show you how to sync your Terraform state with existing resources in Azure specifically. However, the steps to achieve this are virtually the same for other cloud providers.

To sync our Terraform state we’ll be using the `terraform import` command. This command allows us to bring under Terraform management resources that already exist or were created by other means. There is one important thing to know about the usage of this command:

> The current implementation of Terraform import can only import resources into the [state](https://www.terraform.io/docs/language/state/index.html). It does not generate configuration. A future version of Terraform will also generate configuration.
>
> Because of this, prior to running `terraform import` it is necessary to write manually a `resource` configuration block for the resource, to which the imported object will be mapped.
>
> — Terraform’s Documentation

To implement `terraform import` we have to add the code in our Terraform file for the resource we want to import, then we can run the command. Here we have a simple example of the command:

```sh
$ terraform import azurerm_resource_group.default <fully qualified resource group id>
```

In that example, we are importing a resource group into our Terraform state. First, we specify the resource name that the Terraform Azure provider uses for resource groups, followed by the name that we’ll use in Terraform to identify the resource group. In this case, we called it default. Finally, we must include the fully qualified resource group id, the one that looks like this “/subscriptions/\<subscription id\>/resourceGroup/\<resource group name\>”.

## Examples

I’ll show you two examples of how to import a resource into our Terraform state. One example will be a virtual machine, and the other one a key vault with a secret.

### Virtual Machine

Remember that before executing the import command, we have to create the code for the resources we need. For this example, we will need a resource group, a virtual network with a subnet, a network security group, a network interface, and the virtual machine. We’ll assume that the resource group, virtual network, and subnet already exist in the Terraform state, so we don’t need to import those resources.

Our Terraform file to create the resources we need looks like this:

```hcl
# azure_vm.tf
resource "azurerm_network_security_group" "nsg_ssh_allow" {
  name                = "nsg-allow-ssh"
  location            = azurerm_resource_group.default.location
  resource_group_name = azurerm_resource_group.default.name

  security_rule {
    name                       = "SSH"
    priority                   = 300
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface" "nic" {
  name                = "nic-test-001"
  location            = azurerm_resource_group.default.location
  resource_group_name = azurerm_resource_group.default.name
  
  ip_configuration {
    name                          = "ipconfig1"
    subnet_id                     = azurerm_subnet.default.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_network_interface_security_group_association" "nsg_nic_association" {
  network_interface_id      = azurerm_network_interface.nic.id
  network_security_group_id = azurerm_network_security_group.nsg_ssh_allow.id
}

resource "azurerm_virtual_machine" "test_vm" {
  name                  = "vm-test"
  location              = azurerm_resource_group.default.location
  resource_group_name   = azurerm_resource_group.default.name
  network_interface_ids = [azurerm_network_interface.nic.id]
  vm_size               = "Standard_B1ms"

  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  storage_os_disk {
    name              = "my-test-os-disk"
    caching           = "ReadWrite"
    create_option     = "FromImage"
  }

  os_profile {
    computer_name  = "test"
    admin_username = "test-user"
    admin_password = "test-password123"
  }

  os_profile_linux_config {
    disable_password_authentication = false
  }
}
```

Now that we have the resources defined in our file, we can import them to our state. We will skip the initialization of Terraform, and we’ll see how to use the import command.

Let’s import the resources in the order they are in the file. Verify that the resource names are the same in the file as they are in the existing resource.

Network Security Group

```sh
$ terraform import azurerm_network_security_group.nsg_ssh_allow /subscription/<subscription id>/resourceGroup/<resource group name>/providers/Microsoft.Network/networkSecurityGroups/nsg-allow-ssh
```

Network Interface

```sh
$ terraform import azurerm_network_interface.nic /subscription/<subscription id>/resourceGroup/<resource group name>/providers/Microsoft.Network/networkInterfaces/nic-test-001
```

Network Interface and Network Security Group Association

```sh
$ terraform import azurerm_network_interface_security_group_association.nsg_nic_association '/subscription/<subscription id>/resourceGroup/<resource group name>/providers/Microsoft.Network/networkInterfaces/nic-test-001|/subscription/<subscription id>/resourceGroup/<resource group name>/providers/Microsoft.Network/networkSecurityGroups/nsg-allow-ssh'
```

Notice how the ids are separated by “|” and are enclosed by single quotes.

Virtual Machine

```sh
$ terraform import azurerm_virtual_machine.test_vm /subscription/<subscription id>/resourceGroup/<resource group name>/providers/Microsoft.Compute/virtualMachines/vm-test
```

Now that we have imported all the resources, we can run the `terraform show` command to check that they are in the Terraform state.

### Key Vault

Now, let’s see another example. This time we’ll create a key vault with an access policy and one secret. The Terraform code looks like this:

```hcl
# azure_keyvault.tf
resource "azurerm_key_vault" "main" {
  name                       = "kv-myvault-001"
  location                   = azurerm_resource_group.default.location
  resource_group_name        = azurerm_resource_group.default.name
  tenant_id                  = data.azurerm_client_config.current.tenant_id
  sku_name                   = "standard"

  # Infrastructure creator's access policy
  access_policy {
    tenant_id = data.azurerm_client_config.current.tenant_id
    object_id = data.azurerm_client_config.current.object_id

    key_permissions = [
      "Purge",
      "Get",
      "List",
      "Update",
      "Create",
      "Import",
      "Delete",
      "Recover",
      "Backup",
      "Restore"
    ]

    secret_permissions = [
      "Purge",
      "Get",
      "List",
      "Set",
      "Delete",
      "Recover",
      "Backup",
      "Restore"
    ]

    certificate_permissions = [
      "Purge",
      "Get",
      "List",
      "Update",
      "Create",
      "Import",
      "Delete",
      "Recover",
      "Backup",
      "Restore",
      "ManageContacts",
      "ManageIssuers",
      "GetIssuers",
      "ListIssuers",
      "SetIssuers",
      "DeleteIssuers"
    ]
  }
}

resource "azurerm_key_vault_secret" "my_secret" {
  name         = "MySecretName"
  value        = "MySecretValue"
  key_vault_id = azurerm_key_vault.main.id
}
```

There are a few things to notice on the previous GitHub gist. We are creating an access policy for the user applying the changes, we can do that inside the `azurerm_key_vault` Terraform resource, or we can create it as a separate resource using `azurerm_key_vault_access_policy`. That access policy gives all the permissions available to the specified user. However, the correct implementation would be to grant access only to what is needed.

One more thing to notice is how we are using a data source to obtain the tenant and object id of the current Azure CLI session when the Terraform templates are applied. It is an easy data source to call, just add `data "azurerm_client_config" "current" {}` to the .tf file and that’s it.

The last step to import our key vault with an access policy and one secret is to run the Terraform commands:

Key Vault (Access Policy Included)

```sh
terraform import azurerm_key_vault.main /subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.KeyVault/vaults/kv-myvault-001
```

Key Vault Secret

```sh
terraform import azurerm_key_vault_secret.my_secret https://kv-myvault-001.vault.azure.net/secrets/MySecretName/<secret version>
```

As you just noticed, for key vault secrets we don’t have a fully qualified resource id. We must use a “Secret Identifier”, and we can find it on the key vault secret details page in the Azure portal.

## Conclusion

Knowing how to use the `terraform import` command properly is essential when working with Terraform since having existing infrastructure that needs to be managed by it is a common scenario.

Overall, using `terraform import` is very straightforward. However, there are some details to be aware of, I didn’t cover all of them in this post, but I tried to include some of the most common ones. For instance, when importing a virtual machine it’s important to remember and know how to import the NIC and NSG association. Likewise, when importing a key vault secret you need to use the secret identifier since there is no fully qualified resource id.

I hope this post has helped you understand how to sync the Terraform state with your existing infrastructure and made the whole process easier and smooth.
