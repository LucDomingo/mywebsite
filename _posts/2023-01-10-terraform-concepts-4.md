---
title: 'Terraform learn the concepts'
date: 2023-01-10
permalink: /posts/2012/08/terraform-concepts/
tags:
  - terraform
  - Infrastruture as Code (IaC)
  - AWS
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

Let's work on an example
======
<pre>
# We use the aws provider to interact with AWS services in the provider block. One of the option of AWS provider block allows us to choose the region to where deploy our EC2 instances.
  provider "aws" {
  region = "us-west-2"
}

# variable is a very simple block that provides variables definition
# The instance_count variable is defined to control the number of instances to create.
variable "instance_count" {
  description = "Number of instances to create"
  type = number
  default = 2
}

# Provisions a VPC
resource "aws_vpc" "example_vpc" {
  cidr_block = "10.0.0.0/16"
}

#  Provision a subnet for our aws_vpc
resource "aws_subnet" "example_subnet" {
  vpc_id = aws_vpc.example_vpc.id
  cidr_block = "10.0.1.0/24"
}

# The example uses the aws_instance resource to create EC2 instances. The number of instances created is controlled by the instance_count variable.
resource "aws_instance" "example_instance" {
  count = var.instance_count
  instance_type = "t2.micro"
  ami = "XXX" # Replace with your desired AMI

  subnet_id = aws_subnet.example_subnet.id
  associate_public_ip_address = true

  # Each EC2 instance is tagged with a unique name using the `tags` attribute
  tags = {
    Name = "Example Instance ${count.index + 1}"
  }
}

# The public IP addresses of the created instances are exposed as an output after the terraform apply command.
output "instance_ips" {
value = aws_instance.example_instance[*].public_ip
}
</pre>
