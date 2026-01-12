---
title: "Local LLMs on Spare Apple Silicon: A Cautionary Tale"
date: 2026-01-11 20:53:00 -0500
categories:
  - Productivity
  - AI
tags:
  - LLM
  - Local LLM
  - Apple Silicon
  - macOS
  - MacAdmins
  - Homelab
  - Self-Hosting
  - Machine Learning
  - AI Infrastructure
---

Can an admin use old Apple Silicon hardware lying around to run local LLMs?

Are they usable?
Are they practical?
If they *are* usable - under what circumstances?

If you're an IT Admin, chances are you have spare Macs collecting dust for one reason or another. If you're like me, the idea of repurposing that hardware to run local LLMs is immediately appealing:

- No subscriptions
- No usage limits
- No data leaving your local network

It sounds great on paper, so I set out to test that premise.

## TL;DR

Was I successful? In a word: No.

Not because it’s impossible.
Not because the tooling doesn’t exist.
But because the gap between *it runs* and *it’s useful* is far wider than I expected.

What followed was three weeks of trial, error, frustration, and a much clearer understanding of where local LLMs stand today for hobbyist use.

## Why I Went Down This Rabbit Hole

This experiment started with the addition of Remote Direct Memory Access (RDMA) in macOS 26.2.

> RDMA enables direct memory access from one computer into another without involving the operating system, CPU, or cache.

In practical terms, RDMA dramatically reduces overhead when computers communicate with each other. That matters because LLM performance at scale is fundamentally about memory movement, not raw compute.

This is why clustering matters. If you don't have enough raw memory, you'll need more devices.

### The Catch

Apple’s RDMA support is limited to Thunderbolt 5.

That immediately disqualified every piece of spare hardware I had lying around.

I should have stopped at this point, but I didn’t.

## The Tooling

I was pretty pessimistic about having enough RAM in these devices, so I was pretty sure I would need to cluster devices. To do this, I used a relatively new tool called [EXO](https://exolabs.net/).

EXO is genuinely impressive. It abstracts away most of the painful parts of clustering:

- Automatically discovers peers on your network
- Allows you to select multiple machines
- Evenly shards models across devices

From a UX standpoint, it’s exactly what you want this kind of tool to be.

Unfortunately, it's still new and they are still working out some bugs.

## Why is Clustering Probably Required?

The core reason is that the entire model must fit in memory.

I primarily tested with an Apple M3 MacBook Air with 16 GB of unified memory. Once macOS is running, you realistically have only 6–8 GB available for a model.

That severely limits your options, but there *are* models that fit in that space.

## The Reality of Small Models

They are useless.

Completely useless.

I won’t waste your time with benchmarks or examples. If a model is small enough to fit comfortably into 6–8 GB of memory, it is too small to perform any meaningful work beyond novelty demos.

At that point, clustering becomes the only way forward.

---

## Clustering Without RDMA

I then moved on to clustering with EXO. I ran 2–3 MacBooks, each with 16 GB memory.

Functionally, this worked.

Practically, it did not.

Even with larger models distributed across multiple machines, performance was horrific. Latency dominated everything. Token generation crawled. Interactive use was frustrating to the point of being unusable.

This is where RDMA stops being a “nice to have” and becomes table stakes.

Without it, the overhead outweighs any benefit from additional hardware.

## Final Verdict

### Can you run local LLMs on spare Apple Silicon hardware?

Yes.

### Should you?

Not with the hardware most of us have access to today.

Until RDMA-capable machines are common enough and affordable enough to repurpose, clustering older Macs simply doesn’t make sense for serious use.

Unless you go buy new hardware, I think we are a few years away from this type of workflow.

### Is There *Any* Hope?

Possibly.

One area I still find promising is local autocomplete. I wasn’t able to make that work *yet*, but it’s the direction I plan to continue exploring.

## What I Actually Got Out of This

Despite the outcome, I don’t regret the experiment.

I learned a tremendous amount about:

- Model sizing
- Memory constraints
- Distributed inference
- Where the current hype diverges from reality

If local LLMs interest you, I absolutely recommend experimenting with them. Just keep your expectations realistic.

Unless you have some truly beefy hardware lying around, this is still firmly in the *learning and tinkering* phase, not the *daily productivity* phase.

And that's okay, as long as we're honest about it.
