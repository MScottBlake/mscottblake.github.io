---
title: "The Crossroads of Device and Identity Management"
date: 2025-09-23 23:57:00 -0400
categories:
  - Device Management
  - Identity Management
tags:
  - Platform SSO
  - FastPass
  - macOS
  - Identity
  - Okta
---

As MacAdmins, we’ve always had to keep one eye on security and compliance. When our organizations needed to roll out security baselines or enforce compliance policies, it usually fell to us to translate those requirements into device-level controls. FileVault, Gatekeeper, password complexity... the list goes on.

That skillset has served our industry well.

I feel like the same story is repeating itself — this time with identity.

## Identity is the New Security Baseline

At its core, identity management is about ensuring the right person has access to the right resources at the right time.

The methods we use to validate that identity are evolving. Traditional username and password are no longer enough. Multi-factor authentication helped, but even that is giving way to stronger, more seamless methods.

The new frontier is devices as trust signals.

I am attending Oktane 2025 this week and one thing is evident: there is a heavy focus on FastPass sessions. That's no accident. It’s a clear indicator that identity systems are leaning on device attestations and hardware-backed keys to provide assurance that the user really is who they say they are.

## Devices as Identity Anchors

This is where MacAdmins come in.

FastPass and similar solutions don’t just rely on a user’s credentials, they also evaluate the state of the device. Is it compliant? Does it have the right certificate? Is it running on trusted hardware?

Some of the pieces we manage directly feed into this process:

- SCEP certificates provisioned through MDM
- Secure Enclave or TPM-backed keys tied to the hardware
- Compliance signals from MDM policies

Identity providers consume these signals during authentication, and that means our work in device management can directly affect whether someone's login attempt succeeds or fails.

## Platform SSO and the Future of macOS Identity

Apple is also moving in this direction with Platform SSO. This new framework in macOS allows local logins to integrate directly with cloud identity providers, reducing the gap between device state and identity assurance.

For MacAdmins, Platform SSO is more than just a login improvement. It represents Apple’s recognition that identity signals from the device need to be first-class citizens in authentication. Configuring Platform SSO properly means that device compliance, certificate management, and authentication flows are no longer separate concerns — they’re part of the same access story.

## Why Should We Care?

This isn’t just a technical detail buried in the background. It’s a shift in how organizations are approaching access.

Just as we once had to understand security and compliance frameworks to apply them on devices, we now need a baseline understanding of identity concepts to effectively manage the devices that enforce them.

That means getting comfortable with:

- Authentication flows such as OAuth, OIDC, and SAML
- Certificate lifecycles including enrollment, renewal, and revocation
- Conditional access policies and how device state factors into them

Admins who develop these skills will be better prepared as device and identity teams increasingly converge.

## Leveling Up

If you’re looking for practical next steps:

- Learn the basics of identity protocols, even at a high level
- Understand how your MDM handles certificate distribution and attestation
- Experiment with FastPass (or your identity provider’s equivalent)
- Build partnerships with your identity and security counterparts
- Explore Platform SSO on macOS and see how it can streamline logins and authentication flows

This is an area where we, as MacAdmins, can expand our skillset and bring unique value to our organizations.

## Conclusion

The line between device management and identity management is blurring. Our focus used to be securing the endpoint and now we’re becoming part of the system that secures access itself.

FastPass and device attestations are no longer fringe concepts, they’re becoming the new normal.

If history has taught us anything, it’s this: MacAdmins who embrace these changes, learn the new language of identity, and connect it back to their device expertise will be the ones who thrive at this new crossroads.

Stay tuned. I suspect I will have more to say on this topic at the end of this week as I dive head first into my first identity-centric conference.
