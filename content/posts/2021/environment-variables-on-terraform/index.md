---
title: "How to Use Environment Variables on Terraform"
date: 2021-03-23T10:00:00-06:00
draft: false
tags: ["environment variables", "devops", "terraform"]
categories: ["DevOps", "Terraform"]
description: "A beginner's guide."
cover:
  image: cover.jpg
  alt: mountain
  relative: true
  caption: "[Image](https://unsplash.com/photos/green-trees-near-mountain-under-blue-sky-during-daytime-drfTNwoma30) from [Unsplash](https://unsplash.com/) by [@nikola43](https://unsplash.com/@nikola43)."
showToc: true # show Table of Contents
---

## Introduction

Using environment variables with Terraform is very easy, but if you are a beginner at it then it can be a little bit tricky. Using them as input variables is fairly simple and the documentation, blogs, and other sources I found online were pretty useful. However, implementing environment variables that the Terraform providers have predefined for their configurations is not explicitly detailed in the documentation. I’ll try to explain as simple as I possibly can for anyone to easily understand how to use them.

## Using Env Variables as Input Variables

This is very straightforward, if you have ever used environment variables then you already know how to do this. Let’s do a simple example:

We’ll define a couple of env variables in our `.bashrc`, or `.zshrc`, configuration file:

```sh
# ...
export TF_VAR_EXAMPLE_ONE="<value>"
export TF_VAR_example_two="<value>"
# ...
```

Notice that to use environment variables with Terraform they must have the “TF_VAR” prefix. Now we have to define our variables in Terraform:

```hcl
variable "EXAMPLE_ONE" {
  type        = string
  description = "This is an example input variable using env variables."
}

variable "example_two" {
  type        = string
  description = "This is another example input variable using env variables."
}
```

Terraform identifies the environment variables with the specified prefix and now those variables can be used as regular input variables.

## Using Env Variables as Provider Configuration Variables

With most Terraform providers you can use environment variables for some configuration arguments. HashiCorp recommends using environment variables to set those values whenever possible to avoid having credentials saved in source-controlled Terraform code.

This is very simple, you just need to check the provider’s documentation to know what env variables it supports, here we’ll use the [Azure provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs) as an example. Let’s say we want to specify the subscription id, all we have to do is set the next environment variable, and Terraform will automatically identify it:

```sh
export ARM_SUBSCRIPTION_ID="<value>"
```

Notice that all the environment variables that can be used with the Azure provider have the “ARM” prefix. Other providers have their own prefixes as well.

Now let’s see another common example of environment variables implementation.

We need to configure Terraform to store the state remotely so that more people can contribute. To do this we’ll create a blob container in Azure Storage, we’ll keep using the Azure provider, and configure our Terraform project to save the state there. To achieve what I just mentioned we can make two different implementations.

The first option is to set the following configuration in our Terraform file:

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "<storage account rg name>"
    storage_account_name = "<azure storage account name>"
    container_name       = "<blob container name>"
    key                  = "<name of the state file>"
  }

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "2.42.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

In that backend configuration, there is an argument missing that allows Terraform to access the blob container to read and write the state. That argument is the Storage Account Access Key. That is a sensitive piece of information, so we don’t want it to be in our code.

We’ll set an environment variable for the Storage Account Access Key:

```sh
export ARM_ACCESS_KEY="<storage account access key value>"
```

By doing those two simple steps you can now run `terraform init` and you’ll have Terraform configured to store the state on a remote location to promote collaboration.

Now, the second option is to not set any argument related to the backend configuration on your Terraform file and do it at the moment of executing `terraform init`. Our provider configuration would look like this:

```hcl
terraform {
  backend "azurerm" {}

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "2.42.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

The `terraform init` command, using some example values, would look like this:

```sh
terraform init \
  -backend-config="resource_group_name=rg-terraformexample-001" \
  -backend-config="storage_account_name=stterraformexample001" \
  -backend-config="container_name=terraform-state" \
  -backend-config="key=terraform.tfstate" \
  -backend-config="access_key=<storage account access key value>"
```

or we can use our own custom environment variables as well:

```sh
$ terraform init \
  -backend-config="resource_group_name=$resource_group_name" \
  -backend-config="storage_account_name=$storage_account_name" \
  -backend-config="container_name=$container_name" \
  -backend-config="key=$tf_state_file_name" \
  -backend-config="access_key=$my_access_key"
```

or we can even set the “ARM_ACCESS_KEY” environment variables and run `terraform init` specifying the other 4 arguments:

```sh
$ export ARM_ACCESS_KEY="<storage account access key value>"
$ terraform init \
  -backend-config="resource_group_name=$resource_group_name" \
  -backend-config="storage_account_name=$storage_account_name" \
  -backend-config="container_name=$container_name" \
  -backend-config="key=$tf_state_file_name"
```

Whatever works best for you. Now you know how to use environment variables with Terraform, and also how to configure a remote state.

Using environment variables with Terraform is very straightforward, but I decided to write this article to give a clear and simple description of how to implement them. Anyone can figure this out with a bit of research and testing. Hopefully, I was able to make your research and testing shorter and easier.
