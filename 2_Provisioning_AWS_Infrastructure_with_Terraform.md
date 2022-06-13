# 2 - Provisioning AWS Infrastructure with Terraform

## Terraform Code Structure

- The steps of a Terraform workflow are write, plan, and apply.
- A Terraform module is a container for multiple resources that are used together.
- The root module is the directory with the configuration files where you'll run the Terraform command.
- The main Terraform configuration file is commonly named `main.tf`.

## Terraform Providers

- It's important to connect to a cloud platform when starting work with Terraform. Terraform uses **providers**, which are plugins that Terraform uses to create/manage resources on a cloud platform.
- Providers are like modules/packages in a programming language.
- Providers are distributed separately from Terraform itself.
- Providers are often full cloud providers based or software-based. There are providers for IaaS, PaaS, and SaaS.
- The Terraform Registry is the main directory of publicly available Terraform providers for most major infrastructure providers.
- There are Official, Verified, and Community providers.
- Each provider has an extensive documentation for how to use it and what it does.
- Provider configurations belong to the root module of a Terraform project (`main.tf`).
- A provider configuration is made with a provider block.

```terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"
}

# Create a VPC
resource "aws_vpc" "example" {
  cidr_block = "10.0.0.0/16"
}
```

- The given name `"aws"` is the local name of the provider we give to configure, which should be included in the `required_providers` block which must be nested in the `terraform` block.
- The body of the block contains configuration arguments for the provider. `region` is the argument to the `aws` provider.
- Each argument in the `required_providers` block enables one provider. The key `aws` determines the provider's local name. `source` is the global and unique source address forr the provider, which allows Terraform to download that source along with `version` for identifying the provider version.
- You can use multiple `provider` blocks to manage resources from multiple providers.

## Terraform Configuration Syntax

- There are different ways to use Terraform:
  - 1. Native Terraform Language (`.tf` files)
  - 2. JSON
  - 1 + 2 = Hashicorp Configuration Language (HCL)
- Terraform configuration syntax is built around on blocks and arguments
- Each block has a type and label(s).
  - Blocks can have 0 or more labels.
- After block labels are curly braces, and within them are arguments and nested blocks.
- Single-line comments start with a `#` symbol (common) or a `//` symbol.
- Double-line comments start with `/*` and end with `*/`.

## Initializing Working Directors

- We need to initialize a working directory to use, download, and install providers.
- We run `terraform init` in our main directory to download and install our providers locally.
- This is similar to running Python's `pip` or node's `npm`.
- A directory `.terraform` and a file `terraform.lock.hcl` will be made after running `.terraform init`, though they may be hidden on Linux if they start with `.` so use `ls -a` to see them.
- `.terraform.lock.hcl` is a dependency file that captures the version of all Terraform providers you are using each time we run Terraform. This file is part of Terraform's strategy for defending against attacks that seek installing a modified version of a provider.
- `.terraform` is a local cache where Terraform stores some files for subsequent operations. Provider plugins are located in this directory.
- You can initialize a Terraform repository anytime since the `terraform init` command is idempotent, meaning it will have no effect if no changes are required.

## Authenticating to AWS

- Terraform uses access/secret keys to authenticate to an AWS account as a non-privileged user.
  - You can recover/find this as an admin user in the IAM dashboard, clicking your name, and then security credentials to find an access keys section.
  - The secret key cannot be recovered, it is only available on key creation.
- We add `access_key` and `secret_key` with their values in our `provider "aws"` block.
- We can also use environment variables so that we don't save our keys within the configuration files and avoid risking them to the version control system.
  - Open a terminal and use `export AWS_ACCESS_KEY_ID=""` and `export AWS_SECRET_KEY_ID=""` and `export AWS_DEFAULT_REGION=""` which remains until the end of the shell/terminal session. On Windows command prompt use `$Env:AWS_ACCESS_KEY_ID=""` and so on instead of `export`.

## Creating Resources (Part 1): AWS VPC

- We'll provision infrastructure in the following order:

1. VPC
2. Subnet
3. Route Table & Internet Gateway
4. Security Group (ACLs)
5. EC2 Instance

- When creating a `resource`, there's a common syntax to use:

```terraform
resource "<provider> <resource_type>" "local_name" {
  argument1 = value1
  argument2 = value2
  ...
}
```

- A resource type is implemented by a provider. So an `"aws_vpc"` resource is implemented by the `"aws"` provider.
- The resource type and local name together serve as an identifier for a given resource and must be unique within a module. Within the body are configuration arguments for the resource.
- Tags can be added to resources to show them in the AWS Management Console.
- Here's an example of VPC code:

```terraform
# Create a VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    "Name" = "Main VPC"
  }
}
```

## Terraform Plan and Apply

- `terraform plan` does not change resources but shows what would occur if we ran `terraform apply`.
- You can save a Terraform plan with `terraform plan -out=tfplan`, where `tfplan` can be any file name but `tfplan` is typical convention. We can run this plan using `terraform apply "tfplan"`.
- If we agree with the Terraform plan, we should run `terraform apply` to execute planned actions.
- If we run `terraform plan` again after running `terraform apply`, Terraform checks for any changes if needed.

## Formatting and Validating Configuration Files

- Terraform CLI provides a command to format and to validate syntax.
- Use `terraform fmt` to rewrite Terraform configuration files so they are readable and consistent.

### Naming Conventions

- Use `_` (underscore) instead of `-` (dash) everywhere. Prefer lowercase to uppercase.
- Do not repeat resource type in resource name.
  - `resource "aws_route_table" "public" {}`, not `resource "aws_route_table" "public_route_table" {}`

### Formatting and Validating

- You can format another directory of files by simply including the directory name as an argument, like `terraform fmt 01-aws`.
- `terraform fmt -diff` will display the differences/changes before rewriting the file, but **will overwrite**.
- `terraform fmt -diff -check` will display the differences/changes before writing the file, but **will not overwrite**.
- `terraform fmt -recursive` will reformat all the files within subdirectories.
- `terraform validate` validates the format and syntax within a directory. It includes argument names/types for resources in modules. The command runs checks to verify where the configuration is syntactically correct or not, and also checks if arguments have typos.
- `terraform plan` and `terraform apply` automatically validate the syntax before performing any work.

## Destroying Infrastructure with Terraform

### Destroying Infrastructure

- Run `terraform destroy`
- Remove resources from config

### `terraform destroy`

- `terraform destroy` checks for resources in the configuration files and deletes all of them with confirmation.
- To destroy specific resources only, use `terraform destroy -target resource_type.local_name`.
- You can pass `-auto-approve` when applying to auto confirm.
- `-target` is not for routine use.

### Remove resources from config

- You can simply remove resource blocks from the configuration files and then run `terraform apply`.
- You should prefer this option over `terraform destroy`.

## Replacing Infrastructure with Terraform

- Usually Terraform only updates your infrastructure if it does not match your configuration. However, there are cases when users have manually changed settings of a resource and they need to be recreated from scratch.
- If a user SSH into an EC2 instance that was made with Terraform and then made system malfunctions or other issues, using `terraform apply` again wouldn't replace the resource since the instance still exists. To force a replacement of a resource, use `terraform apply -replace="resource_type.local_name"`.
- Older versions of Terraform more commonly use `terraform taint resource_type.local_name`, followed by `terraform apply`.

## Creating Resources (Part 2): AWS Subnet

```terraform
# Create a subnet
resource "aws_subnet" "web" {
  vpc_id = aws_vpc.main.id
  cidr_block = "10.0.10.0/24"
  availability_zone = "us-east-1a"
  tags = {
    "Name" = "Web subnet"
  }
}
```

## Customizing Terraform Configuration with Variables (Part 1)

### Terraform Variables

- Input variables make the Terraform configuration more flexible by defining values that end users can define to customize the configuration.
- Think of variables as function arguments in a programming language.
- Enable a variable with a unique name:

```terraform
variable "unique_name" {

}
```

- Variables can have optional arguments, like `default`, `description`, and `type`.

```terraform
variable "vpc_cidr_block" {
  default = "10.0.0.0/16"
  description = "CIDR Block for the VPC"
  type = string
}
```

- Refer to a variable with `var.unique_name`.
- Variables will be asked for user input before `plan` or `apply`.
- Variable declarations can appear anywhere in configuration files, but it's easier to have them in a `variables.tf` file.

## Customizing Terraform Configuration with Variables (Part 2)

- You can use the `-var` command line option for replace values for variables in `terraform plan` and `terraform apply`.
- `terraform plan -var="variable_name=value"` and `terraform apply -var="variable_name=value"`
- You can use `-var` multiple times in a single command to set multiple variables.
- All `*.tf` files in a directory create a module (i.e. the root module).
- Terraform compiles all the `*.tf` files in a single directory as one module.
- `*.tfvars` and `*.tfvars.json` are variable definition file types which Terraform automatically loads if named `terraform.tfvars` or when specified with `-var-file` in the command line.
- Basically Terraform looks at `terraform.tfvars` or a `-var-file` variables file first before using any default variable values.
- Another way to define variables is to use environment variables but this has the least precedence. Terraform searches for environment variables with `TF_VAR_variable_name`.
- If a variable is not initialized and does not have a default value either, the user will be prompted to enter the value of the variable.

### Variable Precedence

1. `-var` and `-var-file`
2. `terraform.tfvars`
3. environment variables (`TF_VAR_*`)

- You can add variables within strings using `${}`.

## Creating Resources (Part 3): Default Route Table and Internet Gateway

### Route Table

- Route table = virtual router within your VPC.
- The route table that comes with your VPC controls the routing for all subnets that belong to the VPC and are not explicitly associated with any other route table.
- A route table contains routes (rules) that determine where the network traffic (from subnets or gateway) is directed.

### Internet Gateway

- An Internet Gateway connects a VPC with the Internet.
- An Internet Gateway routes internet traffic and performs network address translation for instances that have been assigned public IP addresses. Internet Gateways support IPv4 and IPv6.
- A subnet of a VPC that's associated with a route table that has a route to an Internet Gateway is known as a public subnet.
- A subnet of a VPC that's associated with a route table that does not have a route to an Internet Gateway is known as a private subnet.

## Security Groups and Firewall Configuration

- A security group operates at the instance level
- A network ACL operates with a subnet

### Security Groups

- Security groups are the fundamental of Network Security in AWS.
- Security groups control how traffic is permitted into or out of the EC2 instances (the firewall for EC2 instances or VMs).
- A Security Group supports only **allow** rules. All Intertnet traffic to a security group is implicitly denied.
- Security Groups are stateful, where any changes to an incoming rule will automatically be applied to an outgoing rule (Allow inbound traffic on port 80, automatically allows outbound traffic on port 80).

### Network ACL

- Network ACLs are the firewall of the VPC subnets.

## Launching EC2 Instances in the VPC

- AMIs have an AMI ID, and we'll need the IDs for Terraform but note that these IDs change frequently.
- The same image may have different IDs in different regions, and the same AMI may not be available in another region.

## Creating an SSH Key Pair for EC2

### SSH Public Key Authentication

- SSH Key Pair = public and private key used to authenticate to the EC2 instance.
- The public key is stored on your instance and the private key on the client.
- You can save as a `*.pem` file for OpenSSH connections.
- To authenticate to the instance using the private key, there should be more restrictive permissions. You can use `chmod 400 */ssh_key.pem` as if you don't set these permissions you'll get an error when trying to authenticate with the privatekey.

## Automatic SSH Key Pair Generation with Terraform

- `ssh-keygen -t rsa -b 2048 -C 'test key' -N '' -f ~/.ssh/test_rsa`
  - `-C` is a comment
  - `-N` provides the passphrase, and if the string that follows is empty then it won't encrypt the key
  - `-f` provides an alternate location for the keys

## Terraform Data Sources

### Data Sources

- Each provider may provide data sources alongside data types.
- Data sources are a kind of API that fetch dynamic data from cloud providers.
- Data sources are like queries (i.e. list of AMIs that change frequently or list of availability zones).

## Filtering AMIs using Data Source

- You can use a `data` block to use data sources.
- You can search for owner values for an AMI data source on the AWS Management Console when searching for AMIs.

## Query Data with Outputs

### Output Values

- Output values are used to export structured data about resources.
- Output values are like functions' return values.
- Root module can use outputs to print values at the terminal after running `terraform apply`.
- It's recommended to put output values in a configuration file named `outputs.tf`.
- You can find output value attributes when you run `terraform plan` or `terraform apply` which have the plus sign before them.
- Example `output` block:

```terraform
output "ec2_public_ip" {
  description = "The public IP address of the EC2 Instance"
  value = aws_instance.my_vm.public_ip
}
```

- Terraform stores output values in its state file, and we can run `terraform output` to view the output values.
- To query only one output by name, give the name of the output for example: `terraform output ec2_public_ip`.
- You can generate JSON formatted (machine-readable) output by simply adding the `-json` flag to the `terraform output` command for example: `terraform output -json`. You can then redirect this to a `*.json` file by adding `> outputs.json`.
- Output values can be designated as "sensitive" and Terraform won't output them on the console (i.e. passwords). You simply add the `sensitive = true` line to the `output` block.
- Terraform will not censor sensitive outputs when you query by name or add the `-json` flag.
- All output values are stored in the state file in clear text.

## Understanding Terraform State

### Terraform State

- Terraform stores state information about your managed infrastructure and configuration.
- State is used by Terraform to map real-world resources to your configuration, keep track of metadata, and to improve performance.
- The state is stored by default in a file called `terraform.tfstate`, but it can also be stored remotely which works better in a team environment.
- The state file will not exist until you have completed at least one `terraform apply`.
- The local state file is stored in JSON, but direct file editing is not recommended. Instead, Terraform provides some commands to perform basic modification of the state using the command line.
- The primary purpose of Terraform state is to store bindings between remote objects in the cloud and resources declared in the configuration file.
- By default a backup of your state file is written in `terraform.tfstate.backup` in case your state file is lost or corrupted.
- Prior to an operation often Terraform refreshes the state to check if the resources in the local configuration are in the cloud.
- When Terraform creates objects, it records its identity in the state and removes the bindings when destroyed.
- The state file initially gives information about Terraform like its version. After the outputs come the resources that are all the resources managed by Terraform. Resources have key-value pairs such as:
  - `mode` refers to the type of resource it creates. `managed` or `data` for data source.
  - `type` refers to the resource type Terraform manages.
  - `name` is the given local name of the resource
  - `provider` is the provider of the resource
  - `instances` contains the attributes of the resource
- Terraform also keeps track of resource dependencies so it knows how to properly delete resources in order

## The Terraform State Command

- We can view resources in the state without directly interacting with the state using `terraform show`
- To find a specific resource, we can parse the output for example: `terraform show | grep -A 20 aws_vpc`
- You can get a list of all resource types and local names with `terraform state list`.
- You can get all the attributes of a resource using `terraform state show resource_type.local_name`
- You can replace a resource using the `-replace` flag with either `terraform plan` or `terraform apply` without destroying/creating the entire infrastructure.
- Storing state remotely is more secure.

## Running Commands on EC2

- You can set a startup script for an EC2 instance through Terraform.
- There's various ways to run commands on EC2:

1. `user_data` - an attribute which can be given command/script values
2. `cloud-init` - industry standard for cloud instance initialization
3. Packer - open-source devops tool by Hashicorp to create images from a single JSON file
4. Provisioners

## Running Commands Using User Data

### User_Data

- Passes the commands to the cloud provider which runs them on the instance.
- Idiomatic and native to the cloud provider.
- Viewable in the AWS console and is the recommended way.
- We can write a multiline bash script with `<<EOF` and `EOF`.

```bash
  user_data = <<EOF
  #!bin/bash
  sudo yum -y update && sudo yum -y install httpd
  sudo systemctl start httpd && sudo systemctl enable httpd
  sudo echo "<h1>Deployed via Terraform</h1>" > /var/www/html/index.html
  EOF
```

- Run the commands directly on the VM first before running `terraform apply` to make sure they work.
- We can run a script on an instance called an entry script since it runs at boot time. We put the commands in a `*.sh` file and use it as a value for `user_data`.

## Provision Infrastructure with Cloud-Init

### Cloud-Init

- The standard for customizing cloud instances.
- It runs on most Linux distributions and cloud providers.
- cloud-init can run "per-instance" or "per-boot" configuration.
  - Determines if the current boot is the first boot of the instance or not.
  - Should run all per-instance first-boot configuration and then on subsequent boots only subsequent boot configuration.
- Cloud-init allows you to pass a shell script or a template to your instance that installs or configures the VM to your specifications.
- A `*.yaml` file can be used for configuration.
- You can import a resource into the state that already exists remotely with `terraform import resource_type.local_name remote_name`

## Terraform Provisioners

### Provisioners

- `user_data` is native to the cloud provider and provisioners are native to Terraform.
- Provisioners are a last resort and should be used with care. This is due to a large amount of complexity with this method.
- Usually Terraform connects itself with SSH to the instance and runs commands.
- `self` is the provisioner's parent resource and has all that resource's attributes.

## Terraform Troubleshooting and Logging

- Run `terraform validate` to check for errors.

### Troubleshooting

- Enable detailed logging by setting the environment variable `TF_LOG` to TRACE, DEBUG, INFO, WARN, or ERROR, example: `export TF_LOG=TRACE`
- Saving output to a file: `export TF_LOG_PATH=terraform.log`
- You can generate logs from the core application and the Terraform provider separately by setting `TF_LOG_CORE` or `TF_LOG_PROVIDER` to the appropriate log level.
