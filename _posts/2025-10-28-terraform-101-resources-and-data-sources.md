---
title: "Terraform 101: Resources and Data Sources"
date: 2025-10-28 22:44:00 -0400
categories: ["Terraform"]
series: "Terraform 101"
series_order: 4
tags:
  - Infrastructure as Code
  - IaC
  - Terraform
  - Resources
  - Data Sources
  - Apple Business Manager
  - Apple School Manager
  - axm
  - Jamf Pro
  - Kandji
  - Iru
  - Fleet
header:
  teaser: /assets/images/posts/HashiCorpTerraform.png
  og_image: /assets/images/posts/HashiCorpTerraform.png
---

*This post is part of my Terraform 101 series, exploring how MacAdmins can use Infrastructure as Code to manage Apple environments.*

Previously: [Variables and Secrets](/blog/2025/10/20/terraform-101-variables-and-secrets/)

Up Next: Command Line Interface (Coming soon!)

---

In the last post, we cleaned up our configuration by using variables to safely store credentials and API details. Now it’s time to start putting Terraform to work.

This post introduces the two most fundamental concepts in Terraform: `resources` and `data sources`. These form the basis of every configuration, and they’re how Terraform interacts with the real world.

## Declarative, Not Imperative

Before diving into the details, it’s important to understand how Terraform thinks.

As MacAdmins, we’re used to writing imperative scripts. In other words, we're writing code to perform a list of steps that must happen in order:

1. Create server
1. Enroll device
1. Assign profile

Terraform's declarative configurations flip that logic around. You describe the *end state* you want, and Terraform figures out what needs to happen to make (and keep) that true.

For example, you declare:

> “This Device Management Service should exist in Apple Business Manager and it should manage these specific serial numbers.”

Terraform then checks:

- Does that service already exist?
- If not, create it.
- If it exists but differs from your configuration, update it.
- If it matches perfectly, do nothing.

It continuously compares your *declared state* (your `.tf` files) to the *real state* (Apple Business Manager via the API) and reconciles the two.

## The axm Provider Overview

The [axm Terraform provider](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs) lets Terraform interact with Apple School or Business Manager. As of this writing, it exposes one *resource* and several *data sources*:

### Resources

- `axm_device_management_service` — defines a device management service record (such as Jamf Pro, Kandji, Mosyle, etc.)

### Data Sources

- `axm_device_management_service_serial_numbers` — returns serials assigned to a specific management service
- `axm_device_management_services` — lists all available management services
- `axm_organization_device` — returns information about a specific device
- `axm_organization_device_assigned_server_information` — retrieves server assignment details for a specific device
- `axm_organization_devices` — returns information about multiple devices in your organization

Let’s walk through how these work together.

## Resources: Defining What Should Exist

Resources are defined as `resource "type" "label" {}`. You can use the [Teraform Registry](https://registry.terraform.io/) to find a list of resources supported by a provider as the *type*. The *label* is your description of this specific resource. If you are configuring a domain, you might use the domain name as the label. If you're configuring a Device Management Service, you might use the vendor name. The label must be unique for each type. Therefore, you couldn't have two labels of "jamf_pro" - you would need to do something like "jamf_pro_prod" and "jamf_pro_dev" or "site_A" and "site_B".

Resources describe **what Terraform should create or manage**. In the `axm` provider, there’s one key resource: `axm_device_management_service`.

This resource represents a Device Management assignment within Apple School or Business Manager.

```hcl
resource "axm_device_management_service" "iru" {
  id = "FAKE0000111122223333444444444444"
  device_ids = [
    "FAKE000ABC123",
    "FAKE111DEF456",
    "FAKE222GHI789"
  ]
}
```

This block tells Terraform:

> Find a Device Management Service in Apple Business Manager with an identifier of *FAKE0000111122223333444444444444* and make sure that the given list of devices are assigned to that server.

Terraform will ensure that configuration matches the declared state — creating or updating it if necessary. If someone edits this record directly in Apple Business Manager, Terraform will detect the drift the next time you run it.

Normally, the resource would include other fields like a name, and Terraform would create it if needed. Unfortunately, Apple does not have that ability via API, so this provider is limited to assigning and unassigning devices. If you add a device to the list, Terraform will ensure that it is assigned to the given server, and if you remove it, Terraform will unassign it.

## Data Sources: Reading What Already Exists

Data sources are **read-only**. They are declared just like resources, except you use the `data` keyword instead.

Here are a few examples of querying Apple Business Manager for existing information:

### List all Device Management Services

```hcl
data "axm_device_management_services" "all" {}
```

This returns all available servers in your Apple organization.

### Look up a Specific Device

```hcl
data "axm_organization_device" "conf_room_mac_mini" {
  id = "GX7N12345XYZ"
}
```

This lets you pull information about a specific device such as model, color, status, MDM assignment, order date, etc.

### Get Devices Assigned to an MDM Server

```hcl
data "axm_device_management_service_serial_numbers" "fleetdm_serials" {
  server_id = data.axm_device_management_services.all.services[0].id
}
```

This retrieves all serial numbers assigned to the first MDM service found. In a real-world scenario, you could use this output to audit which devices are attached to which MDM server.

The Apple API and `axm` provider are both in their early days and require ID lookups. I expect that to get easier as both are improved.

## Using Resources and Data Sources Together

Data sources and resources often work hand-in-hand.

For example, in one of my projects (using a different provider), I have user groups defined as resources in one project, and then I refer to them as data sources in all others. I can lookup a group my its name and then use that as the scope for an application assignment.

## When to Use Each

| Type        | Purpose                   | Action            |
| ----------- | ------------------------- | ----------------- |
| Resource    | Defines what should exist | Create / Update   |
| Data Source | Reads what already exists | Query / Reference |

## Wrapping Up

Resources and data sources are the backbone of Terraform. They define what you want Terraform to manage and what context it needs to make decisions.

- Resources declare *end-state*: what should exist.
- Data sources provide *insight*: what already exists.
- Terraform compares the two to keep your infrastructure aligned.

## Next Post

In the next post, we’ll see these concepts in action — using Terraform’s CLI commands like `plan` and `apply` to preview and apply changes, and watching how Terraform figures out what needs to happen to reach your declared state.

Up Next: Command Line Interface (Coming soon!)
