---
title: "What makes a subnet public or private in an AWS VPC?"
date: 2026-08-16T10:00:00-06:00
draft: true # not published
tags: ["aws", "networking"]
categories: ["aws", "cloud"]
description: "A simple explanation for beginners."
cover:
  image: cover.jpg
  alt: AWS VPC subnets diagram
  relative: true
  caption: "Image from draw.io by me."
showToc: true # show Table of Contents
---

## Introduction

I remember when I was first trying to understand what a subnet is, what makes it public or private, and what exactly does that mean. I would see diagrams of AWS infrastructure with public and private subnets mentioned, I understood the concept, but what exactly makes those subnets public or private was not so clear.

I would go to the AWS console, tried to create a subnet, but didn't find any "public/private" toggle button, and was very confused about how to make my subnet public.

Also, I will only cover the usage of IPv4 in tis post for simplicity.
Even though this is a post aimed to beginners, it is assumed that you already understand some basic terms like an Availability Zone, CIDR block, Network Access Translation (NAT)

I hope by the end of this post, I can give you a clear understanding of what exactly makes them public or private.

## Relevant AWS Networking Resources

If you are already familiar with the basic AWS networking resources feel free to skip to the next section. I will not go into technical details here, there is plenty of official documentation and information out there. I will try to give a short explanation of each resource in my own terms that hopefully will make it easier for beginners to understand what they are and what they do.

### Virtual Private Cloud (VPC)

This is your network. Think of it as a logical container with a defined number of slots, IP addresses, for your resources. It needs to be subdivided using subnets before you can add your resources into it.

What defines the number of available slots for your resources within the VPC is a CIDR block. It has to be specified on creation of the VPC.

### Subnets

Following the same analogy, subnets are smaller logical containers within your VPC with more specific purposes. A common example is having a subnet for publicly accessible resources, like a internet-facing Application Load Balancer, another subnet for the application servers, and one last subnet for data bases.

A subnet can only be assigned to one AZ, but one AZ can have multiple subnets. It is a good practice to define subnets in at least 2 different AZs, for high availability.

### Route tables

We have our logical containers for our resources. But they need to be able to communicate. They define where a subnet's traffic has to go. You assigned a route table to a subnet, and this is how it knows where to send the traffic.

For example, in subnet A you need all your resources to be able to access the internet because they need to download version patches or whatever, so in subnet A's route table you define a route to send all traffic, 0.0.0.0/0 to an Internet Gateway or a NAT Gateway, more about them below.

A subnet can only have assigned one route table, but the same route table can be assigned to several subnets. Spoiler alert, the routes in a route table are a very important component of what make sa subnet public or private.

### Internet Gateway (IGW)

As its name says it, this is the VPC's gateway to the internet. It's an AWS managed resources, so once created, it simply has to be attached to the VPC and is ready. A route table can also be associated to an IGW, but this is for more advanced use cases. Usually, the IGW is ready to go as it is.

### NAT Gateway (NAT GW)

It enables communication from resources within your "private" subnets to other resources outside of your VPC, without allowing the external resources to connect.

imagine you live in a closed community, and there is a mesangers company who you can hire anytime you need to run an errand, they do it for you, they go and buy or do wahtever, and bring you back the result, and that's it. So no one from outside of the communicaty can reach you, only you can reach them if you send a messanger. 

There ar public and private ones, but mainly public NAT GW are used. THe pirvate one does the same, but not to the internet but to other VPCs or on-prem networks.

## What makes a subnet public or private?

Basically, a public subnet is one that its route table has a route to the internet through an IGW. Meaning, 0.0.0.0/0 to the IGW. And a private subnet is one that doesn't. But a private subnet can still have a route to the internet, but through a NAT GW instead of an IGW. And another version of a private subnet is one that simply doesn't have any route to the internet at all.

### What makes a resource public?

Ok, so now we know what makes a subnet public or private. But what happens if I create a resources in a public subnet, does that make it public as well? The answer is no, there are a few conditions to be met before a resource can be considered public.

- It must be in a public subnet (0.0.0.0/0 route to IGW)
- It must have a public IP assigned
- Its security group must allow inbound traffic for externals to communicate with it
- The NACL of the subnet must also allow the inbound and outbound communication. By default, all traffic is allowed.

## When To Use one or the other with real examples

Basically, use a public subnet for resources that will be publically accesible throuhg internet. Like an Application Load Balancer, or some service that needs to be publically accessible. And use private subnets for resources that are not publically accessible, like your backend application servers, data bases, internal applications.

### Diagram

 <GENERATE SOME DIAGRAM>

## Common pitfalls

EC2 instance in public subnet, but not publically accessible. DOes it have a public IP address assigned?

DO I need to have a NAT Gateway in my private subnet? It depends, if you need the reousrces in that subnet to have outbound internet communication, then yes. If not, then there's no need for NAT Gateway.


## Conclusion

Basically, the route table associated to the subnet is what make sit public or private. THere is no attibute in the subnet, or name tag or anything else that makes a subnet public or privet. If the route table has a 0.0.0.0/0  route to an INternet Gateway, it will be public. If it has a route to the itnernet through the NAT GAteywa, it is private with outbound internet access, if doesn't have an internet route at all, it is completely private and shut down from the internet.