# Terraform Remote State and Security

## Backends and Remote State Management

- Terraform is a stateful application, keeping track of everything it does in your cloud environment so that changes can be made.
- The State File keeps track of everything Terraform does.
- `terraform.tfstate` is the state file and there's also a `terraform.tfstate.backup`.
- In production when working in a team, using a local state file brings complications.
- Local State is not good for production environments when working in a team.

### Concurrency

- It needs a locking mechanism to avoid running into race conditions.
- Solution: store the state remotely using the correct backend.
- Backends define how operations are executed and where the Terraform State is stored.
- Terraform supports storing state in many backends like Kubernetes, Postgres, S3, etc.

## Terraform Remote State on Amazon S3

- We can make a bucket on Amazon S3 and configure Terraform to use it for remote state.
- You must enable bucket versioning to keep multiple versions of the state file so you can recover from corruption/failure.
- Encryption is necessary if you store sensitive data in the state file, like passwords or keys. However, anyone on your team who has access to the S3 bucket can read the state file in an unencrypted form. This only offers data at-rest and in-transit protection.
- Configure the Terraform remote state from within the S3 bucket.
- You can use a `backend` block within a `terraform` block to specify your bucket to use in a `backend.tf` file or even in `main.tf`. `bucket` represents the name of the bucket, `key` represents the name of the state file in the bucket, and `region` is your bucket's AWS region. You need to copy your access key and secret key as values in `backend` too.

```terraform
terraform {
  backend "s3" {
    bucket = "my_terraform_bucket"
    key = "s3_backend.tfstate"
    region = "us-east-1"
    access_key = ""
    secret_key = ""
  }
}
```

## Implementing State Locking with DynamoDB

- State locking prevents concurrent runs on Terraform against the same state.
- Running Terraform locally has locking run by default, which protects the state from changing by multiple users (terminals) at the same time.
- Use `terraform init -migrate-state` to migrate the remote state back to local state or vice-versa, making sure to comment/uncomment your code.
- To enable state-locking on the S3 backend, use the NoSQL database DynamoDB.
- On DynamoDB, create a table `s3-state-lock` and a required partition key `LockID`.
- In the `backend` block, add `dynamodb_table = "s3-state-lock"` where `s3-state-lock` is whatever you named your table.
- The remote state was changed so we must run `terraform init -reconfigure` to store the current configuration to the remote state. Now state locking should work.

## Terraform Remote State on Terraform Cloud

- Terraform Cloud is an alternative to S3 for storing/sharing remote state.
- To use Terraform Cloud for remote state, add a `cloud` block in `terraform`. Note that you cannot have a `backend` block if you have a `cloud` block.
- In the `cloud` block, add an `organization` block with the name of your organization you set up on `app.terraform.io`, note that this organization must exist.
- In the `cloud` block, make a `workspaces` block with any name but if the workspace exists then there should be no state file in it.
- To see the current workspace, run `terraform workspace show`.

```terraform
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~> 3.0"
    }
    cloud {
      organization = "master-terraform"
      workspaces {
        name = "DevOps-Production"
      }
    }
  }
}
```

- Run `terraform login` to connect to Terraform Cloud and create an API token to generate an authentication token. Copy the token and paste it into the `terraform login` prompt to continue. Terraform will hide the token for security so you won't see it when you paste it. You'll get a "Welcome to Terraform Cloud!" message, and it creates a `credentials.tfrc.json` file in the home directory, so keep it secure since it provides access to your Terraform Cloud for your organization.
- After authenticating to Terraform Cloud, run `terraform init` to initialize the provider(s) and Terraform Cloud.
- The run status of your workspace should be "Applied" on the Terraform Cloud website, meaning the state file is saved in the cloud.

## Managing Secrets in Terraform

- Don't store secrets in plaintext.
- Even if you securely store Terraform secrets, you must securely store your Terraform state file. Keep your state safe from versioning tools too.
- No matter what technique you use to encrypt secrets, they will still end up in plaintext in the Terraform state. By default `terraform.tfstate` is made when you run `terraform apply`. You should store Terraform state in a backend that supports encryption instead of locally. Amazon S3, GCS, Azure Blob Storage, Terraform Cloud are safe options.

### Terraform Security

- Don't store secrets in plain text!
- Store Terraform State in a backend that supports encryption.

## Storing Secrets Using Variables

- Remember you can add `sensitive = true` to prevent Terraform from showing the values in the CLI. Don't set the default value for secret variables either.
- You can also set the Terraform secret as an environment variable prefixed with `TF_VAR`. For example `export TF_VAR_db_password="mypass12345"`. Note that the entire command is stored in your bash history. You can prevent this by adding whitespace before the command name (spaces before typing your command). However the value of the variable it'll still be in your state file, so you should store your state remotely and encrypt it.

## Storing Secrets Securely Using AWS Secrets Manager

- We can use secret stores where secrets/passwords are stored in a dedicated secret store that is designed specifically for storing sensitive data and controlling access to it. This secret store enforces encryption and strict access control, and everything will be defined in the code itself.
- Some popular secret stores are HashiCorp Vault (open-source and cross-platform), AWS Secrets Manager, and GCP Secret Manager.
- On the AWS Management Console, search Secrets Manager. Secrets are local resources so the region in which your infrastructure operates must be selected (same as `provider` region).
- Click store a new secret and click "Other type of secret".
- Click Plaintext and add key/value pairs in a JSON format, like `"password": "mysecret123"` with commans between pairs for more secrets.
- Give the secret a name and click "Store" to save your secret.
- The AWS `provider` providers a data source `aws_secretsmanager_secret_version` that retrieves information including secret values. `secret_id` is an argument which is the name of the secret you've stored.
- Each secret store provides its own data source.
- The keys in the credentials are the names of the keys in your secret, like `username` in `local.db_creds.username`.

```terraform
data "aws_secretsmanager_secret_version" "creds" {
  secret_id = "db_credentials"
}

locals {
  db_creds = jsondecode(data.aws_secretsmanager_secret_version.creds.secret_string)
}

resource "aws_db_instance" "default" {
  allocated_storage = 10
  engine = "mysql"
  engine_version = "5.7"
  instance_class = "db.t3.micro"
  name = "mydb"

  username = local.db_creds.username
  password = local.db_creds.password
  parameter_group_name = "default.mysql5.7"
  skip_final_snapshot = true
}
```

- AWS Secrets Manager is not free, it's free for 30 days but then you must pay.

### Key Takeaways

1. Do not store secrets in plain text.
2. Use environment variables, encrypted files, or a secret store to securely store and retrieve secrets.
3. Use a remote backend that supports encryption such as Amazon S3 or Terraform Cloud.
