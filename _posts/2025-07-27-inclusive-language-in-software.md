---
title: "Writing Inclusive Software: Precision, Respect, and Automation"
date: 2025-07-27T22:45:00-04:00
categories: [automation]
tags: [inclusive-language, github-actions, pre-commit, best-practices]
description: "How inclusive language leads to better software and how to enforce it using GitHub Actions and Pre-Commit."
---

Inclusive language isn't just a social nicety—it's a powerful tool for building better software. Whether you're writing comments, documentation, variables, or interface strings, the words you choose shape how your software is perceived, understood, and maintained. In this post, we’ll explore why inclusive language matters, how it can lead to clearer, more precise communication, and how to enforce it automatically using GitHub Actions or Pre-Commit hooks.

---

## Why Inclusive Language Matters in Software

**1. It’s respectful.**
Software is used and contributed to by people from all walks of life. Inclusive language helps ensure that no group is marginalized, even inadvertently, through outdated or harmful terminology.

**2. It improves clarity.**
Terms like “master/slave” or “whitelist/blacklist” are vague and carry historical baggage. Replacing them with “primary/replica” or “allowlist/blocklist” not only avoids harm but also improves technical precision.

**3. It enhances localization.**
Inclusive and unambiguous language is easier to translate. Terms with cultural or idiomatic meanings often create problems for non-native speakers or machine translation systems. Choosing clear, descriptive words helps your software scale across borders.

**4. It reflects your values.**
Codebases—especially open-source projects—are a reflection of the communities behind them. Adopting inclusive language communicates that your team values equity, professionalism, and collaboration.

---

## Practical Examples of Improved Language

| Original Term   | Improved Term  | Reason                                  |
| --------------- | -------------- | --------------------------------------- |
| `master` branch | `main` branch  | Neutral and descriptive                 |
| `blacklist`     | `denylist`     | More descriptive and culturally neutral |
| `dummy data`    | `sample data`  | "Dummy" can be derogatory and imprecise |
| `man hours`     | `person hours` | Gender-neutral and accurate             |
| `sanity check`  | `quick check`  | Avoids ableist language                 |

---

## Enforcing Inclusive Language Automatically

To make inclusive language a consistent part of your development process, you can integrate automated checks into your CI pipeline and local development tools.

### GitHub Actions

The [get-woke/woke](https://github.com/get-woke/woke) action is a simple way to add inclusive language checks to your GitHub workflows.

```yaml
name: Inclusive Language Check
on:
  - pull_request
permissions:
  contents: read
jobs:
  inclusive_language:
    name: "🔤 Inclusive Language Check"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      - name: Run woke
        uses: get-woke/woke-action@v0
        with:
          fail-on-error: true
```

This will scan your codebase for non-inclusive terms on every pull request or commit, depending on how you configure your workflow.

**Example output:**

```sh
example.py:6:18: warning: "whitelist" may be insensitive, use "allowlist", "safelist" instead (rule: whitelist)
# Add user to the whitelist
                  ^
```

You can customize the ruleset using a `.woke.yml` config file in your repo.

**Example `.woke.yml`:**

```yaml
# .woke.yml
ignore_files:
  - vendor/*
rules:
  - name: blacklist
    terms:
      - blacklist
    alternatives:
      - denylist
  - name: whitelist
    terms:
      - whitelist
    alternatives:
      - allowlist
  - name: master-slave
    terms:
      - master
      - slave
    alternatives:
      - primary/replica
      - leader/follower
```

---

### Pre-Commit Hook

You can also catch non-inclusive terms before they’re even committed, using [pre-commit](https://pre-commit.com) and [woke](https://docs.getwoke.tech).

**1. Install woke:**

```sh
brew install get-woke/tap/woke
```

**2. Install pre-commit:**

```sh
uv tool install pre-commit
```

**3. Add to `.pre-commit-config.yaml`:**

```yaml
repos:
  - repo: https://github.com/get-woke/woke
    rev: v0.19.0  # Use the latest tag
    hooks:
      - id: woke
```

**4. Install the hook:**

```sh
pre-commit install
```

Now, each time you try to commit code, `woke` will run and alert you to any terms that should be replaced.

---

## Building a Better Software Culture

Inclusive language is a small but significant step toward building more respectful, accessible, and globally-friendly software. It's about fostering empathy, improving communication, and helping contributors and users feel welcome. And thanks to modern tools like `woke`, it’s also easy to automate.

---

## Further Reading

- [Inclusive Naming Initiative](https://inclusivenaming.org)
