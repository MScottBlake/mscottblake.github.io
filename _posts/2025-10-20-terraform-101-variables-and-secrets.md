---
title: "Terraform 101: Variables and Secrets"
date: 2025-10-20 21:33:00 -0400
categories:
  - Terraform
series: "Terraform 101"
series_order: 3
tags:
  - Infrastructure as Code
  - IaC
  - Terraform
  - Variables
  - Apple School Manager
  - Apple Business Manager
header:
  teaser: /assets/images/posts/HashiCorpTerraform.png
  og_image: /assets/images/posts/HashiCorpTerraform.png
---

_This post is part of my Terraform 101 series, exploring how MacAdmins can use Infrastructure as Code to manage Apple environments._

Previously: [Getting Started](/blog/2025/10/14/terraform-101-getting-started/)

Up Next: [Resources and Data Sources](/blog/2025/10/28/terraform-101-resources-and-data-sources/)

---

In the last post, we set up a basic Terraform project and configured the [axm](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs) provider to connect to Apple School or Business Manager.

That worked fine for a quick test, but we hardcoded sensitive information directly into our configuration files. API credentials, sensitive file paths, and other secrets should not be in version control.

In this post, we'll clean that up by using variables, `.tfvars` files, and environment variables. These are essential for making your configurations reusable and secure.

## Why Variables Matter

Terraform variables let you make your code flexible without hardcoding values. Think of them like placeholders: you define a variable once, and then reference it throughout your configuration.

Without variables, you'd have to manually edit credentials, region names, or identifiers in multiple files whenever they change. With variables, you just change one place.

## Declare a Variable

In most programming languages, you need to define a variable before you can use it. For example, in Java, you would add something like `int variable_name = value;` before you could use `variable_name` in the code.

In Terraform, the concept is the same, but the syntax is different. We use the _variable block_.

When you define a variable, you are telling Terraform that a given `variable_name` is available for use. Optionally, you can add more data such as a `type` (string, integer, etc.), `description`, `default` value, and more.

Here's what that looks like in practice:

```hcl
variable "database_password" {
  description = "The password to use to connect to the database."
  type        = string
  sensitive   = true
}
```

The official documentation describes the [variable block](https://developer.hashicorp.com/terraform/language/block/variable) in detail. I recommend a quick glance to get familiar with the arguments that I did not list.

## Defining Variables

Now that we know all about the _variable block_, let’s update our project. We'll start by creating a new file called `variables.tf` within the same project directory we used in the last post.

```sh
touch variables.tf
```

Inside it, declare your variables:

```hcl
variable "client_id" {
  description = "Apple Business Manager API Client ID"
  type        = string
}

variable "key_id" {
  description = "Apple Business Manager API Key ID"
  type        = string
}

variable "private_key_path" {
  description = "Path to the Apple Business Manager API private key (.pem) file"
  type        = string
  sensitive   = true
}

variable "scope" {
  default     = "business.api"
  description = "API scope: business.api or school.api"
  type        = string
}
```

> 📝 Note that this file does not contain the actual values of your variables. It only defines each variable's metadata. We'll provide the values soon.

The updated project directory should now contain:

```sh
terraform/
├── providers.tf
├── terraform.tf
└── variables.tf
```

## Updating the Provider Configuration (providers.tf)

Before:

```hcl
provider "axm" {
  client_id   = "BUSINESSAPI.abcdef12-3456-4789-abcd-ef1234567890"
  key_id      = "98765432-dcba-4321-9876-543210fedcba"
  private_key = file("/path/to/private_key.pem")
  scope       = "business.api"
}
```

After:

```hcl
provider "axm" {
  client_id   = var.client_id
  key_id      = var.key_id
  private_key = file(var.private_key_path)
  scope       = var.scope
}
```

Now Terraform knows to look for these values when you run a command.

## Providing Values

There are several ways to provide values for defined variables. Let’s go over the most common and secure options.

### Command Line Flags

You can provide variables directly in your command line:

```sh
terraform apply -var "client_id=BUSINESSAPI.abcdef12-3456-4789-abcd-ef1234567890" -var "key_id=987654..."
```

This may be fine for quick tests, but not ideal for longer-term projects. It is cumbersome to maintain and the values can show up in shell history, causing security concerns. This approach is not recommended for anything sensitive.

### TFVars File

Create a file named `terraform.tfvars` in your project directory. Terraform will automatically loads this file when you run `terraform plan` or `terraform apply`.

Inside it, add your unique values that we retreived from the Apple School or Business Manager console in [Terraform 101: Getting Started](/blog/2025/10/14/terraform-101-getting-started/).

```hcl
client_id        = "BUSINESSAPI.abcdef12-3456-4789-abcd-ef1234567890"
key_id           = "98765432-dcba-4321-9876-543210fedcba"
private_key_path = "/Users/yourname/Documents/private_key.pem"
scope            = "business.api"
```

> ⚠️ **Important**: Add `*.tfvars` files to your `.gitignore`. These files should not be committed to version control.

```sh
echo "terraform.tfvars" >> .gitignore
```

Using `.tfvars` is a great approach. It allows you to have unique files for each environment. For instance, instead of `terraform.tfvars`, you could have two files names `development.tfvars` and `production.tfvars`. These files would not be auto loaded though. Terraform auto loads `.tfvars` files only when they are named `terraform.tfvars` or `*.auto.tfvars`. If you rename the file, you use the `-var-file` argument to specify the file to use.

```sh
terraform plan -var-file=development.tfvars
terraform apply -var-file=development.tfvars
```

### Environment Variables

One of the coolest tidbits about variables is that Terraform automatically detects environment variables prefixed with `TF_VAR_`.

```sh
export TF_VAR_client_id="BUSINESSAPI.abcdef12-3456-4789-abcd-ef1234567890"
export TF_VAR_key_id="98765432-dcba-4321-9876-543210fedcba"
export TF_VAR_private_key_path="/Users/yourname/Documents/private_key.pem"
```

When you run Terraform, it picks these up automatically as long as they are declared. You do not need to pass them in manually.

## Validating Variables

Terraform lets you define `validation` rules to catch bad input early. Each rule contains a `condition` and an `error_message`. For example:

```hcl
variable "scope" {
  default     = "business.api"
  description = "API scope: business.api or school.api"
  type        = string

  validation {
    condition     = contains(["business.api", "school.api"], var.scope)
    error_message = "Scope must be either 'business.api' or 'school.api'."
  }
}
```

The logic above checks the value to make sure it contains one of two values: either `business.api` or `school.api`. This provider only allows those two values, so it makes sense to validate the variable to prevent failures. Each variable is going to be different and you may or may not have the need to validate the inputs.

If the condition fails, Terraform will show the designated error message and then stop processing.

I recommend that you update the `variables.tf` file with this new `scope` validation definition, but it's not necessary.

## Variable File Recap

Do not confuse `terraform.tfvars` files with `variables.tf` files. They are completely different and they serve different purposes.

- `variables.tf` are files where all variables are declared.
- `terraform.tfvars` are files where the variables are assigned a value.

## Next Post

By moving sensitive information out of your configuration and into variables, you’ve made your project cleaner and more secure.

In the next post, we’ll start using resources and data sources. These are the primary building blocks of every Terraform project. We’ll create real objects through the Apple APIs and learn how Terraform reads existing ones.

Up Next: [Resources and Data Sources](/blog/2025/10/28/terraform-101-resources-and-data-sources/)

---

## Series Index

1. [Terraform 101: Introduction](/blog/2025/10/14/terraform-101-introduction/)
1. [Terraform 101: Getting Started](/blog/2025/10/14/terraform-101-getting-started/)
1. Terraform 101: Variables and Secrets
1. [Terraform 101: Resources and Data Sources](/blog/2025/10/28/terraform-101-resources-and-data-sources/)
1. [Terraform 101: Command Line Interface](/blog/2025/11/05/terraform-101-command-line-interface/)
