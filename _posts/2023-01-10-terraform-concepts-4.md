---
title: 'Terraform learn the concepts'
date: 2023-01-10
permalink: /posts/2012/08/terraform-concepts/
tags:
  - terraform
  - Infrastruture as Code (IaC)
---

The goal of this post is to expose terraform concepts. Understanding theses concepts give solid basis.

Concepts
======
**Infrastruture as Code (IaC)**: With terraform, you define your insfrastruture in a declarative ways using configuration files which allows for version control, reprodicibility and automation.

**Providers**: Terraform uses providers to interact with an certain type og infrastructure platform. It could be AWS, Google Cloud Platform and [more](https://registry.terraform.io/browse/providers). Each providers has its own particularities.

**Resources**: They are the fundamental building blocks in Terraform. They represent and infrastructure component you want to provision. So  it could be network, storage components and more.

**Terraform Configuration Language (HCL)**: Terraform uses its own configuration Language named HCL (HashiCorp Configuration Language). This language allows you to  define resources, variables, outputs and more.

**State**:
Terraform keeps track of the state of your infrastructure using a state file (a JSON file of the resources and their current state). It helps Terraform understand the changes that need to be made to your infrastructure to reach the desired state.

**Execution Plan**:
Before making any changes to your infrastructure, Terraform generates an execution plan. The plan shows you what actions Terraform will take to achieve the desired state. It displays the resources that will be created, modified, or destroyed. So this is crucial to review the plan to ensure that you understand the impact of the changes.

**Apply**:
The `terraform apply` command is used to apply the changes specified in your Terraform configuration files. It creates, modifies, or deletes resources as needed to achieve the desired state. Terraform will prompt for confirmation before making any changes, allowing you to review and approve the modifications.

So these are the key concepts in Terraform that provide a foundation for building and managing infrastructure. 

But to better understand these concepts, it could be interestinf to work on a basic example...
