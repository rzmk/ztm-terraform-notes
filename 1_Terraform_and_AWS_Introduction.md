# 1 - Terraform and AWS Introduction

## IAC Tool Comparison: Terraform vs Ansible

### Provisioning Tools

- Provisioning tools are used to provision infrastructure (servers, load balancers, databases, etc.), such as Terraform and CloudFormation.

### IAC Tools

- Configuration management tools are designed to install and manage software on existing softwares, such as Ansible, Chef, puppet, SaltStack. Terraform has some configuration management capabilities.

### Declarative (Functional) vs. Imperative (Procedural)

- Terraform is declarative, describing an intended goal rather than the steps needed to reach that goal - **What to do**.
- Procedural tools specify step by step how to achieve the desired end state - **How to do**.
- Terraform is aware of previous state, so changing the code (i.e. adding more servers) is based on the end result.

## Creating an IAM User

- When creating an AWS account, a root account is available. It's recommended to create an IAM user for keeping access secure, along with enabling Multi-Factor Authentication.

## AWS Basics VPC

### Virtual Private Cloud (VPC)

- VPC closely resembles a traditional network in a data center and you'll launch the other resources in a VPC.

## AWS Basics EC2

- EC2 provides scaling computing capacity in the AWS cloud. EC2 provides virtual computing environments, or instances (i.e. virtual machines)
- You can launch a VM with an Amazon Machine Image (AMI), which is a special type of virtual appliance to make a virtual machine within EC2. It includes an OS and additional software for our server. AMIs are available on the AWS Marketplace.
