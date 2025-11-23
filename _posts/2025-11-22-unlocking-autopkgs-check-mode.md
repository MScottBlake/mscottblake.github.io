---
title: "Unlocking AutoPkg’s Check Mode"
date: 2025-11-22 22:20:00 -0400
categories: ["AutoPkg"]
tags:
  - AutoPkg
  - CI/CD
  - Cloud AutoPkg Runner
---

AutoPkg has plenty of features that MacAdmins rely on every day, but every so often you find one that quietly enables smarter, more efficient workflows. The `--check` flag on the `autopkg run` verb is one of those features. It’s been around for years, but it’s rarely discussed and not widely understood.

This post walks through what the flag does, how it works internally, and how recipe authors use the `EndOfCheckPhase` processor to control AutoPkg’s behavior during a check run.

---

## What the `--check` Flag Actually Does

You might think that `--check` behaves like a dry-run, simulating behavior but not performing real work, but that isn’t how AutoPkg works.

When you run a recipe with:

```bash
autopkg run Zoom.pkg --check
```

AutoPkg performs a real run of the recipe  until it reaches a processor named *EndOfCheckPhase*. At that point, it stops. Recipe authors explicitly declare where the check phase ends based on where they place the processor in their recipes.

---

## How a Check Run Works Internally

Here’s the actual lifecycle when you invoke `--check`:

### 1. AutoPkg validates the recipe

Before processing begins, AutoPkg scans the recipe  for a processor named `EndOfCheckPhase`. If none is found, it logs an error:

> Recipe at Zoom.pkg is missing EndOfCheckPhase Processor, not possible to perform check.

This ensures the recipe has a defined stopping point.

### 2. AutoPkg runs processors normally

Every processor *before* `EndOfCheckPhase` runs exactly as it would during a full run:

* Downloads occur
* Inputs and outputs are populated
* Caches are written
* Archives are unpacked
* Metadata is extracted and compared
* Code signatures are verified

### 3. AutoPkg halts when it reaches `EndOfCheckPhase`

This processor acts as a sentinel. As soon as AutoPkg encounters it, check mode stops execution immediately. This means that if you define a post-processor, it will not run. I believe that pre-processors run since they are added to the front end of the process array.

*If I am wrong, please correct me in the comments.*

Ideally, processors that  perform heavyweight or destructive work such as unarchiving, packaging, code signing,  imports, etc.  are never reached. More on this later.

### 4. Cached artifacts remain available

If a new version was detected and downloaded by `URLDownloader` or similar processor, the file is stored in the cache just like normal.
If no new version exists, the cache remains untouched.

Either way, the system ends in a state where you can make fast decisions about whether to run the recipe fully afterward.

---

## Where to Place `EndOfCheckPhase` (and Why)

The placement of the `EndOfCheckPhase` processor is intentional by recipe authors. It should be placed at the earliest point where AutoPkg can determine whether new work is needed.

### The most common and recommended placement

Immediately after the download processor.

Example:

```xml
<dict>
    <key>Processor</key>
    <string>URLDownloader</string>
</dict>
<dict>
    <key>Processor</key>
    <string>EndOfCheckPhase</string>
</dict>
```

Let's say you have a step in a script or CI that performs a check run before a full run. You would want to check the output for a `download_changed` environment variable (set by `URLDownloader`) that can be used to determine if a full recipe run is necessary.

This process can be significantly faster because most download recipes perform more processing, even if nothing was downloaded. If you look around at a bunch of download recipes, you'll see that most have a `CodeSignatureVerifier` processor. Many others will unarchive the download and perform other steps.

If nothing was downloaded, all of that processing is unnecessary, and if you are paying by the minute for your AutoPkg runs, this can add up to a lot of wasted time.

### An alternative: a custom processor

You may find cases where a custom processor is necessary to gather metadata. If that processor is able to  determine whether a new version exists *without downloading anything*, that could be a good place to stop.

In those cases, placing `EndOfCheckPhase` after the metadata processor avoids unnecessary downloads.

Note that admins will need to know the output variable to use for your custom processor in order to make those determinations in their scripts.

General rule: place `EndOfCheckPhase` immediately after the earliest processor that can accurately determine whether the recipe has actual work to perform. This gives you the fastest possible check runs while maintaining correctness.

---

## Why You Would Use `--check`

### Avoid unnecessary work

You control the flow. If a recipe has no updates, you skip processing steps such as unarchive and code signature verification, and you aren't relying on recipe authors to implement `StopProcessingIf` checks.

### Reduce hits to vendor servers

If the version can be determined before downloading, check mode becomes extremely lightweight.

### Speed up automation

Check runs complete faster and help you build pipelines where metadata determines whether additional steps should run.

### Warm the cache for subsequent runs

If a new version *does* exist, the downloaded file is cached during the check run, making the full run faster.

---

## A Note on Cloud AutoPkg Runner

Although it's not the focus of this post, I feel it's worth mentioning that this exact mechanism  is what powers Cloud AutoPkg Runner. The service relies on `--check` to evaluate recipes efficiently and decide when a full run is needed. This is one of several ways it processes AutoPkg recipe lists faster.
