---
title: "Terraform 101: Command Line Interface"
date: 2025-11-05 18:47:00 -0400
categories:
  - Terraform
series: "Terraform 101"
series_order: 5
tags:
  - Infrastructure as Code
  - IaC
  - Terraform
  - Command Line Interface
  - CLI
  - CI/CD
  - Apple Business Manager
  - Apple School Manager
  - axm
header:
  teaser: /assets/images/posts/HashiCorpTerraform.png
  og_image: /assets/images/posts/HashiCorpTerraform.png
---

*This post is part of my Terraform 101 series, exploring how MacAdmins can use Infrastructure as Code to manage Apple environments.*

Previously: [Resources and Data Sources]({% post_url 2025-10-28-terraform-101-resources-and-data-sources %})

---

In the last post, we looked at **resources** and **data sources**. Now let's see how those declarations come to life
using the **Terraform Command Line Interface (CLI)**.

The CLI is how you interact with Terraform: initializing a project, previewing changes, applying updates, and cleaning
up resources. It's also where you'll see Terraform's declarative nature in action by analyzing the difference between
your configuration and reality, then taking the steps needed to bring them in sync.

---

## Understanding the Terraform Workflow

Terraform's CLI workflow revolves around three main phases:

1. Write – Describe your infrastructure in `.tf` configuration files.
1. Plan – See the changes Terraform will do (like a dry run).
1. Apply – Make the changes described above.

You'll use the CLI for each phase, and Terraform will guide you through what's about to happen.

### `terraform init`

Before you can do anything, you must initialize your project:

```sh
terraform init
```

This command:

- Downloads the provider plugins defined in your configuration (like `axm`).
- Sets up local caches and directories (e.g. `.terraform/`).
- Prepares your working directory for future commands.

You'll typically only need to run `init` once per project. It's also needed any time you add or upgrade providers.

If you've been following along with this Terraform 101 series, we already ran this command after installing terraform.

### `terraform plan`

This is where Terraform starts showing off its declarative power. `plan` looks at your configuration and your current
environment, then figures out what needs to happen to reach your declared state.

```sh
terraform plan
```

Terraform will output a summary of the actions it intends to take. For example:

```diff
Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # axm_device_management_service.iru will be updated in-place
  ~ resource "axm device_management_service" "iru" {
      ~ device ids = [
          - "FAKE000ABC123"
          + "FAKE222GHI789",
            # (2 unchanged elements hidden)
        ]
        id = "FAKE0000111122223333444444444444"
    }

PLan: 0 to add, 1 to change, ø to destroy.
```

Lines starting with `+` represent resources to be created, `-` represent deletions, and `~` represent changes. Nothing
actually happens during a plan. It's a dry run.

This is your chance to double-check Terraform's understanding before approving any changes.

### `terraform apply`

Once you've reviewed the plan, it's time to apply it.

```sh
terraform apply
```

Terraform will:

- Re-run the plan (to ensure nothing changed in the meantime).
- Prompt for confirmation.
- Perform the actions needed to reach the declared state.

When complete, you'll see a summary.

```text
Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

At this point, your declared configuration matches the real environment.

### Saving a Plan File

Usually, you'll want to make sure that the configuration you reviewed with `plan` is *exactly* the same one that gets
used when you `apply`. To do that, you can save the plan output to a file.

```sh
terraform plan -out=plan.tfplan
```

Later, instead of re-running the plan (which could pick up new changes), you can apply exactly what you reviewed.

```sh
terraform apply "plan.tfplan"
```

This workflow is especially important in automated pipelines or compliance workflows. It guarantees that the plan you
approved is the one being executed.

### `terraform show`

After applying, you can inspect the current state of your infrastructure with:

```sh
terraform show
```

This prints out the values Terraform knows about, including outputs and resource attributes.
It's a great way to verify that your resources exist and contain the expected data.

### `terraform destroy`

Sometimes you need to remove everything your configuration created. That's where `destroy` comes in:

```sh
terraform destroy
```

Terraform will generate a plan showing what it intends to remove, then prompt you for confirmation before proceeding.

This is particularly useful when testing new configurations or performing a full teardown of a development environment.

> ⚠️ **Be careful!** This command permanently deletes the resources managed by your configuration. Always double-check which environment you're in.

## Additional Helpful Commands

Here are a few more CLI commands you'll use regularly:

- `terraform fmt` — Formats your `.tf` files to standard Terraform style.
- `terraform validate` — Checks syntax and verifies that configuration is valid.
- `terraform output` — Displays any defined output values from your configuration.
- `terraform providers` — Lists the providers used in your project.

These commands don't modify your infrastructure but are great for keeping things clean and correct.

## Bringing It All Together

Now that you've seen how `init`, `plan`, and `apply` work individually, we can envision a full continuous integration
(CI) for Terraform projects.

1. Write or modify your `.tf` files.
1. Run `terraform init` (if this is the first time or you've added providers).
1. Run `terraform validate` to check for issues.
1. Run `terraform plan -out=plan.tfplan` to preview changes.
1. Run `terraform apply plan.tfplan` to make them real.
1. (Optional) Run `terraform destroy` to clean up.

Make sure you are running these commands from within your Terraform working directory (the one containing your `.tf`
files). By default, the Terraform CLI looks in the current working directory for the configuration files.

## Wrapping Up

With these commands, you've seen the complete Terraform workflow in action — from initialization through planning,
applying, and even tearing down. These steps form the basis of any Terraform project, whether you're testing locally or
automating with CI/CD.

If you've followed along with this series, you should now feel comfortable creating, inspecting, and managing Terraform
configurations. I don't currently have another post planned in this series, but I do intend to continue writing about
Terraform workflows and MacAdmin-oriented infrastructure automation in the future.

Let me know if any step felt unclear or if you'd like to see a topic expanded or introduced (modules, remote backends,
CI integration, etc.). Your feedback helps shape future posts.

Lastly, if you've made it this far: thank you. I hope this series helped demystify Terraform and gave you the confidence
to use it in your environment.

---

## Series Index

1. [Terraform 101: Introduction]({% post_url 2025-10-14-terraform-101-introduction %})
1. [Terraform 101: Getting Started]({% post_url 2025-10-14-terraform-101-getting-started %})
1. [Terraform 101: Variables and Secrets]({% post_url 2025-10-20-terraform-101-variables-and-secrets %})
1. [Terraform 101: Resources and Data Sources]({% post_url 2025-10-28-terraform-101-resources-and-data-sources %})
1. Terraform 101: Command Line Interface
