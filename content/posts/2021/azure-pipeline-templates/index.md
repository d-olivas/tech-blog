---
title: "How to Use Azure Pipeline Templates"
date: 2021-08-31T10:00:00-06:00
draft: false
tags: ["azure", "pipeline", "devops", "yaml", "templates", "continuous integration"]
categories: ["Azure", "DevOps"]
description: "Easy guide on how to get started with Azure Pipeline Templates"
cover:
  image: cover.jpg
  alt: alt_text
  relative: true
  caption: "[Image](https://unsplash.com/photos/pile-of-brown-wooden-blocks-L0oJ4Dlfyuo) from [Unsplash](https://unsplash.com/) by [@lunarts](https://unsplash.com/@lunarts)."
showToc: true # show Table of Contents
---

## Introduction

There’s always that situation where we need to use the same elements, or code, in several places and we want to avoid copying and pasting those elements everywhere because we like to keep it DRY (Don’t Repeat Yourself). That’s where pipeline templates come in handy when facing this dilemma while working with Azure Pipelines.

## What are Pipeline Templates?

Pipeline templates help us define reusable blocks that can be used across as many pipelines as we need, making maintenance easier and less prone to errors.

We can define individual steps, jobs with several steps, and stages with several jobs in pipeline templates. Variables and parameters can also be specified.

Pipeline templates are only available for YAML pipelines, so it makes sense that the templates are also defined as YAML files. Classic pipelines have task groups, but they are more limited in functionality.

## Pipeline Template Structure

As mentioned above, we can define parameters, variables, steps, jobs, and stages in our templates. Here we’ll see in more detail each of those elements.

### Parameters

Parameters help us pass values from the main pipeline to our template. Parameters must be defined with a name and a type, a default value is optional. For parameters of data types string and number, we can optionally specify the allowed values that it will accept.

```yaml
parameters:
- name: myParam
  type: number
  default: 9
  values:
  - 3
  - 6
  - 9
```

To use the parameter value in the template we use the following expression “`${{ parameters.<parameter name> }}`”. For example, to use the parameter we just defined we’d do it like this:

```yaml
${{ parameters.myParam }}
```

### Variables

We cannot define variables in a template where we will use steps, jobs, or stages. Variables in a pipeline template are used similarly as we would use a variable group. We can define a template with several variables and use that template in several pipelines to have access to those variables. For example:

```yaml
variables:
  myVarEng: "Hello World!"
  myVarEsp: "Hola Mundo!"
```

That’s all we need to define in the template file. Then to use it in a pipeline we call it this way:

```yaml
variables:
- template: template-vars.yml # Name of the template file
steps:
- script: echo ${{ variables.myVarEng }}
- script: echo ${{ variables.myVarEsp }}
```

This is a good option to share variables around different pipelines, but I would only recommend it if for some reason it is required for those variables to be stored in a repository. If that’s not the case, then a variable group would be a better implementation.

When a more secure implementation is necessary to share variables between pipelines; Azure Key Vault secrets and variables groups are a great option. Check out [this post]({{< ref "/posts/2021/key-vault-secrets-in-azure-pipelines" >}}).

### Steps

We can create a template to reuse one or more steps in a job of a pipeline. Simply define the steps in the template as we would do in a YAML pipeline. Using a pipeline template this way is very similar to using task groups in classic pipelines.

```yaml
steps:
- script: echo "First very useful step"
- script: echo "Second equally useful step"
```

Then, in the pipeline, we have to define a job and reference the template to use the steps.

```yaml
jobs:
- job: Linux
  pool:
    vmImage: 'ubuntu-20.04'
  steps:
  - template: template-steps.yml # Name of the template file
```

### Jobs

Pipeline templates also give us the ability to define jobs. Again, we define the jobs the same way we would in a YAML pipeline.

```yaml
jobs:
- job: Ubuntu
  pool:
    vmImage: 'ubuntu-20.04'
  steps:
  - script: echo "Hello World from Ubuntu!"
- job: Windows
  pool:
    vmImage: 'windows-latest'
  steps:
  - powershell: echo "Hello World from Windows!"
```

Using the template in a pipeline is very easy.

```yaml
jobs:
- template: template-jobs.yml # Name of the template file
```

One thing to keep in mind when using templates that contain jobs is that job names must be unique within a pipeline. So if a template is used more than once in the same pipeline and the jobs in the template have a name, the pipeline will show an error that job names can only appear once. The easiest solution is to remove the name of the jobs in the template, names are not required.

### Stages

The same principle applies to stages. We can define one or several stages in our template just like we would in a YAML pipeline.

```yaml
stages:
- stage: MyCoolStage
  jobs:
  - job: MyCoolJob
    pool:
      vmImage: 'ubuntu-20.04'
    steps:
    - script: echo "Hello World!"
```

Using the stages template is basically the same implementation used with the jobs template.

```yaml
stages:
- template: template-stages.yml # Name of the template file
```

More stages and jobs can be added before and after the template reference in the pipeline.

```yaml
stages:
- stage:
  jobs:
  - job: MyCoolJob
    pool:
      vmImage: 'ubuntu-20.04'
    steps:
    - script: echo "Hello World!"
- template: template-stages.yml # Name of the template file
```

## Example Scenario

Now let’s see an example scenario where pipeline templates are useful. We have three containerized applications with Docker, we need to create a pipeline for each app, they have their own repo. The build pipeline for the three apps needs to do the same thing; build the Docker image and push it to an ECR (Elastic Container Registry) repo in AWS with two tags: latest and a unique value to identify that specific image.

Without a pipeline template, we would have to add the required tasks in each of the pipelines. That means repeating the same process three times. Plus, if a change has to be done in the future, there are three different places where we would have to apply the changes.

The pipeline template for this scenario looks like this:

```yaml
# docker-ecr-pipeline-template.yml
parameters:
  - name: dockerfilePath
    type: string
  - name: dockerImageName
    type: string
  - name: ecrRegion
    type: string
    default: "us-east-1"
    values:
    - "us-east-1"
    - "us-east-2"

steps:
  - task: Docker@2
    displayName: "Build Docker Image \"${{ parameters.dockerImageName }}\"."
    inputs:
      repository: ${{ parameters.dockerImageName }}
      command: build
      Dockerfile: ${{ parameters.dockerfilePath }}
      addPipelineData: false
      tags: |
        latest
        $(Build.SourceVersion)
  - task: ECRPushImage@1
    displayName: "Push \"${{ parameters.dockerImageName }}:$(Build.SourceVersion)\" to ECR."
    inputs:
      awsCredentials: "AwsServiceConnection"
      regionName: ${{ parameters.ecrRegion }}
      sourceImageName: ${{ parameters.dockerImageName }}
      sourceImageTag: $(Build.SourceVersion)
      repositoryName: ${{ parameters.dockerImageName }}
      pushTag: $(Build.SourceVersion)

  - task: ECRPushImage@1
    displayName: "Push \"${{ parameters.dockerImageName }}:latest\" to ECR."
    inputs:
      awsCredentials: "AwsServiceConnection"
      regionName: ${{ parameters.ecrRegion }}
      sourceImageName: ${{ parameters.dockerImageName }}
      sourceImageTag: latest
      repositoryName: ${{ parameters.dockerImageName }}
      pushTag: latest
```

The template receives three parameters that are used by the tasks. We are only using steps, the job for the template will be defined in the pipeline itself.

The first step builds the Docker image with two tags: “latest” and the latest commit id from which the pipeline was triggered (learn more about predefined variables [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml)).

The second and third steps push the image to the same ECR repository using different tags. That’s the way the “ECRPushImage@1” task works, it can only push one tag at a time. It won’t create two images in the ECR repo, it will be only the one image with the two tags we specified.

Now we can use the pipeline template in our three pipelines, and if we have to make any update to how we build or push the image, we only have to update the template. We won’t even need to touch the pipelines unless there’s a change that affects the parameters.

The pipeline looks the same for the three apps, so here is how we use the template in the app-one pipeline:

```yaml
# app-one-pipeline.yml
resources:
  repositories:
    - repository: templates
      type: git
      name: MyDevOpsProject/PipelineTemplates

trigger:
  - master

jobs:
  - job: TemplateUseExample
    pool:
      name: "Ubuntu-18.04"

    steps:
      - checkout: self
        fetchDepth: 1

      - template: docker-ecr-pipeline-template.yml@templates
        parameters:
          dockerfile: Dockerfile
          dockerImageName: app-one
      
      - script: echo "Final step, have a great day!"
```

We have to specify a repository resource because for this scenario we have our pipeline template in a different repository in Azure DevOps. So we have to specify where we are getting the template file from. When referencing it in the pipeline, we have to do it like this “docker-ecr-pipeline-template.yml@templates”, use the name of the file and the repository resource name.

Notice how we are not specifying the “ecrRegion” parameter, since we defined a default value for it we can ignore it when referencing the template, and it will use that default value.

This is an adaptation of a real implementation for a project. There are some details about the push to ECR task that are out of the scope of this post, but to learn more about it check out the [AWS Toolkit for Azure DevOps](https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.aws-vsts-tools).

## Final Thoughts

Pipeline templates are very helpful to keep our pipelines clean, organized, maintainable, and flexible. Using a template is very straightforward, it’s almost the same as a regular pipeline, but defining some parts in a different reusable file.

Whenever we might need to use a specific set of tasks in more than one pipeline, we should use a pipeline template.

This was an introduction to how to use pipeline templates. Please check [the official Microsoft documentation about Azure pipeline templates](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops) to learn more about them. There we can find out more about all the available parameter data types, template expressions, limits, and other very interesting details.
