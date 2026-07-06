---
title: "YAML vs Classic Azure Pipelines"
date: 2021-01-16T10:00:00-06:00
draft: false
tags: ["azure", "pipeline", "devops", "yaml"]
categories: ["DevOps"]
description: "An honest comparison based on experience."
cover:
  image: cover.jpg
  alt: "Pipelines"
  relative: true
  hiddenInList: true
  hiddenInSingle: false
showToc: true # show Table of Contents
---

## Introduction

Classic pipelines are set up using a UI and clicking around, moving through tabs, and choosing between the options the UI shows you. The same definition applies to release pipelines, the difference is that the former are used generally to build applications and the latter to deploy them.

On the other hand, YAML pipelines are set up using code on, you guessed it, a YAML file. Azure DevOps has a task assistant that helps you find the tasks you need and add them to the YAML file.

You can build and deploy an application using classic pipelines. However, release pipelines have more features to manage deployments for different environments, triggers, pre, and post-deployment conditions, etc.

With YAML pipelines you can build and deploy an application as well. They have something called “deployment jobs” and “environments” that can be used to deploy to different environments, similar to what the release pipelines do.

Your content here in **Markdown**.

## Pros and Cons

### YAML Pipelines

✅ Collaboration is easier since it is code and it can be treated as such (code reviews, pull requests, formatting tools, in-code comments).

✅ Another benefit of it being code is that it is easy to make changes to multiple values at the same time. For example, using your IDE’s search tool to find and replace several appearances of the same value.

✅ Using a version control tool, such as git, makes it very easy to compare changes.

✅ Since the pipeline is in a YAML file in the repo, if the repo needs to be reverted to an older commit, the pipeline version will also be reverted.

✅ It is now the default option in Azure DevOps for build pipelines.

✅ Most CI/CD tools support YAML, which makes it easier for you to change from one tool to another.

✅ Container jobs are exclusive to YAML pipelines.

🚫 It may take a bit more effort and time to get the hang of it. However, you can use the task assistant to add the tasks you need. Similarly, an existing classic pipeline can be exported as YAML, though a few adjustments will be necessary to get it working properly.

🚫 Some configurations cannot be modified in the file, they have to be changed using the UI, which can be a little hard to find the first time (in the pipeline editor, click on the ellipsis menu icon, and click on Triggers). One example of these configurations is the “Default branch for manual and scheduled builds” option.

🚫 Some of the CD features available for the release pipelines are not available yet for YAML pipelines. This “con” will eventually disappear, as Microsoft adds more features to the YAML pipelines, but that might take a while.

### Classic Pipelines

✅ They are very user-friendly. Everyone knows how to click around.

✅ It is easy to quickly discover a setting by just clicking around and not having to visit the documentation all the time.

🚫 It is not the default option in Azure DevOps for pipelines anymore. It will be deprecated eventually.

## Conclusion

In my experience, and in the state Azure Pipelines are at the moment of writing this article, YAML pipelines are better for CI, and release pipelines are better for CD.

The truth is, it depends on what requirements you have at the moment of selecting the pipeline implementation. Make sure to analyze how the pipelines will be triggered and the environments where the application will be deployed, how the branching strategy works, and affects the deployment flow. If you need to have a single CI/CD pipeline or two individual pipelines. Make sure to know well your workflow and requirements.

YAML pipelines are continuously improving, but for CD they are still missing important features. I once tried to set up a CD YAML pipeline to be triggered after a CI pipeline was automatically triggered by a Pull Request, that’s a very easy setup for release pipelines, but for a YAML pipeline you’d have to do a more complex workaround to get it working, I ended up using release pipelines.
