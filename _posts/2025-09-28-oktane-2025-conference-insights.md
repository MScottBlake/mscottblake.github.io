---
title: "Oktane 2025: Conference Insights"
date: 2025-09-29 01:26:00 -0400
categories:
  - Device Management
  - Identity Management
tags:
  - Chrome Enterprise Core
  - Conference
  - Desktop MFA
  - FastPass
  - Identity Governance
  - MacAdmins
  - Okta
  - Oktane
  - Platform SSO
  - Threat Insights
---

I spent the majority of the past week in Las Vegas for [Oktane](https://www.okta.com/oktane/). Conferences like these do a great job at immersing attendees. I've been living, breathing, and thinking [Okta](https://www.okta.com/) for days. It was an incredible experience full of sessions, conversations, F1 cars, and puppies. I even got the chance to see The Wizard of Oz at Sphere (which was amazing) with some friends old and new after the conference.

Much of the keynote focus this year was on AI agents and [Cross App Access](https://www.okta.com/integrations/cross-app-access/) and while those are certainly interesting topics, I was primarily focussed on device trust and authentication policies this year. As a MacAdmin walking the expo floor and attending sessions, I had a slightly different point of view from many who live the identity mangement life every day.

### Sessions that Stood Out

I attended a session on [Okta Desktop MFA](https://www.okta.com/blog/product-innovation/secure-your-devices-with-oktas-desktop-mfa/) by fellow MacAdmin [Nathaniel Strauss](https://nwstrauss.com/) and his teammate, Chris Durham. They spoke about using Okta to login to macOS and Windows directly. To set it up, there are a few prerequisites, but nothing that seems particularly difficult. This type of solution highlights some of the topics I referenced in my last post. As MacAdmins, we need to be ready to manage certificates and trust chains in ways that directly affect how our users sign in. This is a tool that my team will likely evaluate. Comment below if you're interested in a deeper dive and I might post it here.

Another highlight was a session that showed how [Box](https://www.box.com/) is combining [Okta Identity Governance](https://www.okta.com/products/identity-governance/), [Okta Workflows](https://www.okta.com/products/workflows/), [Kandji](https://explore.kandji.io/77RrP6), and Jira to build an "admin on demand" solution for their devices. Instead of giving permanent administrative rights, access can be elevated temporarily through an automated workflow and then automatically revoked once the task is complete. I love the concept here. Removing standing admin rights lowers the attack surface on each device and their solution allows their users to elevate whenever there is a need as long as they provided a reason. The whole process was logged, including supervisor approvals, if needed. They only made this workflow available to designated teams, but the requests made by those teams were often automatically granted. I love workflows that allow IT to get out of the way.

I also attended a couple security-related sessions that spoke to me. One really hammered home the idea that you should always explain the "why" to achieve buy-in from your collegues and create a security-minded culture. I think that resonated with me because I have seen first-hand that the more transparent you are about changes, the more people respect the decisions that have been made and they are less likely to cause issues with deployments. The other session explained that Okta Threat Insights was a sort of "herd immunity." Every time you use FastPass, you are sending signals back to Okta that are then used as aggregated data to track trends in phishing and other attacks. They noted that one of the recent trends they are seeing is called _MFA downgrade_ where adversaries are directing users to use less secure login factors as part of their phishing attacks. It's a reminder that enabling FastPass is not enough to protect your organization, it's also important to remove less secure authenticators.

### Conversations on the Expo Floor

As always, some of the most practical insights came from the hallways and vendor booths, not from the stage.

I spent some time talking to the Chrome Enterprise team about their Okta integration. This conversation highlighted that we aren't just managing the device anymore. Using Chrome Enterprise Core or Chrome Enterprise Premium, admins can manage Google Chrome on the profile level. The Chrome Device Trust integration can then use those signals as part of Okta Device Assurances and those can be used in authentication policies to really secure your corporate resources. That means users can’t just flip over to a personal Chrome profile or a different browser, bypassing your configurations and security requirements. And with Chrome Enterprise Core consolidating all those policies into one portal, it feels less like a bolt-on and more like a natural extension of the management tools we already use.

There were also vendors showing off their approach to protecting Okta configurations. One backup solution looked and felt like a git interface for identity objects, displaying changes line by line before restoring. These solutions highlight that identity provider settings have become too critical to continue without a disaster recovery solution, and the patterns we’ve used for device management (such as version control) are showing up here too.

### Takeaways

Overall, as someone who focuses on devices with a side of identity, I often felt like an outsider. Despite that, I would definitely go again. With any luck, that'll be in 2026.

It was incredibly inspiring and I certainly left with a large list of roadmap recommendations.
