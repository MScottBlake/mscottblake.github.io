---

title: "From ClickOps to Code: Terraforming a Live System"
date: 2026-03-03 19:40:00 -0500
toc: true
toc_sticky: true
categories:
  - Terraform
  - Okta
tags:
  - Terraform
  - Infrastructure as Code
  - IaC
  - Okta
  - Automation
  - Maintainability
excerpt: "A practical guide to importing and maintaining an existing live system with Terraform - from API discovery to generated configuration and long-term maintainability."

---

There are hundreds if not thousands of Terraform examples and walk throughs on the internet today. Most of them start in
a clean environment. They assume a brand new account. No history. No surprises.

But that's rarely the situation most of us inherit.

More often, we're working on something that's been evolving for years:

- Resources created manually in a GUI
- Naming conventions that shifted over time
- Temporary fixes that became permanent
- Configurations that "just work," but nobody is entirely sure why

Applying Infrastructure as Code (IaC) using Terraform in an existing, already-provisioned environment is often called
*Brownfield Terraform*.

You're not building from scratch.

You're (very carefully) translating the current reality into code.

## What Does Success Actually Look Like?

Before getting into mechanics, it helps to define the goal.

In a brownfield migration, success is not:

- Rebuilding everything
- Refactoring immediately
- "Cleaning it up" on day one

Success is simpler. You run:

```sh
terraform plan
```

And Terraform responds with:

```text
No changes. Your infrastructure matches the configuration.
```

That's the first milestone.

At that moment:

- Terraform understands your system
- Your state matches production
- Future changes can flow through code

Everything else builds from there.

## Step 0: Decide Where State Lives

Before importing anything, answer one question:

Where will Terraform state live long-term?

If this system matters, you probably don't want local state files. A remote backend with locking enabled sets the tone
early.

Example:

```hcl
terraform {
  required_version = ">= 1.5.0"

  backend "s3" {
    bucket  = "terraform-state"
    key     = "live-system/terraform.tfstate"
    region  = "us-east-1"
    encrypt = true
  }
}
```

Then:

```sh
terraform init
```

It's a small step, but it changes the way the project feels. It becomes intentional.

## Step 1: Configure the Provider

For this post, I'll use Okta as an example provider because it's a system I'm familiar with, but the concepts apply to
many others.

A minimal provider setup might look like:

```hcl
provider "okta" {
  org_name  = "example"
  base_url  = "okta.com"
  api_token = var.okta_api_token
}

variable "okta_api_token" {
  type      = string
  sensitive = true
}
```

Then:

```bash
export TF_VAR_okta_api_token="****"
terraform init
```

At this point, Terraform can talk to the API, but it doesn't know anything about what already exists.

## Step 2: Discover What Exists (This Is the Real Work)

Before Terraform can manage something, it needs to know it exists.

In a greenfield project, that's trivial because you're defining everything.

In a brownfield system, discovery is often the most time-consuming step.

Some Terraform providers support advanced "list" or
[query](https://developer.hashicorp.com/terraform/language/v1.14.x/import/bulk#define-a-query)-style features that can
generate configuration automatically.

Okta does not. At least not yet.

So how do you move from "resources exist somewhere" to usable Terraform
[import blocks](https://developer.hashicorp.com/terraform/language/import/single-resource#define-an-import-block)?

You have two practical approaches:

1. Manual discovery (reasonable for small systems)
2. Scripted discovery (necessary for large systems)

Let's walk through both.

### Option 1: Manual Discovery (Small Environments)

If you're working with:

- A handful of resources
- A limited number of types
- A system you can comfortably review in the UI

You can query the API manually.

For example, listing groups:

```bash
curl -s \
  -H "Authorization: SSWS ${OKTA_TOKEN}" \
  -H "Accept: application/json" \
  "https://example.okta.com/api/v1/groups"
```

This returns JSON listing each group. The output will include something similar to this, but likely with more data
included:

```json
[
  {
    "id": "00g1abcdXYZ12345",
    "profile": {
      "name": "Engineering"
    }
  },
  {
    "id": "00g2efghABC67890",
    "profile": {
      "name": "Finance"
    }
  }
]
```

From here, you could manually write import blocks:

```hcl
import {
  id = "00g1abcdXYZ12345"
  to = okta_group.engineering
}

import {
  id = "00g2efghABC67890"
  to = okta_group.finance
}
```

For very small environments, this is perfectly reasonable.

But it doesn't scale.

### The Scaling Problem

In a mature system, you might have:

- Hundreds or thousands of resources
- Multiple resource types
- Deep relationships between objects

At that point:

- Manual copy/paste becomes error-prone
- Naming conventions drift
- You will miss something

This is where scripting stops being optional.

### Option 2: Scripted Discovery (Large Environments)

The pattern is consistent across systems:

1. Query the API
2. Extract stable identifiers
3. Output deterministic Terraform import blocks
4. Repeat by resource type

> You don't need to over-engineer this.
> The goal of this script is repeatability. If you can re-run it next week and get the same import blocks, you're on the
> right track.
{: .notice--info }

Here's an example script that:

- Queries the API
- Handles pagination
- Outputs Terraform import blocks

```python
#!/usr/bin/env python3
"""
Generate Terraform import statements for Okta groups.

This script:
1. Fetches all groups from Okta API
2. Generates import blocks with resource names based on group names
3. Handles pagination automatically
4. Respects Okta API rate limits

Usage:
    export OKTA_API_TOKEN=your_token_here
    python3 generate_group_imports.py

Output:
    - terraform/imports.tf: Import statements
"""

import os
import re
import sys
import time
import requests

# Okta API configuration from environment variables
OKTA_ORG_NAME = os.getenv("OKTA_ORG_NAME", "example")
OKTA_BASE_URL = os.getenv("OKTA_BASE_URL", "okta.com")
OKTA_API_TOKEN = os.getenv("OKTA_API_TOKEN")
OKTA_URL = f"https://{OKTA_ORG_NAME}.{OKTA_BASE_URL}"

# Output file
IMPORT_FILE = "terraform/imports.tf"


def _slugify(text: str) -> str:
    """
    Convert a group name to a valid Terraform resource identifier.

    Example: "Admin - Read Only" -> "admin_read_only"
    """
    # Remove special characters and replace spaces/hyphens with underscores
    slug = re.sub(r"[^\w\s-]", "", text)
    slug = re.sub(r"[\s-]+", "_", slug).lower().strip("_")
    return slug


def _get_okta_headers() -> dict[str, str]:
    """Return headers for Okta API authentication."""
    if not OKTA_API_TOKEN:
        print("Error: OKTA_API_TOKEN environment variable not set", file=sys.stderr)
        print("\nSet it with: export OKTA_API_TOKEN=your_token_here", file=sys.stderr)
        sys.exit(1)

    return {
        "Authorization": f"SSWS {OKTA_API_TOKEN}",
        "Accept": "application/json",
        "Content-Type": "application/json",
    }


def _fetch_all_groups() -> list[dict[str, str]]:
    """
    Fetch all groups from Okta API with pagination.

    Returns:
        list: List of group dictionaries with 'id' and 'name'
    """
    print("Fetching groups from Okta API...")
    print(f"  URL: {OKTA_URL}/api/v1/groups")

    url = f"{OKTA_URL}/api/v1/groups"
    headers = _get_okta_headers()
    all_groups = []
    page_count = 0

    while url:
        page_count += 1
        print(f"\n  Fetching page {page_count}...", end=" ", flush=True)

        # Make API request
        response = requests.get(url, headers=headers)

        # Handle rate limiting (429 Too Many Requests)
        if response.status_code == 429:
            reset_time = int(
                response.headers.get("X-Rate-Limit-Reset", str(int(time.time() + 1)))
            )
            sleep_duration = max(reset_time - int(time.time()) + 1, 1)
            print(
                f"\nRate limit hit. Retrying in {sleep_duration}s...",
                end=" ",
                flush=True,
            )
            time.sleep(sleep_duration)
            continue

        # Check for other errors
        if response.status_code != 200:
            print(f"\n\nError: HTTP {response.status_code}")
            print(f"Response: {response.text}")
            sys.exit(1)

        # Parse response
        groups = response.json()
        print(f"found {len(groups)} groups")

        # Process each group
        for group in groups:
            group_id = group.get("id")
            group_name = group.get("profile", {}).get("name", "")

            if not group_id or not group_name:
                print(f"    Warning: Skipping group with missing id or name: {group}")
                continue

            all_groups.append({"id": group_id, "name": group_name})

        # Check for next page (pagination via Link header)
        url = None
        if "link" in response.headers:
            links = response.headers["link"].split(", ")
            for link in links:
                if 'rel="next"' in link:
                    # Extract URL from: <https://...>; rel="next"
                    url = link[link.find("<") + 1 : link.find(">")]
                    break

    print(f"\n✓ Fetched {len(all_groups)} total groups across {page_count} pages")
    return all_groups


def _generate_import_file(groups: list[dict[str, str]]) -> None:
    """
    Generate Terraform import file.

    Args:
        groups: List of group dictionaries
    """
    print("\nGenerating Terraform file...")

    # Ensure output directory exists
    os.makedirs(os.path.dirname(IMPORT_FILE), exist_ok=True)

    with open(IMPORT_FILE, "w") as import_file:
        for group in groups:
            group_id = group["id"]
            group_name = group["name"]

            # Create resource name from group name
            resource_name = _slugify(group_name)

            # Write import block
            import_file.write("import {\n")
            import_file.write(f'  id = "{group_id}"\n')
            import_file.write(f"  to = okta_group.{resource_name}\n")
            import_file.write("}\n\n")

    print(f"✓ Generated {IMPORT_FILE}")
    print(f"\n✓ Created {len(groups)} import statements")


def main() -> None:
    """Main execution flow."""
    print("=" * 80)
    print("Okta Group Import Generator")
    print("=" * 80)

    # Fetch groups from Okta API
    groups = _fetch_all_groups()

    if not groups:
        print("\nNo groups found. Exiting.")
        sys.exit(0)

    # Generate Terraform file
    _generate_import_file(groups)

    # Success message
    print("\n" + "=" * 80)
    print("SUCCESS")
    print("=" * 80)
    print("\nNext steps:")
    print("  Review the generated file:")
    print(f"   - {IMPORT_FILE}")


if __name__ == "__main__":
    main()
```

Run it:

```sh
export OKTA_API_TOKEN=your_token_here
python generate_group_imports.py
```

Now you have import blocks for each ID returned and the script is reproducible and versionable.

### Why Naming Strategy Matters

Notice we didn't use raw IDs as Terraform resource names.

This works:

```hcl
okta_group.group_00g1abcdXYZ12345
```

But this is more descriptive and easier to maintain:

```hcl
okta_group.engineering
```

One optimizes for speed of import.

The other optimizes for long-term readability.

Neither is universally correct, but it's important to be intentional about your naming systems.

## Step 3: Let Terraform Generate the Configuration

Now we lean on a powerful feature introduced in newer Terraform versions:

```sh
terraform plan -generate-config-out=generated_groups.tf
```

Terraform will:

- Process the import blocks
- Query the provider for resource details
- Generate HCL that matches production

Instead of guessing at arguments, you let the provider declare them.

For a single group, you might see:

```hcl
resource "okta_group" "engineering" {
  name        = "Engineering"
  description = "Engineering department users"

  custom_profile_attributes = {}
}
```

The generated file will likely be very verbose.

That's expected.

Accuracy matters more than aesthetics at this stage.

## Work in Batches

The biggest mistake is trying to import everything at once.

Importing 1,000 resources at once creates a massive, unreadable diff. Instead, work by resource type. It reduces
cognitive load and keeps the blast radius of errors small.

In Okta, a logical batching order might look like:

1. **Network Zones:** Low volume, high impact.
2. **Auth Policies & Device Posture:** The foundation of your security.
3. **Groups & Rules:** The identity core.
4. **Applications:** The most verbose and complex.

**Generate → Review → Commit → Repeat.**

Each batch builds confidence.

## Refactor Only After Parity

Wait until you reach a clean `plan` for a resource type before you start cleaning up the code. Once the state matches
production exactly, that's when you can:

- Standardize resource names.
- Extract common patterns into modules.
- Pull hardcoded values into variables.

**The Golden Loop:**

1. **Generate** raw config.
2. **Verify** no changes.
3. **Commit.**
4. **Refactor.**
5. **Verify** no changes again.
6. **Commit** again.

By separating the "import" phase from the "aesthetic" phase, you ensure that every stylistic change is verified against
the live environment.

## What Changes After Parity?

Once Terraform reflects reality, something subtle shifts.

Small UI edits start to show up in `plan`.

You might see:

```text
~ attribute changed
```

And now you have options:

- Accept the change and codify it in the Terraform config
- Revert it with `terraform apply`
- Intentionally modify it to something else

Before Terraform, those changes were silent.

Now they're observable.

That shift alone is often worth the effort.

## A Few Things I've Learned Along the Way

The process of moving from ClickOps to Code isn't just a technical migration; it's an educational one. Here are a few
things that have become clear to me during this process:

### Generated configuration is noisier than expected

When you let Terraform generate your configuration, it doesn't just capture the settings you care about. It captures
*everything*. You'll find default values, deprecated attributes, and internal metadata that you never see in the UI.
Sifting through this noise to find the "intent" of a resource is the most time-consuming part of the refactoring phase.

### API pagination matters sooner than you think

If you're writing scripts to generate import blocks, don't assume a single API call will return everything. In a
production Okta environment, "all groups" or "all users" almost always requires handling pagination. Check the API
documentation to see if it applies to you on every object you're trying to import. Failing to do this is silent and you
may not even realize you're missing part of your environment.

### API rate limits can be quite harsh

Terraform is fast. APIs are often slower. When you run a `plan` or `apply` against hundreds of resources, you will
likely hit rate limits. Some systems are stricter than others. Plan accordingly. Implementing exponential backoff in
your generation scripts and understanding your provider's concurrency settings is essential for a smooth workflow.

### Naming conventions become very visible

In the UI, a group named "Engineering-Prod-Access" and "engineering_prod_access" might be similar enough visually. In
code, those inconsistencies are glaring. This process forces a conversation about naming standards that probably should
have happened years ago.

### "Temporary" resources are everywhere

Every environment has them: the "test-policy-do-not-delete" from 2022 or the "temp-access-for-contractor" that expired
months ago. Terraforming a live system acts as a high-resolution audit. You will find things you forgot existed, and
you'll finally have the visibility needed to delete them.

## Closing Thought

Terraforming a live system isn't about control. It's about clarity.

You're not rewriting history. You're documenting what exists and choosing how it evolves.

Once the system lives in code, the conversation changes.

It's no longer:

> "Who changed this?"

It becomes:

> "Let's look at the plan."

That's the real benefit of going from ClickOps to Code.
