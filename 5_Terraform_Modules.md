# Terraform Modules

## Intro to Terraform Modules

### Issues with Complex Configurations

- Updates may cause problems in different parts of your configuration.
- You'll repeat parts of your configuration.
- Sharing is not easy.

### Terraform Modules Advantages

- DRY = "Do Not Repeat Yourself"
- Organize configuration.
- Encapsulate configuration.
- Re-use configuration.
- Provide consistency.
- Ensure best practices.
- Think of modules as functions in programming languages.
- A module is a set of configuration files in a single directory.

### Types of Modules

- Local
- Remote

### Terraform Modules Importance

- Modules are the key to writing reusable, maintainable, and testable Terraform code.

## Creating Your First Module

- The root module has the resources defined in the `.tf` files in the main directory.
- You can make a `modules` directory and then child modules like `modules/ec2`. A typical structure for a new Terraform module is to have `main.tf`, `variables.tf`, and `outputs.tf`. You can also have a `README.md` and a `LICENSE` file.
- It's recommended for each (child) module to declare its own `required_providers` block inside a `terraform` block so Terraform knows theres a single provider that's compatible with all modules in the Terraform configuration. A child module automatically inherits its providers from its parents so it's not necessary to add these blocks to child modules.
- You can move the EC2 `resource` block into the EC2 module's `main.tf`.
- Use the `module` block to import a module, and use the `source` object with a relative path to the module's directory.
- You must run `terraform init` to initialize modules.

```terraform
module "myec2" {
  source = "../modules/ec2"
}
```

## Parameterizing Modules (Part 1)

- Set a `variables.tf` in your child modules so that you can edit the values in your main/root module within the `module` block by directly naming the variables and editing their values. This way you can use child modules like a template.

## Parameterizing Modules (Part 2)

- It's better to declare variables for the arguments of the child module in `variables.tf` and `terraform.tfvars` files rather than directly in the root module (i.e. `main.tf`).
- It's good to name the variables the same as the argument names.

## Refactoring the Infrastructure Using Models

- It's good to refactor your code with modules.
- If making a VPC, make a directory `modules/vpc` with `vpc/main.tf`, `vpc/outputs.tf`, and `variables.tf`.
- Parameterize variables that are defined within the module and give them default values.
- Call the child module within the root module by using the `module` block and the `source` argument. Run `terraform init`.
- The child module must use `output` blocks to expose/export values/variables to the root module.

## Accessing Child Module Output Values

- You can have a calling module access an `output` value of a child module by using `module.local_name.output_variable`, where `local_name` is the name you provide in the `module` block and `output_variable` is the name defined in the child module of the `output` block.
- In `outputs.tf` of a child module, use the `output` block along with `value` arguments to expose values to the root module.

Child VPC module exporting VPC ID:

```terraform
output "main_vpc_id" {
  value = aws_vpc.main_vpc.id
}
```

Root module accessing child VPC module VPC ID:

```terraform
vpc_id = module.vpc.main_vpc_id
```

- Child modules cannot access each other's variables, so you can set empty variables for them to be filled in the root module later.
- You can output an entire resource using `resource_type.local_name` in the `value` argument.

## Intro to Terraform Registry

- The Terraform Registry provides [a public registry](https://registry.terraform.io/browse/modules) that has official Terraform modules and community modules, and a private registry that is available as part of Terraform Cloud and hosts modules used internally within an organization.
- You can use sample `module` blocks, look at inputs/outputs (arguments/outputs), and a resources section that lists all resources that the resource can create.
- You can use outputs with `module.resource_name.output_name`.
- When working with modules, you must first run `terraform init` to download and initialize modules. Terraform uses Git to download modules.
