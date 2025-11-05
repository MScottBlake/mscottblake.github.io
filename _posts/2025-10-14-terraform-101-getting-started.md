---
title: "Terraform 101: Getting Started"
date: 2025-10-14 19:17:00 -0400
categories: ["Terraform"]
series: "Terraform 101"
series_order: 2
tags:
  - Infrastructure as Code
  - IaC
  - Terraform
  - Apple School Manager
  - Apple Business Manager
  - Homebrew
header:
  teaser: /assets/images/posts/HashiCorpTerraform.png
  og_image: /assets/images/posts/HashiCorpTerraform.png
---

*This post is part of my Terraform 101 series, exploring how MacAdmins can use Infrastructure as Code to manage Apple environments.*

Previously: [Introduction](/blog/2025/10/14/terraform-101-introduction/)

Up Next: [Variables and Secrets](/blog/2025/10/20/terraform-101-variables-and-secrets)

---

## Getting Started

I find that it is easiest to learn new tools and concepts when I have a project. For this post, we're going to use Neil Martin's [axm Terraform Provider](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs) as an example. This provider will allow us to interact with the Apple School/Business Manager API.

Before we can do that, we're going to need some credentials.

1. Login to [Apple School Manager](https://school.apple.com/) or [Apple Business Manager](https://business.apple.com/).
1. Click your name in the lower left to open the menu and select *Preferences* and then *API*.
1. Find the + icon and select *Create API Account*.
1. Provide a label such as "Terraform" that will describe the purpose of this account and click *Create*.
1. Click *Generate Private Key* and a `.pem` file will be downloaded by your browser.
1. Find the account you just created and click *Manage*. Note the *KeyID* and *ClientID*. You will need these values later.

## Install Terraform

The easiest way to install Terraform is to use [Homebrew](https://brew.sh/).

First, install the HashiCorp's official tap containing all of their Homebrew packages.

```sh
brew tap hashicorp/tap
```

Then, install Terraform from hashicorp/tap/terraform.

```sh
brew install hashicorp/tap/terraform
```

Optionally, you can install tab completion in your shell.

```sh
terraform -install-autocomplete
```

Lastly, verify that the installation is working correctly.

```sh
terraform -version
```

For more detailed installation instructions, check out Hashicorp's [installation documentation](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli).

## Terraform Project

A Terraform project is any directory containing `.tf` files that has been initialized with the `terraform init` command, which sets up Terraform caches and a local state file.

With that knowledge, let's create a directory called *terraform*. We'll use this as our working directory moving forward.

```sh
mkdir terraform
cd terraform
```

Now we'll create a `.tf` file. This file can be named whatever you want. For now, let's call it `terraform.tf`.

```sh
touch terraform.tf
```

Open this file in your editor of choice so that we can build out its contents.

```hcl
terraform {
  required_providers {
    axm = {
      source  = "neilmartin83/axm"
      version = "~> 1.2"
    }
  }

  required_version = ">= 1.13"
}
```

Let's talk about what this means. We have a `terraform {}` block that defines the terraform CLI version requirements (`required_version`) as well as the provider requirements (`required_providers`) for this project.

You'll notice that there is a weird operator (`~>`). This operator is equivalent to `>= 1.2.0, < 2.0.0`. It can be `1.3` or `1.10.2` but not `2.0`. See the Terraform documentation for more information about [version constraints](https://developer.hashicorp.com/terraform/language/expressions/version-constraints).

## Provider Configuration

Terraform doesn't care how the configurations are organized as long as they are `.tf` files and all in the same directory. The entire configuration could be in a single file, but that gets pretty difficult to read as the project gets bigger.

I like to separate out the `terraform` and `provider` blocks into their own files, so let's create a new `providers.tf` file.

```sh
touch providers.tf
```

The contents of that file will depend on the provider you are using. In Each provider configuration will be different. Basically, this is where you define authentication for the system you are trying to integrate.

In our example, we are using the axm provider, so the contents of `providers.tf` will depend on your credentials from Apple School Manager or Apple Business Manager.

```hcl
provider "axm" {
  client_id   = "BUSINESSAPI.abcdef12-3456-4789-abcd-ef1234567890"
  key_id      = "98765432-dcba-4321-9876-543210fedcba"
  private_key = file("/path/to/private_key.pem")
  scope       = "business.api"
}
```

> ⚠️ It is recommended that you do not enter secret values here. We will fix this in the next post when we talk about variables.

Alternatively, you can set environment variables for `AXM_CLIENT_ID`, `AXM_KEY_ID`, `AXM_PRIVATE_KEY`, and `AXM_SCOPE`. Any value that is set in those environment variables can be omitted from the `provider` block. You can even define it with no values if everything is being set by environment variable.

```hcl
provider "axm" {}
```

The `scope` value will depend on which portal you are using. Enter `school.api` for Apple School Manager and `business.api` for Apple Business Manager.

At this point, your project directory should contain:

```sh
terraform/
├── providers.tf
└── terraform.tf
```

## Initialization

Now that we have `terraform {}` and `provider {}` blocks, it's time to initialize the terraform directory.

```sh
terraform init
```

After running `terraform init`, you’ll see a new `.terraform` directory. This stores plugin binaries and local state information. You shouldn't need to modify anything within this directory.

## Next Post

At this point, you have Terraform installed, configured, and ready to talk to Apple School or Business Manager.

In the next post, we’ll introduce variables: the key to making your configurations reusable, modular, and secure.

Up Next: [Variables and Secrets](/blog/2025/10/20/terraform-101-variables-and-secrets)

---

## Series Index

1. [Terraform 101: Introduction](/blog/2025/10/14/terraform-101-introduction/)
1. Terraform 101: Getting Started
1. [Terraform 101: Variables and Secrets](/blog/2025/10/20/terraform-101-variables-and-secrets/)
1. [Terraform 101: Resources and Data Sources](/blog/2025/10/28/terraform-101-resources-and-data-sources/)
1. [Terraform 101: Command Line Interface](/blog/2025/11/05/terraform-101-command-line-interface/)
