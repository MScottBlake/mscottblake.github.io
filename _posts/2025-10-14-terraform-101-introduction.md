---
title: "Terraform 101: Introduction"
date: 2025-10-14 19:13:00 -0400
categories: ["Terraform"]
series: "Terraform 101"
series_order: 1
tags:
  - Infrastructure as Code
  - IaC
  - Terraform
  - GitHub
  - GitLab
  - BitBucket
  - Jamf
  - JNUC
  - Apple School Manager
  - Apple Business Manager
  - MacAdmins Conference
  - MacAdmins Slack
  - Zentral
  - Fleet Device Management
youtubeId: 38ayowbtW2o
header:
  teaser: /assets/images/posts/HashiCorpTerraform.png
  og_image: /assets/images/posts/HashiCorpTerraform.png
---

If you've tried to get into Infrastructure as Code (IaC) in the past and felt overwhelmed, you are not alone. This is going to be the first installment of a series of posts where I'm going to dive into [Terraform](https://www.hashicorp.com/en/products/terraform). What is it and why use it? Where do we start? All valid questions and all things I hope to cover.

I have been wanting to make a Terraform 101 post for a couple months, but not only is it a daunting topic to learn, it's also a daunting topic to write about.

## How did we get here?

Compliance is becoming a bigger emphasis in many organizations. As a result, IT is being asked to show proof of configurations and audit trails while many of our tools don't have this capability.

Enter [GitHub](https://github.com/), [GitLab](https://about.gitlab.com/), [BitBucket](https://bitbucket.org/product/), and others. These compliance and audit requests are all made easier as more configurations are controlled in code.

Back in April 2025, I made a [comment](https://macadmins.slack.com/archives/C060PU4T1/p1744842844707939?thread_ts=1744842086.529909&cid=C060PU4T1) in the [MacAdmins Slack](https://www.macadmins.org/) saying "I think the traditional sysadmin is being morphed into a DevOps admin. If you can't do any IaC, I feel it's going to get harder and harder to maintain continued employment." I still stand by that opinion today. The world is changing around us and if we don't continue to evolve, we will be left behind.

Code repositories have built-in features that make these tasks significantly easier. You get built-in history, including why a change was made as well as who made it. Most code repositories will also give you the ability to add access controls so only authorized people can make changes. These usually come with extra features for code reviews and other requirements.

## Recent Events

[Zentral](https://www.zentral.com/) "applies infrastructure-as-code principles to device management" using Terraform.

[Fleet Device Management](https://fleetdm.com/) is also leaning heavily on a [GitOps approach](https://fleetdm.com/docs/configuration/yaml-files) to management using YAML files.

Jamf made an [announcement](https://www.jamf.com/blog/jnuc-2025-keynote/) at JNUC 2025 that they are releasing an official Terraform provider.

Soon after the release of the Apple School Manager and Apple Business Manager API, [Neil Martin](https://soundmacguy.wordpress.com/) began work on the [axm Terraform Provider](https://registry.terraform.io/providers/neilmartin83/axm/latest/docs). Since my goal is to remain vendor agnostic, this is the provider we'll use in these Terraform 101 posts.

If you learn better from conference sessions, I presented *Processing Webhooks with Terraform and AWS* at [MacAdmins Conference](https://macadmins.psu.edu/) 2023. While I was preparing that session, I quickly realized that I should have submitted a Terraform 101 talk instead because that was half of the session. I didn't go into as much detail as I will here, but I still think it's worth a watch.
{% include youtubePlayer.html id=page.youtubeId %}

With so many tools and vendors adopting Terraform, it’s worth understanding what it actually is and why it’s becoming a key piece of modern IT workflows.

## What is Terraform?

This is a bit of a loaded question, so I decided to ask AI to help me out. Like most AI answers, it was pretty long-winded, so I trimmed it down:

>Terraform is an open-source infrastructure as code (IaC) tool developed by HashiCorp that enables users to define, provision, and manage infrastructure resources across various cloud platforms and on-premises environments using declarative configuration files.
>
>It uses a human-readable configuration language called HashiCorp Configuration Language (HCL) or JSON to specify the desired state of infrastructure, allowing Terraform to automatically handle the creation, modification, and destruction of resources to match that state.
>
>This declarative approach eliminates the need for manual, error-prone processes like clicking through cloud dashboards or writing complex shell scripts.
>
>The tool creates and maintains a state file that acts as a source of truth, tracking the current state of infrastructure and ensuring consistency across deployments.
>
>The core Terraform workflow consists of three stages: Write, Plan, and Apply. In the Write stage, users define infrastructure in configuration files. During the Plan stage, Terraform generates an execution plan detailing the changes needed to achieve the desired state. Finally, in the Apply stage, Terraform executes the proposed changes after user approval, respecting resource dependencies and performing operations in the correct order.
>
>Terraform is widely adopted by organizations for its ability to streamline infrastructure management, reduce human error, ensure consistency, and integrate seamlessly into CI/CD pipelines.

As you can see, there is a lot to this tool. I don't view it as a learning curve, instead I describe it as a learning cliff. Nothing makes sense until you have a semi-firm understanding of it all, then it just *clicks*. My hope is that this collection of posts will help when you inevitably ask yourself the same question a second, third, or even fourth time and you need a reference.

## Next Up

My goal with this series is to help you reach that moment faster, with examples that make sense for MacAdmins.

In the next post, we’ll get hands-on: setting up Terraform, installing the necessary tools, and connecting to the Apple School or Business Manager API using Neil Martin’s `axm` provider.

[Getting Started](/blog/2025/10/14/terraform-101-getting-started/)
