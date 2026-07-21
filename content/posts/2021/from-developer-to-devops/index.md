---
title: "How I Ended Up Working on DevOps"
date: 2021-10-01T10:00:00-06:00
draft: false
tags: ["software development", "career change", "devops", "motivation"]
categories: ["DevOps", "Motivation"]
description: "From Developer to DevOps Engineer."
cover:
  image: cover.jpg
  alt: "intersection sign"
  relative: true
  caption: "[Image](https://unsplash.com/photos/silhouette-of-road-signage-during-golden-hour-C7B-ExXpOIE) from [Unsplash](https://unsplash.com/) by [@soymeraki](https://unsplash.com/@soymeraki)."
showToc: true # show Table of Contents
---

## Introduction

There are many paths we can take in software engineering, but the most common one is to start as a developer. I started as a software developer for a small Costa Rican startup, then moved to a medium-sized US company in Costa Rica, where I had the opportunity to learn DevOps. As of right now, I am in the process of moving to a different country because of a job opportunity.

## Software Developer Experience

In the university where I studied, one of the last requisites to graduate is to do an internship at a company working on something related to software engineering. Ninety-nine percent of the time everyone ends up coding.

Four classmates and I found a small startup that accepted us. We didn't have any cool benefits or perks that the big companies give to their interns. We were given just enough money for transportation and we had to bring our computers for work. However, the company was using cutting-edge technologies at that moment and we were given a lot of freedom and responsibility on their projects, which now I realize was incredibly helpful to gain experience on high-demand tools, and also to build a sense of belonging, responsibility, and confidence.

After the internship finished I was offered a job, which I accepted and started working as a full-stack software developer. After a few months, the startup outsourced me to a Costa Rican FinTech company to work on a mobile app. That project introduced me to the cloud and broaden my tech horizons. We were creating a serverless backend, everything was in AWS, and all the tech we were using was new to me. There was no senior engineer, no architect, no DevOps, no nothing, we were just a bunch of mid-level developers who had to start learning AWS on the fly.

It was a frightening experience, we had a lot of responsibility on our shoulders, but at the same time, it was the best experience that I’ve had so far working on a project. We were taking almost all the tech decisions. Of course, they had to be discussed and approved by the Product Owner, but most of the time whatever new tool or service we proposed, it was implemented. Technically speaking, we were creating the application the way we wanted. My colleagues were amazing, it was the best team that I’ve worked with yet, that was a key ingredient to make it such an amazing experience.

On that project, I realized that I wanted to continue working on the cloud. When the project was getting close to being released to production I started looking for opportunities at other companies, higher salaries, better benefits, etc. At that moment I didn’t even understand what DevOps was, I just knew that I wanted to continue working on the cloud.

I was interviewed for a software developer position, I completed the whole process and I felt like I did well, but I didn’t hear back from the company for about a month. I was looking forward to getting that position at that specific company, I was certain I was not going to hear from them again. Then, one day I revied a call from that company asking me if I would be interested in learning DevOps. One of their clients was looking for someone who wanted to get started on DevOps but didn't have any experience. The client had only one DevOps Engineer that was being overwhelmed with work, but they didn’t want to hire an experienced DevOps Engineer.

That was my opportunity. I had not planned to move away from software development, but it was a great opportunity that I couldn’t let off. I had an interview with the client, they liked me, and that’s how I started working as a DevOps Engineer, or should I say, that’s how I started learning what DevOps is.

## DevOps Engineer Experience

It is very exciting when starting at a new job, new office, new laptop, new colleagues, new challenges, etc. It is very exciting for me at least. I had done some research about DevOps, what it meant, what it did, how it worked, but still, I didn’t understand at that moment what it was. I was ready to start learning by doing.

The first tasks that I had, were to create PowerShell scripts to provision infrastructure in Azure. Not the best approach, but I had no idea IaC tools existed at that moment. I started to get familiarized with the Azure portal and PowerShell, something I had never used before.

For the first 6 months, I was focused on creating provisioning scripts, setting up Azure DevOps pipelines to run the scripts, and using the Azure portal to check the provisioned services. I was trying to wrap my head around VNets, Subnets, NSGs, VMs, RBAC permissions, Azure Active Directory, Build and Release Pipelines, Regions, Availability Zones, Scalability, High Availability, Disaster Recovery, and several other terms. It was very overwhelming.

Then, I was moved to work as a developer again. The client needed a developer for a project, and it was at the time of the covid-19 pandemic, so there was a lot of uncertainty about the future, they didn’t want to hire a new person. Since I had a full-stack developer background, it was the perfect solution. I worked coding for about 4 months, after repeatedly expressing that I wanted to get back to work in DevOps I was finally moved out of that project. While I was waiting to be moved back to do DevOps work I was preparing and eventually obtained the Azure Administrator Associate Certification from Microsoft.

Once I finally got assigned a DevOps project, I started with a new client. The client was migrating from AWS to Azure. I was responsible for creating pipelines in Azure DevOps to deploy the applications to Azure services and create the infrastructure, among other things. They were not using any IaC tool, I had some experience with ARM Templates, so I suggested using them. As I was creating the templates for the Azure services, I came across Terraform, I had heard of it but had no experience using it. I did a little research about Terraform and decided to talk to the engineer in charge of the migration effort and convinced him to drop the ARM templates and create all the Azure infrastructure using Terraform, a tool no one in the project had ever used, including myself.

All the infrastructure was created using Terraform, I had gained very valuable experience with one of the most used DevOps tools. We finished the migration, all the infrastructure and pipelines were working, now it was the client’s responsibility to maintain all that. Working on the migration helped me a lot to get a better feel of real DevOps work.

After that project finished, at least for me, I had no client to work with. I was converting some AWS CloudFormation templates into Terraform templates for an internal project of the company I was still working for, while they were looking for a new client for me. I also took advantage of having a bit more slack time to prepare for the Microsoft DevOps Engineer Expert Certification.

About two months passed until I started working with a new client. This time, the client had most of their infrastructure in AWS, and, funny enough, they were migrating some applications they had in Azure to AWS.

It was a big client, they had several projects and a lot of services in AWS. I had worked a little bit with AWS as a developer, but this was my first time working with AWS as a DevOps Engineer, it was both exciting and a little scary. I felt very comfortable working with Azure at that moment, and now I was going to switch to AWS. I had to start learning how AWS worked, the services that the client used. But at least they also used Azure DevOps, so all that knowledge I gained from the previous projects was going to be handy.

With this client, I learned Pulumi, something I had never heard about before. It is an IaC tool to build infrastructure using popular coding languages (Python, C#, JS, TS, and Go). However, after a few weeks in a project, they decided to switch to Terraform. So all the work I had done for the project using Pulumi, now had to be re-done using Terraform. The good thing is that now I can add Pulumi to my resume 😅.

My main responsibilities working with this client were creating Terraform modules and templates for AWS infrastructure and creating and maintaining pipelines in Azure DevOps. Working with this client I gained a lot of experience in AWS. I had the opportunity to work with very talented people that had vast knowledge and experience with AWS, I learned a lot from them.

That’s my professional experience so far. About two years of experience as a DevOps Engineer, at the moment of writing this article. I still have a lot of technologies to learn, a lot of scenarios to face, and a lot of experience to gain. I look forward to seeing what the future will bring.

## Final Thoughts

There are a few things I want to highlight.

First, change can be frightening but only when we get out of our comfort zone is when we start making significant progress. This is especially true in the tech industry, where we must constantly update and learn new tools to stay relevant. Don’t let fear nor laziness hold you back.

Second, good opportunities may come at any time. But it is naive to think that you can sit and wait for the best opportunity to magically appear out of nowhere. If you don’t actively look for them, a better job, better relationships, better anything, it will probably never come.

Finally, speak your mind. Share what you think about how the project is being handled, what you think can be improved, what you think is working, what you think is not working. Now that Agile is widely used, there are more opportunities to share how you feel about a project, a client, a co-worker, thanks to ceremonies like a sprint retrospective. Feel free to express your thoughts. I hope you work in a healthy environment where your input will be properly considered if that’s not the case then I wish you the best to find a healthier environment to work in.

This has been my experience working as a Software Developer and then as a DevOps Engineer. Hopefully, this will motivate you to get out of your comfort zone, if you haven’t already, and try something new, professionally or personally. We will never know what opportunities life has for us if we never get out to find them.

Changing my career path was something that had never crossed my mind at that moment, and now I am very happy I did. Taking that opportunity to start learning DevOps opened a lot of other opportunities that made me be where I am right now, preparing to move to a different country to start a very exciting new job.