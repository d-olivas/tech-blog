---
title: "Shifting-Left Security in Azure Pipelines"
date: 2021-07-11T10:00:00-06:00
draft: false
tags: ["azure", "pipeline", "devops", "security"]
categories: ["Azure", "DevOps", "Security"]
description: "Tools for Azure Pipelines to have a more secure development process."
cover:
  image: cover.jpg
  alt: "security cameras"
  relative: true
  caption: "[Image](https://unsplash.com/photos/assorted-color-security-cameras-LfaN1gswV5c) from [Unsplash](https://unsplash.com/) by [@lianhao](https://unsplash.com/@lianhao)."
showToc: true # show Table of Contents
---

## Introduction

Shifting-left security is applying security validations to our software development process as soon as possible. Let’s say we have a development process with different steps that run from left to right consecutively. Those steps are: writing the code, creating a build, releasing it to a QA environment, releasing it to a Stage environment, releasing it to the production environment to be consumed by the end-users, and constantly monitoring the app, and the process repeats indefinitely. That’s a common and simple development cycle.

Checking the security of the app was usually done too far to the right of the development process, on the QA environment in the best cases, or when the app was already released to production, or just not at all. Finding security issues too late can be very costly for our app.

With shifting-left security what we want to achieve is to identify security issues as soon as possible. When creating a build and even while the developers are writing the code.

This post shows which tools can be used to shift-left security and how to do it using Azure Pipelines.

## Azure Pipeline Tools

### Static Code Analysis

Doing this type of analysis will help us detect common bugs, vulnerabilities, and code-smells in the app’s code at the moment of building the app.

One of the most common tools for static code analysis is called [SonarQube](https://www.sonarqube.org/) ([SonarCloud](https://sonarcloud.io/) for cloud-based code, more about the difference [here](https://blog.sonarsource.com/sq-sc_guidance)), it has an Azure DevOps extension to easily analyze code in Azure Pipelines.

Other static code analyzers can be used with Azure Pipelines as well, like [Fortify](https://www.microfocus.com/en-us/cyberres/application-security), [PMD](https://pmd.github.io/), [FindBugs](http://findbugs.sourceforge.net/), [Code Dx](https://codedx.com/), and several others.

### Open-Source Management Solutions

Checking the security of our open-source components is essential to make sure our app is truly secure. Using open-source components that have security vulnerabilities will make our app vulnerable as well.

[WhiteSource Bolt](https://marketplace.visualstudio.com/items?itemName=whitesource.ws-bolt) is a good option to easily implement in Azure Pipelines to check for vulnerabilities in open-source components, and it also checks [license risks](https://www.synopsys.com/blogs/software-security/top-open-source-licenses/). It is very easy to implement and it’s free to use, although it is a limited version of the WhiteSource complete solution, it works pretty well.

There are other great tools to make security checks of open-source components, such as [Snyk](https://snyk.io/product/open-source-security-management/) and [Black Duck](https://www.synopsys.com/software-integrity/open-source-software-audit.html). These tools also have Azure DevOps extensions to be easily implemented in Azure Pipelines.

### Variable Groups and Key Vault Secrets

Keeping sensitive information away from everyone’s eyes is another good practice to enforce security in the development cycle. We want sensitive information to be accessed and managed only by the chosen few and our pipeline.

To achieve this we can store our sensitive information, which needs to be accessed by the pipeline, as Key Vault secrets in Azure, and then read the secret values from Azure Pipelines using variable groups linked to the Key Vault. There are other ways to access Key Vault secrets from a pipeline, but this is the most effective one most of the time, it all depends on our needs.

## Azure Pipeline Example

In this example, we’ll see how to implement SonarCloud and WhiteSource Bolt in a pipeline to analyze our app’s code and open-source components when building.

> Check out these two posts about how to install and implement SonarCloud in an Azure Pipeline and how to use Azure Key Vault secrets in Azure Pipelines.

- [Setting Up SonarCloud for Azure Pipelines]({{< ref "/posts/2021/sonarcloud-in-azure-pipelines" >}})
- [How to Use Key Vault Secrets in Azure Pipelines]({{< ref "/posts/2021/key-vault-secrets-in-azure-pipelines" >}})

To use WhiteSource Bolt in our pipeline we simply install the extension to our organization and that’s it, we can start analyzing our project’s open-source dependencies.

Once we have our extensions installed we can start using them in our pipeline.

```yaml
# azure-pipeline-security-check.yml
trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '10.x'
  displayName: 'Install Node.js'

- task: SonarCloudPrepare@1
  inputs:
    SonarCloud: 'SonarCloud'
    organization: 'dolivas'
    scannerMode: 'CLI'
    configMode: 'manual'
    cliProjectKey: 'dolivas_CoolProject'
    cliProjectName: 'CoolProject'
    cliSources: '.'

- script: |
    npm install
    npm run build
  displayName: 'npm install and build'

- task: WhiteSource@21
  inputs:
    cwd: '$(System.DefaultWorkingDirectory)/src'
    projectName: 'OurVeryCoolProject'

- task: SonarCloudAnalyze@1

- task: SonarCloudPublish@1
  inputs:
    pollingTimeoutSec: '300'

- task: CopyFiles@2
  displayName: 'react publish $(buildConfiguration)'
  inputs:
    SourceFolder: '$(System.DefaultWorkingDirectory)'
    Contents: |
      **\*.js
      package.json
    TargetFolder: '$(Build.ArtifactStagingDirectory)'

- task: PublishBuildArtifacts@1
  displayName: 'publish artifacts'
```

For the WhiteSource Bolt task, we must specify the directory where the libraries we want to analyze are located. We also define a name for the project that will be created in WhiteSource. In the example above, the open-source dependencies we want to analyze are located in “$(System.DefaultWorkingDirectory)/src”. It’s important to note that the WhiteSource Bolt task must be added after building those dependencies, in this case after doing “npm install”.

The Sonar Cloud tasks do the following; first, we indicate which SonarCloud project we are going to analyze. Then, after building the project, we run the SonarCloud analysis. Finally, we publish the results of the analysis to make them visible in Azure DevOps, this last step is optional, the results can always be seen in the SonarCloud portal.

## Additional Ways to Shift Security Left

SonarCloud has a VS Code extension called SonarLint to help developers check their code as soon as they write it. Using a linter is a great option to add more security and code quality verifications to a development process.

When working with docker container registries, such as Azure Container Registry (ACR), they usually have options to analyze and update images. ACR tasks can be used to automatically build images when a base image is updated, more about ACR tasks [here](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-tasks-base-images). Additionally, Azure Defender can be enabled in ACR to analyze every pushed image, more about it [here](https://docs.microsoft.com/en-us/azure/security-center/defender-for-container-registries-introduction).

If you know other ways to shift security left that are not mentioned here, feel free to share them in a comment.

## Final Thoughts

We are all humans, no one is perfect. Because of this, we need to identify and fix vulnerabilities in our projects as soon as we can. Shifting security left helps us achieve this with our code and our open-source dependencies.

Shifting-left security is a good way to start improving our development process. However, this is not the ultimate solution to all our security issues. Having an effective testing strategy, using penetration testing tools like [OWASP Zap](https://www.zaproxy.org/), following the least-privilege principle, constantly monitoring an app’s health and behavior, as well as the overall network and infrastructure’s health and state, all these things, and even more, must be taken into consideration when thinking about security.
