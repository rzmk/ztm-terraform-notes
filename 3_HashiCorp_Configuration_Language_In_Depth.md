# HashiCorp Configuration Language (HCL) In Depth

## Terraform Simple Types: Number, String, Bool

### Variables in Depth

In Terraform and HCL, there are simple and complex types. A simple type represents a single value, while a complex type groups several values into a single one.

- Simple: `number`, `string`, `bool`, `null`
- Complex:
  - Collection types: `list`, `map`, `set`
  - Structural types: `tuple`, `object`

Types are useful when defining variables by using `type = number` (where `number` can be any simple type) to ensure type.

## Complex Types

- Collection: `list`, `map`, `set`
  - it groups values of the same type
- Structural: `tuple`, `object`
  - it groups values of different types

## The Count Meta-Argument

By default, Terraform configures one resource for one resource block. Terraform allows you to work with multiple with `count` and `for_each`.

- `count` is used to manage similar resources
- `count` and `for_each` are looping techniques

You can use `count` with every resource type.

You can use `element(list, index)` where `list` is a list of values and `index` is the index of the specific element you want from the list. You can also use `length(value)` to get the length of a value like a list. This is useful when working with `count`:

```terraform
variable "users" {
  type = list(string)
  default = [ "demo-user", "admin1", "john" ]
}

resource "aws_iam_user" "test" {
  name = "${element(var.users, count.index)}"
  path = "/system/"
  count = "${length(var.users)}"
}
```

## The `for_Each` Meta-Argument

When you run `terraform apply` after removing elements in the middle of a list, an error can occur. `count` is sensitive to anything in a list order.

The `for_each` meta-argument accepts a `map` or a `set` of strings and creates an instance for each item in the `map` or `set`. Sets and Maps don't allow duplicates and are unordered.

You can use `toset(value)` to convert a value like a `list` to a `set`. You can refer to a `for_each` value by using `each`, like getting `each.key` for the values of element in the list.

This is why it may be better to use `for_each` rather than `count` so there's no errors when manipulating values/lists and things in ordered format.

## Using Dynamic Blocks

We can use dynamic blocks to remove repeated code. You can generate nested blocks in `resource`, `data`, `provider`, and `provisioner` blocks. The `for_each` object provides the collection (i.e. list or map) to iterate over in the dynamic block. To access an element of the list at the current iteration, use the name of the dynamic block followed by `.value`, and you can use `.key` if it is a map instead. The nested `content` block defines the body of each generated block.

```terraform
dynamic "ingress" {
  for_each = var.ingress_ports
  content {
    from_port = ingress.value
    to_port = ingress.value
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

You can use `iterator` to change the name you refer to for the current collection value instead of having to use the block name.

```terraform
dynamic "ingress" {
  for_each = var.ingress_ports
  iterator = iport
  content {
    from_port = iport.value
    to_port = iport.value
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Notice the difference between the previous two code blocks in how the current variable is named for use.

Note that using dynamic blocks can make it harder to read and maintain, so try using them only when needed to hide details and build a clean user interface.

## Conditional Expressions

You can conditionally build resources by commenting or using a count. However, it's better to parameterize our code. We can make a variable like `istest` with `type = bool` and then change the value in `terraform.tfvars`. Then we can use a conditional syntax for values using the ternary operator. The two result values may be of any type but they must both be of the same type.

```terraform
resource "aws_instance" "test-server" {
  ami = "ami-05cafdf7c9f772ad2"
  instance_type = "t2.micro"
  count = var.istest == true ? 1:0
}

resource "aws_instance" "prod-server" {
  ami = "ami-05cafdf7c9f772ad2"
  instance_type = "t2.large"
  count = var.istest == false ? 1:0
}
```

## Terraform Locals

### Locals

- Local values (locals) are named values that you can refer to in your configuration. They help you simplify your configuration and write more readable code.
- Compared to variables, locals do not change values during a Terraform run, and are not submitted by users but calculated inside the configuration.
- Locals are available only in the current module. **They are scoped locally!**
- You can put locals in `main.tf` if there's not many, or in `variables.tf` or `locals.tf`.
- You can start defining locals in a `locals` block.

```terraform
locals {
  owner         = "DevOps Corp Team"
  project       = "Online Store"
  cidr_blocks   = ["172.16.10.0/24", "172.16.20.0/24", "172.16.30.0/24"]
  common-tags   = {
    Name        = "dev"
    Environment = "development"
    Version     = 1.10
  }
}
```

- When using locals, you use `local.key` where `key` is the name of the local.

```terraform
resource "aws_subnet" "dev_subnets" {
  vpc_id            = aws_vpc.dev_vpc.id
  cidr_block        = local.cidr_blocks[0]
  availability_zone = "eu-central-1a"
  tags              = local.common-tags
}
```

## Intro to Terraform Built-in Functions

- Terraform has [many built-in functions](https://www.terraform.io/language/functions).
- There are no user-defined functions in Terraform.
- You can try the Terraform console with `terraform console`.
- One example that may be useful is: `formatdate("DD-MM-YYYY hh:mm", timestamp())`

## Using Splat Expressions in Terraform

- A splat expression captures all items in a list that share a common attribute.
- The `*` symbol iterates over all the elements in a given list and returns the information based on the shared attribute.
- A splat expression can only be used with lists, sets, and tuples. They cannot be used with maps and objects.

The following code would output a list.

```terraform
output "private_addresses" {
  value = aws_instance.server[*].private_ip
}
```
