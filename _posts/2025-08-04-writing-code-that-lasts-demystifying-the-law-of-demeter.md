---
title: "Writing Code That Lasts: Demystifying the Law of Demeter"
date: 2025-08-04T12:58:00-04:00
categories: [Python]
tags:
  - Low Coupling
  - Law of Demeter
  - Clean Code
  - Python
  - Python Scripting
  - Software Design
  - Refactoring
  - Maintainability
---

As MacAdmins, we tend to write a lot of scripts. One day we're trying to automate onboarding. The next, we're integrating with an API. Maybe we're digging through preference files for information. These scripts often start life as quick solutions, born from a Slack thread and stitched together with `grep`, `awk`, and optimism. As they grow, they can become hard-to-follow and fragile. Why?

Because we chain things together.
We assume structure we can't control.
We do too much in one place.

That’s where the concept of **low coupling** comes in. As a foundational principle in software design, it encourages us to keep modules, functions, and objects from knowing too much about each other. Loosely coupled code is easier to test, easier to change, and far more resilient to breakage.

In this post, we’ll explore one practical tactic for achieving low coupling: the **Law of Demeter**.

Yes, it applies just as easily to your Zsh and Python scripts as it does to traditional, compiled object-oriented code.

## What Is the Law of Demeter?

> "Only talk to your immediate friends."

This simple phrase sums up the Law of Demeter (LoD), a design principle says that an object should only call methods on:

- Itself — Methods defined in the current class or module.
- Its parameters — Inputs passed directly to it.
- Objects it creates — Instances it instantiates with `new()`, `__init__()`, etc.
- Objects it directly holds — Attributes or variables it owns (e.g. `self.foo`).

In other words: don’t chain too many calls together and don’t reach deep into nested objects to grab something from something’s something’s something. That kind of tight coupling creates brittle, unreadable code.

## Why Low Coupling should matter for MacAdmins

MacAdmin scripts often parse plist files or JSON blobs. They also often use 3rd-party libraries that return deeply nested structures If you find yourself doing `.get().get()["key"]` chains or drilling several layers deep, you're setting yourself up for brittle tooling. Loosely coupled code:

- Is easier to test
- Breaks less when internal structures change
- Encourages clear, single-responsibility interfaces

Following the Law of Demeter is just one strategy to achieve this, but it’s a powerful one, especially when you’re building tools that need to last longer than a single ticket or incident.

---

## Example 1: Publishing a Blog Post

Let’s say you’re modeling content for a blog. You can represent some of the data with a BlogPost class containing a `title`, `body`, and `author`. The author has more detailed information such as a `name` and `email`.

```python
class Author:
    def __init__(self, name: str, email: str) -> None:
        self.name = name
        self.email = email

class BlogPost:
    def __init__(self, title: str, body: str, author: Author) -> None:
        self.title = title
        self.body = body
        self.author = author
```

And then you publish a post like this:

```python
def publish(post: BlogPost) -> None:
    print(f"Title: {post.title}")
    print(f"By: {post.author.name} <{post.author.email}>")
    print("---")
    print(post.body)
```

### ❌ Law of Demeter Violation

This line is the problem:

```python
print(f"By: {post.author.name} <{post.author.email}>")
```

We're reaching into `post`, grabbing the `author`, and then reaching into `author` to get `name` and `email`. That’s a chain of calls that leaks knowledge about the internal structure of both `BlogPost` and `Author`.

This tightly couples the caller (`publish`) to the shape of the `Author` class. Any changes to `Author`'s data model might break `publish()`.

### ✅ Refactor: Let `Author` Provide Its Own Presentation

```python
class Author:
    def __init__(self, name: str, email: str) -> None:
        self.name = name
        self.email = email

    def get_author_signature(self) -> str:
        return f"{self.name} <{self.email}>"

class BlogPost:
    def __init__(self, title: str, body: str, author: Author) -> None:
        self.title = title
        self.body = body
        self.author = author

    def get_title(self) -> str:
        return self.title

    def get_body(self) -> str:
        return self.body

    def get_author(self) -> str:
        return self.author.get_author_signature()


def publish(post: BlogPost) -> None:
    print(f"Title: {post.get_title()}")
    print(f"By: {post.get_author()}")
    print("---")
    print(post.get_body())
```

The `publish` function is no longer coupled to the internal structure of the `Author` class.

This change allows you to change how `Author` stores or formats its `name` or `email` while protecting other portions of the code. For instance, lets say that you need to separate `first_name` and `last_name`. The `Author` class is the only object that needs to be modified. The `BlogPost` class and the `publish` function remain untouched.

Additionally, each of the new methods become easier to test, so it's a win-win situation.

To reiterate, we don't want `BlogPost` to need to know anything about how `Author` stores its data, we just want a string representation of the author. Even though we are the developers of both classes, the classes themselves shouldn't need to know the internals of each other.

---

## Example 2: Parsing Munki Reports

Now let’s step into real-world MacAdmin scripting. Here’s a script that reads data from the latest `managedsoftwareupdate` (Munki) run:

```python
import plistlib
from pathlib import Path

plist_path = Path("/Library/Managed Installs/ManagedInstallReport.plist")
data = plistlib.loads(plist_path.read_bytes())
print(f"Primary IP address: {data["MachineInfo"]["ip_address"][0]}")
```

### ❌ Why This Is Fragile

The calling code has to know:

1. That the plist is a dictionary
1. That it has a `MachineInfo` key
1. That inside `MachineInfo` is an `ip_address` key
1. That `ip_address` is a list, and the first value is what we want

Any changes to this structure will break this logic. Even worse, it’s difficult to test and hard to reuse.

### ✅ Refactor: Use a Purpose-Built Class

Let’s wrap this in a class that presents a stable interface to the outside world:

```python
class MunkiReport:
    def __init__(self, plist_path: str) -> None:
        raw = plistlib.loads(Path(plist_path).read_bytes())
        self._machine_info = raw.get("MachineInfo", {})
        self._ip_addresses = self._machine_info.get("ip_address", [])

    def primary_ip_address(self) -> str:
        return self._ip_addresses[0] if self._ip_addresses else "unknown"
```

Instead of requiring the caller to know the exact nested structure of the plist, we encapsulate all of that inside a class with a clear public interface. In this simple example, that interface is the `primary_ip_address()` method:

```python
report = MunkiReport("/Library/Managed Installs/ManagedInstallReport.plist")
print(f"Primary IP address: {report.primary_ip_address()}")
```

If we wanted, we could take this further and create methods for each plist level, catching exceptions, and providing useful information to pinpoint failures.

#### Key Benefits of This Refactor

##### Encapsulation of structure

The consumer of `MunkiReport` no longer needs to know anything about plist format, dictionary keys, or list indexing. If the underlying plist schema changes, only `MunkiReport` needs to be updated—not every script that reads the data.

##### Single-responsibility

The responsibility of decoding and interpreting Munki’s report format is now handled by one object, reducing duplication of plist-reading logic across scripts.

##### Improved readability

`report.primary_ip_address()` clearly expresses intent. Compare that with `data["MachineInfo"]["ip_address"][0]` which might work, but says _how_ to get the data, not _what_ the data means.

##### Safer defaults

The refactor handles missing data gracefully. It defaults to using an empty dictionary if `MachineInfo` is missing as well as an empty list if `ip_address` is missing. It also returns `"unknown"` if no IP address exists. This prevents runtime exceptions (e.g. `IndexError`, `KeyError`) and allows you to handle any exceptions gracefully.

##### Easier testing

You can now write unit tests for `MunkiReport.primary_ip_address()` using test plist inputs, without worrying about mocking out nested dictionaries everywhere in your logic.

---

## Wrapping Up

Whether you're writing scripts to parse API data or validating that preferences match expected values, minimizing coupling through clear interfaces can dramatically improve maintainability. The Law of Demeter is one tool that encourages this kind of clean separation. By avoiding deep object chains and hiding implementation details, your code becomes easier to reuse, test, and adapt to future changes.

Like any design principle, the Law of Demeter isn’t a rule to follow blindly. In some cases, its benefits may not outweigh the added complexity.

### When _Not_ to Refactor

Sometimes, the simplest solution is the right one. If you're:

- Writing a throwaway script for a one-time task
- Parsing data that’s stable and guaranteed not to change
- Working in an environment where indirection would confuse more than clarify

…then introducing new layers or abstractions may be overkill.

As with all design tradeoffs, balance is key. Aim for clarity, not ceremony. The moment a script starts getting reused, shared, or built upon is when it’s time to think about refactoring to reduce coupling.

Remember: Design principles should serve the code. Not the other way around.

If you’re interested in a follow-up post that covers this concept in Zsh or Bash scripting, let me know. There’s plenty to say there too.
