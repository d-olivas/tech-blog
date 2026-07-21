---
title: "How to Use Key Vault Secrets in Azure Pipelines"
date: 2021-06-14T10:00:00-06:00
draft: false
tags: ["azure", "pipeline", "devops", "yaml"]
categories: ["Azure", "DevOps"]
description: "Different ways to access Key Vault secrets in Azure Pipelines."
cover:
  image: cover.jpg
  alt: "Woman shushing"
  relative: true
  caption: "[Image](https://unsplash.com/photos/grayscale-photo-of-woman-doing-silent-hand-sign-BcjdbyKWquw) from [Unsplash](https://unsplash.com/) by [@tinaflour](https://unsplash.com/@tinaflour)." 
showToc: true # show Table of Contents
---

## Introduction

Using Key Vault secrets to store sensitive data is essential to keep it safe. By using secrets we can control who has access to see and manipulate the data stored in them, making sure only the correct people have the appropriate level of access.

It is very common to have a pipeline that needs a key, a password, a connection string, or any other type of data that can be considered sensitive and we don’t want anyone seeing or manipulating its value. For these scenarios, most of the time the best solution is to use Key Vault secrets.

There are several different ways to access Key Vault secrets from an Azure Pipeline. This post will cover the two most common ones, using a variable group and using the Key Vault task. Another way of accessing secrets from the pipeline is by using the Azure CLI, but doing it this way is over-complicating the implementation when there are easier ways like the two mentioned above.

## Prerequisite

Before creating a variable group or using the Key Vault task, we must configure an access policy in our Key Vault to allow the service connection to have get and list access to the secrets. To achieve this, we will need the service principal display name or application id. To get those values we can go to “Project settings” in Azure DevOps, then click on “Service connections”, select the service connection that will be used by the variable group, and then click on “Manage Service Principal”.

{{< figure src="images/service-connections.webp" alt="See service principal info." caption="See service principal info." align="center" >}}

That will takes us to Azure Active Directory, where we’ll be able to see all the information related to the service principal. There we can copy the display name or the application id, either of them will work to create the access policy in the Key Vault.

Now we must go to our Key Vault to create a new access policy. For this access policy, we only need the get and list permissions for secrets.

{{< figure src="images/secret-permissions.webp" alt="Get and list secret permissions." caption="Get and list secret permissions." align="center" >}}

Now we must select the service principal that we are going to apply this policy to. In this example, we copied the display name of the service principal, we’ll use it to search for it, select it, and add it to the access policy.

{{< figure src="images/service-principal-to-access-policy.webp" alt="Add service principal to the access policy." caption="Add service principal to the access policy." align="center" >}}

Important, always make sure to save any change made to the “Access policies” section of a Key Vault, Azure even shows us an alert message to make sure we save our changes.

{{< figure src="images/save-access-policy.webp" alt="Save access policy changes." caption="Save access policy changes." align="center" >}}

Now that we have provided access to get and list secrets from the Key Vault to our service principal we can implement the two methods to get secrets from a pipeline.

## Azure DevOps Variable Groups

Variable groups are useful when we have several variables that are used across different pipelines. Instead of defining the same variable over and over again in each pipeline, simply add it to a variable group and link that group to the pipelines where the variable needs to be used.

To create a variable group we have to go to Azure DevOps, under “Pipelines” click on “Library”, and then “+ Variable group”.

{{< figure src="images/new-variable-group.webp" alt="Create a new variable group, part 1." caption="Create a new variable group, part 1." align="center" >}}

When creating a new variable group we must define a name, the description is optional, and toggle the option to link secrets from Key Vault. By toggling that option we’ll see two drop-down menus to select our service connection and Key Vault, we’ll choose the correct values we want to use.

{{< figure src="images/new-variable-group-details.webp" alt="Create a new variable group, part 2." caption="Create a new variable group, part 2." align="center" >}}

When the Key Vault has been selected, we’ll see a new section appear called “Variables”, here we can choose exactly which secrets we want to add to the variable group.

{{< figure src="images/new-variable-group-secret.webp" alt="Create a new variable group, part 3." caption="Create a new variable group, part 3." align="center" >}}

We only have one secret in our Key Vault, so we’ll select it and click on “Ok”. After choosing the secrets to add to the variable group we can save our changes and start using our new variable group in our pipelines.

First, let’s see how to access a variable group from a YAML pipeline. It is very simple, all we need to do is to call the group by its name in the variables section.

```yaml
variables:
  - group: "Var Group 01"
```

To get the secret value, use it as a regular pipeline variable.

```yaml
$(my-first-secret)
```

If we needed to have other variables in our pipeline, we can define them in the same variables section.

```yaml
variables:
  - group: "Var Group 01"
  - name: my-pipeline-var
    value: "This is my pipeline variable"
```

Finally, using a variable group in release pipelines. For this, we need to go to the “Variables” section of our pipeline, select “Variable groups”, and “Link variable group”.

{{< figure src="images/var-group-in-release-pipeline.webp" alt="Use variable group in a release pipeline, part 1." caption="Use variable group in a release pipeline, part 1." align="center" >}}

Then we select the variable group that we want to link to our pipeline and set the scope for that variable group. The scope can be the whole release or, it can be specific to only certain stages. For simplicity, we’ll select the release as the scope. And click on “Link”.

{{< figure src="images/link-var-group-to-release-pipeline.webp" alt="Use variable group in a release pipeline, part 2." caption="Use variable group in a release pipeline, part 2." align="center" >}}

Now we can access the secret by using it as a pipeline variable, within the scope defined for the variable group.

## Azure Pipeline Key Vault Task

Azure pipelines have a task called “[Azure Key Vault](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/azure-key-vault?view=azure-devops)” that can be used to download secrets and use them as pipeline variables. After setting up this task, all the following tasks will have access to the downloaded secrets.

Using this pipeline task is quite simple, in the pipeline assistant look for a task called “Azure Key Vault” and click on it. We must select the service connection we want to use and the Key Vault from where we must read the secrets. There is a task parameter called “Secrets filter”, with this parameter we can specify exactly which secrets, separated by a comma, we want to download from the Key Vault. We can also use “*” to get all the secrets.

{{< figure src="images/key-vault-pipeline-task.webp" alt="Azure Key Vault pipeline task." caption="Azure Key Vault pipeline task." align="center" >}}

The option to “Make secrets available to whole job” makes the secrets downloaded using this task available to all the other tasks in the job, no matter their position. By default, only the tasks defined after this task will be able to use the downloaded secrets. Also, only tasks within the same job will be able to access the secret.

The task setup is the same for YAML and release pipelines. Accessing the secret is done the same way in both types of pipelines. It can be accessed as a pipeline variable.

```yaml
$(my-first-secret)
```

This is what the task looks like in a YAML pipeline.

```yaml
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'personal-projects (...)'
    KeyVaultName: 'kv-demotest-001'
    SecretsFilter: '*'
    RunAsPreJob: false
```

## Final Thoughts

It’s essential to always avoid storing sensitive data as plain text in a variable which we can’t control who access its value. That’s why Key Vaults are a great tool to store sensitive data, and variable groups and the Key Vault pipeline task are the best methods to use secrets in a pipeline.

Choosing between one method or the other is going to depend on our needs. There is no good or wrong implementation, simply understand as clearly as possible what the needs are and use the method that best fits those needs.
