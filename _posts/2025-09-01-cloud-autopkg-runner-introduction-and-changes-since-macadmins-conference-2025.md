---
title: "Cloud AutoPkg Runner: Introduction and Changes Since MacAdmins Conference 2025"
date: 2025-09-01 20:48:30 -0400
header:
  teaser: /assets/images/posts/AutoPkgRunner.png
  og_image: /assets/images/posts/AutoPkgRunner.png
categories: ["AutoPkg", "Cloud AutoPkg Runner", "Presentation"]
tags:
  - AutoPkg
  - Cloud AutoPkg Runner
  - macOS
  - MacAdmins
  - CI/CD
  - automation
  - Python
  - open source
  - package management
  - git
  - git client
  - singleton pattern
  - integration testing
  - AWS
  - Azure
  - Google Cloud
  - cloud storage
  - inclusive language
  - code quality
  - CLI
  - development update
---

It's been a few weeks since the MacAdmins Conference at Penn State University where I got to introduce Cloud AutoPkg Runner to the community in my talk, _Celebrating AutoPkg with a new Runner_. If you missed the session or want to re-watch the video, check out this [post]({% post_url 2025-08-15-presentation-from-macadmins-conference %}) I made when the video went live.

The feedback and excitement I've received since then have been incredibly encouraging! I'm thrilled to see the community's enthusiasm and the valuable feedback that has already helped shape the project. Since the presentation, I've been busy pushing several updates to the library to enhance its stability and user experience.

However, since this blog wasn't around when the project launched, I wanted to take a step back and talk about the highlights around what it is and why I wrote it.

## Background Introduction

[Cloud AutoPkg Runner](https://github.com/MScottBlake/cloud-autopkg-runner) was built from the ground up to better support CI/CD pipelines with easy installation and efficient processing of recipes. It achieves this through features like intelligent caching, which optimizes performance by reducing redundant downloads, and concurrent recipe runs, allowing you to process multiple recipes simultaneously. It is an open source project that I created to make AutoPkg easier to use and more performant - especially in environments where the AutoPkg cache does not persist between runs (such as CI/CD workflows). It's a Python project that is hosted on [PyPI](https://pypi.org/project/cloud-autopkg-runner/) and can be easily installed with `uv`, `poetry`, or `pip`. It can also be used as a command line binary.

The command line interface (CLI) is the simplest way to get started. There are [documented arguments](https://github.com/MScottBlake/cloud-autopkg-runner/wiki/Command-Line-Arguments) you can pass to configure how it operates and to define which recipes to run.

If you need more control, you can treat it as a Python import (`import cloud_autopkg_runner`) and you can define a workflow that perfectly suits your needs. You can look at [`__main__.py`](https://github.com/MScottBlake/cloud-autopkg-runner/blob/main/src/cloud_autopkg_runner/__main__.py) or the [Cloud AutoPkg Runner Examples](https://github.com/MScottBlake/cloud-autopkg-runner-examples) repository for usage examples.

### Quick Start

Install with [`uv`](https://docs.astral.sh/uv/), then run your first recipe:

```bash
uv tool install cloud-autopkg-runner
cloud_autopkg_runner --recipe Firefox.pkg
```

---

## Recent Enhancements & New Features

Here's a rundown of the key changes and improvements made to Cloud AutoPkg Runner since MacAdmins Conference 2025.

### Inclusive Language Check ([v0.15.1](https://github.com/MScottBlake/cloud-autopkg-runner/releases/tag/v0.15.1))

The inclusive language check is an automation that runs during Pull Requests to ensure the language used in the commits are non-derogatory, inclusive, and precise. You can read more about my philosophy on this in my previous post, [Writing Inclusive Software: Precision, Respect, and Automation]({% post_url 2025-07-27-inclusive-language-in-software %}), and check the [code quality](https://github.com/MScottBlake/cloud-autopkg-runner/blob/main/.github/workflows/code_quality.yml) GitHub Actions workflow to see it in action.

### Added `--autopkg-pref-file` CLI argument ([v0.16.1](https://github.com/MScottBlake/cloud-autopkg-runner/releases/tag/v0.16.1))

This argument allows you to define a custom AutoPkg preference file to use. I also added `--prefs` to `autopkg run` calls to make sure to use the defined preferences whenever we are using AutoPkg.

### Added Integration Tests ([v0.18.0](https://github.com/MScottBlake/cloud-autopkg-runner/releases/tag/v0.18.0))

I mentioned at the end of my presentation that one of the things I wanted to improve was automated integration tests. Since that talk, I have added tests for AWS S3, Azure Blob Storage, and Google Cloud Storage. These tests each spin up an emulator and runs tests to ensure that the metadata cache file is properly stored and retrieved for each cloud provider. You can now trust that the cache works reliably across providers without manually testing each one.

### Added a `GitClient` class ([v0.18.0](https://github.com/MScottBlake/cloud-autopkg-runner/releases/tag/v0.18.0))

One of the questions I was asked most often during MacAdmins Conference 2025 related to `git`. Some existing tooling around AutoPkg performs automatic code commits of each recipe whenever a new version is downloaded and processed. I told most people at the time that I was hoping to be able to do that, but I couldn't figure out how to do it effectively while not alienating non-git users.

What I came up with is a `GitClient` module within the library in [git_client.py](https://github.com/MScottBlake/cloud-autopkg-runner/blob/main/src/cloud_autopkg_runner/git_client.py). This module provides common git actions with minimal boilerplate.

Here's a quick look at how straightforward it is to use `GitClient` for common operations:

```python
from cloud_autopkg_runner import GitClient

git_client = GitClient(repo_path=".")
git_client.add(".")
git_client.commit("Updated recipes")
git_client.push()
```

For a more thorough example, check out [git_worktree_example.py](https://github.com/MScottBlake/cloud-autopkg-runner-examples/blob/main/examples/git_worktree_example.py).

It is not a full featured git wrapper at this time as my goal was to provide the common actions. If you find yourself needing additional `git` actions, I certainly welcome [pull requests](https://github.com/MScottBlake/cloud-autopkg-runner/pulls).

### Refactored the `AutoPkgPrefs` class ([v0.19.1](https://github.com/MScottBlake/cloud-autopkg-runner/releases/tag/v0.19.1))

I originally made some assumptions that turned out to be wrong. I couldn't think of any reason why you would want more than a single instance of AutoPkg preferences at any given time, so I used the [singleton pattern](https://en.wikipedia.org/wiki/Singleton_pattern) to prevent multiple instances. I also viewed this as a read-only class, so I didn't add any ability to change values after instantiation.

Both of these decisions turned out to hinder workflows, so I refactored the `AutoPkgPrefs` class to remove the singleton pattern. I also added the ability to set new values and to clone the instance.

One example of why this change matters is that by allowing multiple, configurable instances of `AutoPkgPrefs`, you can now easily configure distinct AutoPkg environments. This enables advanced workflows such as creating a `git worktree` and updating recipe paths within the worktree to point to local, isolated AutoPkg data, preventing conflicts with the main branch's preferences.

You can see this in action in [this workflow](https://github.com/MScottBlake/cloud-autopkg-runner-examples/blob/main/.github/workflows/scheduled_python_run.yaml) (which calls [git_worktree_example.py](https://github.com/MScottBlake/cloud-autopkg-runner-examples/blob/main/examples/git_worktree_example.py)).

---

## Join the Discussion

If you’re experimenting with Cloud AutoPkg Runner, I'd love to hear your feedback and workflows. Join the conversation in the #cloud-autopkg-runner channel on the [MacAdmins Slack](https://www.macadmins.org/).

---

## Wrap Up

These updates are just the beginning, and I'm incredibly excited about the future of Cloud AutoPkg Runner.

Ready to get started? Install `cloud-autopkg-runner` today with `uv` or `pip` - check the [Wiki](https://github.com/MScottBlake/cloud-autopkg-runner/wiki) for detailed instructions.

Your feedback is crucial, so please continue to submit [issues and feature requests](https://github.com/MScottBlake/cloud-autopkg-runner/issues) on the GitHub repository or reach out to me directly.
