---
title: "Reduce Your AI Skill Costs"
date: 2026-09-28 19:40:00 -0400
toc: true
toc_sticky: true
categories:
  - AI Tooling
tags:
  - MacAdmins
  - Python
  - Automation
  - Agentic Coding
  - Claude Code
  - Skills-based Workflows
  - Deterministic Agents
  - Fleet Device Management
excerpt: "Frontier models are for thinking, but you don't need all the power for everything. Here is the workflow I use every day, and the skills that keep the costs down."
---

In ["Stop Prompting, Start Orchestrating"]({% post_url 2026-05-03-stop-prompting-start-orchestrating %}) I argued that
the way out of inconsistent output is structure, not better prompts. Define the workflow. Call scripts you already
trust. Stop asking the model to reinvent logic.

I have been doing that for months now, and it has a pretty cool side effect. If you have a skill that
is only "run this script and read the output," then we can make it much cheaper to run.

## How I work today

I use Claude Code almost exclusively for work. I am often running the same few commands over and over again with
different sets of data. In an effort to be as deterministic as possible, I've written skills for most of these common
tasks. The skills don't live in my user directory (`~/.claude`) though. They are purpose-built for the
project-of-the-day and live in `project_root/.claude`.

The details of the skills don't matter here, but how they are created does. Here's how I go about creating robust skills
that I can run all day long for pennies.

First, I make sure I am in **Plan Mode** and I set the model to **Opus**. I then define my goal as clearly as possible
and read the resulting plan carefully. I make adjustments where I wasn't clear enough and make sure the plan is updated
accordingly.

> It is _much_ easier to fix things in the plan phase before any code is written. Once assumptions are made, they are
> harder to remove.
{: .notice--info }

When the plan is perfect, I switch the model to **Sonnet** and apply the plan. I no longer need the reasoning ability of
Opus, and using it at this stage is a waste of money. We already worked out all the reasoning in the plan phase, now
we're following directions which is perfect for Sonnet.

The next thing I do is test the script to make sure it performs as expected. I do this manually to be sure. Once I'm
satisfied, I prompt Sonnet to create a skill that uses the script to perform a set of actions. I make sure to do this as
a separate prompt. This ensures that the script is actually created and used.

The last thing I do is update the skill frontmatter (at the top of the skill file) to use **Haiku** and to split the
invocation into a subagent.

```markdown
---
name: skill-name-here
description: Normal description.
model: haiku
context: fork
---
```

To summarize:

1. Opus plans the script.
2. Sonnet writes the script.
3. Sonnet wraps the script in a skill.
4. Haiku runs the skill.

> **Model selection rule:**
> If the skill's job is to call a script, the skill does not need a frontier model. Reasoning belongs in the design of
> the script, not in the execution of it.
{: .notice--info }

A similar concept seems to be possible with Codex. According to documentation, you can specify the model in an agent
definition, and then you can pin skills to the agent. I am relatively unfamiliar with that tool and ecosystem, but if that's where you live, I would encourage you to look into it.

## Savings

Does this approach save tokens? Sort of.

Forking the context does reduce some token use.

Does it save money? Absolutely.

Not all tokens are created equal. Tokens used by Fable and Opus are far more expensive than tokens used by Sonnet and
Haiku.

At the time of this writing, Fable is 10 times more expensive than Haiku.

I do not have precise numbers on what this saves me. What I can tell you is that my routine tasks now run on the
cheapest model I have, and the output quality did not change at all.

If dropping to a small model degrades the result, my skill is doing too much thinking, which is incredibly useful
feedback! A skill that breaks on Haiku is a skill with ambiguity still baked into it. It tells me that it's time to fix
the script.

## Example: Fleet labels

I'm going to walk through this concept to manipulate [Fleet labels](https://fleetdm.com/docs/rest-api/rest-api#labels)
via API. However, the content within the skills doesn't matter as long as we're testing the script to make sure it functions as expected and that we're defining the skills to run on smaller models.

To set the stage, I'm working in a completely empty directory called `~/fleet-mgmt`.

I have also made a copy of the [full project]({{ "/assets/code/fleet-mgmt.zip" | relative_url }}) available. It includes all relevent files from this example, including `pyproject.toml` and other files that are not described in this post.

### Add CLAUDE.md

The first thing we're going to do is create a `CLAUDE.md` file:

```markdown
# Rules for Python modules in this repo

## Runtime and invocation

- Target Python 3.14.
- Every script must be run with `uv` (for example: `uv run script.py`).

## Code style

- Use strict type annotations on all functions, methods, and variables where useful.
- Follow high cohesion and low coupling: each module does one job; modules talk through small, clear interfaces.
- Every module, class, function, and method needs a docstring in Google pydocstyle format.
- Each module docstring must show example invocations (how to run or import and call it).

## Quality gates

- Lint all code with `ruff`.
- Check all type annotations with `ty`.
- Write unit tests and regression tests for every code path.
- Use `pytest` for all tests.

## Skills

- For each new module, create a matching skill so AI agents can invoke it correctly.
```

I use this file to set the coding standards for the project. By stating that code must be linted and type checked,
Claude Code will loop through the output and fix code that is generated. It produces a much higher quality script when
you are explicit about what you want.

Now that I've laid out the ground rules, let's start prompting.

### Handling Authentication

The first thing we need to do when working with any API is to handle authentication. In this case, Fleet doesn't support
OAuth or mTLS for REST clients. Only bearer tokens.

If I am working with an AI agent, I don't want it to have to figure that out every time. Presumably, I want to
manipulate many different things within Fleet in this project, and I want to make sure that I create a script that
provides a consistent connection, and I don't have to reinvent that wheel every time.

The prompt is simple:

```text
Create a module to handle authentication using the Fleet Device Management API.
Use `httpx` to create the connection.
Environment variable names must follow Fleet's GitOps convention: `FLEET_URL`, `FLEET_API_TOKEN`.
```

The script and skill produced:

<details markdown="1">
<summary>View <code>src/fleet_mgmt/auth.py</code></summary>

```python
"""Authenticate against the Fleet Device Management REST API.

This module reads a Fleet server URL and API token from the environment,
builds an ``httpx.Client`` that sends the token on every request, and
verifies the token by calling ``GET /api/v1/fleet/me``.

Required environment variables:
    FLEET_URL: Base URL of the Fleet server, e.g. ``https://fleet.example.com``.
    FLEET_API_TOKEN: A Fleet API token. For a human user, copy one from the
        Fleet UI under "My account" > "Get API token". For a service
        account, create one with
        ``fleetctl user create --name "svc" --api-only``.

Invocation examples:
    Check that the token in the environment is valid::

        $ export FLEET_URL=https://fleet.example.com
        $ export FLEET_API_TOKEN=xxxxx
        $ uv run fleet-auth check
        Authenticated as Jane Doe <jane@example.com> (role: admin)

    Same check, machine-readable output::

        $ uv run fleet-auth check --json
        {"id": 1, "name": "Jane Doe", "email": "jane@example.com", ...}

    Run as a module instead of the installed script::

        $ uv run python -m fleet_mgmt.auth check

    Use the module as a library to build a client for other Fleet API
    calls::

        from fleet_mgmt.auth import build_client, load_credentials, verify_token

        credentials = load_credentials()
        with build_client(credentials) as client:
            user = verify_token(client)
            print(user.email)
"""

from __future__ import annotations

import argparse
import json
import os
import sys
from collections.abc import Mapping
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, Any
from urllib.parse import urlsplit

import httpx

if TYPE_CHECKING:
    from collections.abc import Sequence

_ME_PATH = "/api/v1/fleet/me"


class FleetAuthError(Exception):
    """Base class for authentication errors raised by this module."""


class MissingCredentialsError(FleetAuthError):
    """Raised when ``FLEET_URL`` or ``FLEET_API_TOKEN`` is missing or invalid."""


class InvalidTokenError(FleetAuthError):
    """Raised when the Fleet server rejects the token as invalid or expired."""


class RateLimitedError(FleetAuthError):
    """Raised when the Fleet server responds with HTTP 429.

    Attributes:
        retry_after: Seconds to wait before retrying, if the server sent a
            ``Retry-After`` header, otherwise ``None``.
    """

    def __init__(self, retry_after: float | None) -> None:
        """Initialize the error.

        Args:
            retry_after: Seconds to wait before retrying, or ``None`` if the
                server did not send a ``Retry-After`` header.
        """
        self.retry_after = retry_after
        message = "Fleet API rate limit exceeded"
        if retry_after is not None:
            message = f"{message}; retry after {retry_after:g}s"
        super().__init__(message)


class FleetAPIError(FleetAuthError):
    """Raised when the Fleet server responds with an unexpected error status.

    Attributes:
        status_code: The HTTP status code returned by the server.
    """

    def __init__(self, status_code: int, message: str) -> None:
        """Initialize the error.

        Args:
            status_code: The HTTP status code returned by the server.
            message: A human-readable description of the failure.
        """
        self.status_code = status_code
        super().__init__(f"Fleet API error ({status_code}): {message}")


@dataclass(frozen=True, slots=True)
class FleetCredentials:
    """Credentials used to authenticate against a Fleet server.

    Attributes:
        base_url: The Fleet server's base URL, with no trailing slash.
        token: The bearer token sent with every request.
    """

    base_url: str
    token: str = field(repr=False)


@dataclass(frozen=True, slots=True)
class FleetUser:
    """A Fleet user, as returned by ``GET /api/v1/fleet/me``.

    Attributes:
        id: The user's numeric ID.
        name: The user's display name.
        email: The user's email address.
        global_role: The user's global role (e.g. ``"admin"``), or ``None``
            if the user only has team-level roles.
        api_only: ``True`` if this is a service account created with
            ``fleetctl user create --api-only``.
    """

    id: int
    name: str
    email: str
    global_role: str | None
    api_only: bool


def load_credentials(env: Mapping[str, str] | None = None) -> FleetCredentials:
    """Load Fleet credentials from environment variables.

    Args:
        env: A mapping to read ``FLEET_URL`` and ``FLEET_API_TOKEN`` from.
            Defaults to ``os.environ``. Passing an explicit mapping keeps
            this function pure for testing.

    Returns:
        The loaded credentials, with the base URL stripped of any trailing
        slash.

    Raises:
        MissingCredentialsError: If ``FLEET_URL`` or ``FLEET_API_TOKEN`` is
            unset or blank, or if ``FLEET_URL`` is not a valid ``http`` or
            ``https`` URL with a host.
    """
    source = os.environ if env is None else env

    base_url = source.get("FLEET_URL", "").strip()
    if not base_url:
        message = "FLEET_URL environment variable is not set"
        raise MissingCredentialsError(message)

    parsed = urlsplit(base_url)
    if parsed.scheme not in {"http", "https"} or not parsed.netloc:
        message = f"FLEET_URL must be a valid http(s) URL, got: {base_url!r}"
        raise MissingCredentialsError(message)

    token = source.get("FLEET_API_TOKEN", "").strip()
    if not token:
        message = "FLEET_API_TOKEN environment variable is not set"
        raise MissingCredentialsError(message)

    return FleetCredentials(base_url=base_url.rstrip("/"), token=token)


def auth_headers(credentials: FleetCredentials) -> dict[str, str]:
    """Build the HTTP headers needed to authenticate a Fleet API request.

    Args:
        credentials: The credentials to authenticate with.

    Returns:
        A headers mapping with a ``Bearer`` authorization header and an
        ``Accept: application/json`` header.
    """
    return {
        "Authorization": f"Bearer {credentials.token}",
        "Accept": "application/json",
    }


def build_client(
    credentials: FleetCredentials,
    *,
    timeout: float = 30.0,
    transport: httpx.BaseTransport | None = None,
) -> httpx.Client:
    """Build an ``httpx.Client`` authenticated against a Fleet server.

    Args:
        credentials: The credentials to authenticate with.
        timeout: Request timeout, in seconds.
        transport: An optional custom transport, used in tests to avoid
            real network calls.

    Returns:
        A configured, not-yet-opened ``httpx.Client``. Callers are
        responsible for closing it, typically with a ``with`` statement.
    """
    return httpx.Client(
        base_url=credentials.base_url,
        headers=auth_headers(credentials),
        timeout=timeout,
        transport=transport,
    )


def verify_token(client: httpx.Client) -> FleetUser:
    """Verify the client's token by calling ``GET /api/v1/fleet/me``.

    Args:
        client: An authenticated client, as returned by :func:`build_client`.

    Returns:
        The authenticated user.

    Raises:
        InvalidTokenError: If the server responds with HTTP 401.
        RateLimitedError: If the server responds with HTTP 429.
        FleetAPIError: If the server responds with any other non-2xx status.
        httpx.HTTPError: If the request itself fails (e.g. connection error).
    """
    response = client.get(_ME_PATH)
    raise_for_status(response)
    return _parse_user(response.json()["user"])


def raise_for_status(response: httpx.Response) -> None:
    """Translate a non-2xx Fleet API response into a specific exception.

    Shared across ``fleet_mgmt`` modules so every Fleet API call maps HTTP
    status codes to the same exception types.

    Args:
        response: The HTTP response to check.

    Raises:
        InvalidTokenError: If the status code is 401.
        RateLimitedError: If the status code is 429.
        FleetAPIError: If the status code is any other non-2xx value.
    """
    if response.status_code == httpx.codes.UNAUTHORIZED:
        message = "Fleet API token is invalid or expired"
        raise InvalidTokenError(message)

    if response.status_code == httpx.codes.TOO_MANY_REQUESTS:
        retry_after_header = response.headers.get("Retry-After")
        retry_after = float(retry_after_header) if retry_after_header else None
        raise RateLimitedError(retry_after)

    if response.is_success:
        return

    try:
        body = response.json()
    except ValueError:
        body = None
    message = _error_message(body) if isinstance(body, Mapping) else None

    raise FleetAPIError(response.status_code, message or response.text)


def _error_message(body: Mapping[str, Any]) -> str | None:
    """Build a readable message from a Fleet JSON error body.

    Fleet puts a generic summary in ``message`` (for example "Bad request")
    and the specific cause in ``errors[].reason``. Both are joined so the
    caller sees why the request failed.

    Args:
        body: The decoded JSON error body.

    Returns:
        ``"<message>: <reason>; <reason>"``, or only the parts present, or
        ``None`` if the body has neither.
    """
    summary = str(body.get("message") or "")
    errors = body.get("errors")
    reasons: list[str] = []
    if isinstance(errors, list):
        reasons = [
            str(error["reason"])
            for error in errors
            if isinstance(error, Mapping) and error.get("reason")
        ]
    detail = "; ".join(reasons)
    if summary and detail:
        return f"{summary}: {detail}"
    return summary or detail or None


def _parse_user(data: Mapping[str, Any]) -> FleetUser:
    """Parse a Fleet API ``user`` object into a :class:`FleetUser`.

    Args:
        data: The ``user`` field from a Fleet API response body. Typed as
            ``Any``-valued because it is untrusted JSON crossing a process
            boundary; every field is explicitly converted below.

    Returns:
        The parsed user.
    """
    return FleetUser(
        id=int(data["id"]),
        name=str(data["name"]),
        email=str(data["email"]),
        global_role=None if data.get("global_role") is None else str(data["global_role"]),
        api_only=bool(data.get("api_only", False)),
    )


def _build_arg_parser() -> argparse.ArgumentParser:
    """Build the command-line argument parser for this module.

    Returns:
        A configured ``argparse.ArgumentParser``.
    """
    parser = argparse.ArgumentParser(
        prog="fleet-auth",
        description="Check Fleet Device Management API credentials.",
    )
    subparsers = parser.add_subparsers(dest="command")
    check_parser = subparsers.add_parser(
        "check",
        help="Verify FLEET_URL and FLEET_API_TOKEN against the Fleet server.",
    )
    check_parser.add_argument(
        "--json",
        action="store_true",
        help="Print the authenticated user as JSON instead of plain text.",
    )
    return parser


def main(argv: Sequence[str] | None = None) -> int:
    """Run the ``fleet-auth`` command-line tool.

    Args:
        argv: Command-line arguments, excluding the program name. Defaults
            to ``sys.argv[1:]``.

    Returns:
        A process exit code: ``0`` on success, ``2`` for missing or invalid
        credentials, ``3`` for an invalid or expired token, ``4`` for a
        rate-limited request, and ``1`` for any other API or network error.
    """
    parser = _build_arg_parser()
    args = parser.parse_args(argv)
    command = args.command or "check"

    if command != "check":  # pragma: no cover - argparse rejects unknown commands
        parser.error(f"unknown command: {command}")

    try:
        credentials = load_credentials()
    except MissingCredentialsError as error:
        print(f"error: {error}", file=sys.stderr)
        return 2

    try:
        with build_client(credentials) as client:
            user = verify_token(client)
    except InvalidTokenError as error:
        print(f"error: {error}", file=sys.stderr)
        return 3
    except RateLimitedError as error:
        print(f"error: {error}", file=sys.stderr)
        return 4
    except (FleetAPIError, httpx.HTTPError) as error:
        print(f"error: {error}", file=sys.stderr)
        return 1

    if getattr(args, "json", False):
        print(
            json.dumps(
                {
                    "id": user.id,
                    "name": user.name,
                    "email": user.email,
                    "global_role": user.global_role,
                    "api_only": user.api_only,
                },
            ),
        )
    else:
        print(f"Authenticated as {user.name} <{user.email}> (role: {user.global_role})")

    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

</details>

<details markdown="1">
<summary>View the <code>fleet-auth</code> skill</summary>

````markdown
---
name: fleet-auth
description: Check or use Fleet Device Management API credentials (FLEET_URL, FLEET_API_TOKEN) via the fleet_mgmt.auth module. Use when asked to verify a Fleet API token, log in to Fleet, check Fleet auth, or build an authenticated Fleet API client.
model: haiku
context: fork
---

# Fleet API authentication

Use the `fleet_mgmt.auth` module to check Fleet credentials or to get an
authenticated `httpx.Client` for other Fleet API calls. This module never
performs email/password login — it only validates a pre-issued API token.

## Required environment variables

- `FLEET_URL`: Fleet server base URL, e.g. `https://fleet.example.com`.
- `FLEET_API_TOKEN`: A Fleet API token.

## Commands

Check the token is valid:

```
uv run fleet-auth check
```

Same check, JSON output (for scripting):

```
uv run fleet-auth check --json
```

## Exit codes

| Code | Meaning                                             |
| ---- | --------------------------------------------------- |
| 0    | Token is valid                                      |
| 2    | `FLEET_URL` or `FLEET_API_TOKEN` missing or invalid |
| 3    | Token is invalid or expired                         |
| 4    | Rate limited by the Fleet server                    |
| 1    | Any other API or network error                      |

## JSON output shape

```json
{
  "id": 1,
  "name": "Jane Doe",
  "email": "jane@example.com",
  "global_role": "admin",
  "api_only": false
}
```

## Using it as a library

```python
from fleet_mgmt.auth import build_client, load_credentials, verify_token

credentials = load_credentials()
with build_client(credentials) as client:
    user = verify_token(client)
    # client is now ready for other Fleet API calls, e.g.:
    # client.get("/api/v1/fleet/hosts")
```

## What to tell the user on failure

- **Exit 2 (missing credentials):** Ask them to set `FLEET_URL` and
  `FLEET_API_TOKEN`.
- **Exit 3 (invalid token):** The token is wrong, revoked, or expired.
  - SSO or MFA (multi-factor authentication) users cannot get a token by
    logging in from a script. Tell them to open the Fleet UI, go to
    "My account", and click "Get API token".
  - For a service account, create a token with
    `fleetctl user create --name "svc" --api-only`.
- **Exit 4 (rate limited):** Wait and retry; the error message includes the
  `Retry-After` seconds when the server sent one.
````

</details>

### Labels

Now that we have an authentication module, let's move onto a simple example, labels.

Here's a sample prompt to get a useful python module to manipulate labels:

```text
Create another module to create and delete manual labels using the Fleet Device Management API.
It must also be able to add or remove labels from a set of devices.
```

The script and skill produced:

<details markdown="1">
<summary>View <code>src/fleet_mgmt/labels.py</code></summary>

```python
"""Manage Fleet manual labels: create or delete them, and add or remove them on hosts.

Host commands (``add``, ``remove``) resolve each host identifier (hostname,
hardware serial, or UUID) to a Fleet host ID via
``GET /api/v1/fleet/hosts/identifier/{id}``, then call ``POST`` or
``DELETE /api/v1/fleet/hosts/{id}/labels`` for each host in turn. Only
manual labels can be assigned this way; Fleet rejects dynamic
(query-based) and builtin labels with a 400 response.

Label commands (``create``, ``delete``) manage the labels themselves.
``create`` calls ``POST /api/v1/fleet/labels`` with a manual membership
type. ``delete`` looks up label IDs once via
``GET /api/v1/fleet/labels/summary`` and then calls
``DELETE /api/v1/fleet/labels/id/{id}``. Deleting by ID rather than by name
works for names that contain ``/``, which Fleet's by-name route cannot
match. Fleet refuses to delete builtin labels with a 422 response.

A failure on one host or label (host not found, unknown label, label is
dynamic, duplicate name) does not stop the run: it is recorded and the next
item is tried. A failure that would affect every remaining item (an invalid
token, a rate limit) stops the run immediately.

Required environment variables:
    FLEET_URL: Base URL of the Fleet server, e.g. ``https://fleet.example.com``.
    FLEET_API_TOKEN: A Fleet API token with the maintainer, admin,
        technician, or gitops role. See :mod:`fleet_mgmt.auth` for how to
        obtain one.

Invocation examples:
    Add a label to two hosts by hostname::

        $ export FLEET_URL=https://fleet.example.com
        $ export FLEET_API_TOKEN=xxxxx
        $ uv run fleet-labels add --label canary web-01 web-02
        ok   web-01 (host 12)
        ok   web-02 (host 13)
        2 succeeded, 0 failed

    Remove two labels from a host identified by hardware serial::

        $ uv run fleet-labels remove --label canary --label beta C02XK1ABJG5H
        ok   C02XK1ABJG5H (host 41)
        1 succeeded, 0 failed

    Read hosts from a file, one identifier per line, and get JSON output::

        $ uv run fleet-labels add --label canary --file hosts.txt --json
        {"action": "add", "labels": ["canary"], "succeeded": 2, "failed": 0, ...}

    Read hosts from standard input::

        $ cat hosts.txt | uv run fleet-labels add --label canary --file -

    Create two manual labels with a shared description::

        $ uv run fleet-labels create canary beta --description "Early rollout ring"
        ok   canary (label 21)
        ok   beta (label 22)
        2 succeeded, 0 failed

    Delete a label (``--yes`` is required because deletion is permanent)::

        $ uv run fleet-labels delete canary --yes
        ok   canary (label 21)
        1 succeeded, 0 failed

    Run as a module instead of the installed script::

        $ uv run python -m fleet_mgmt.labels add --label canary web-01

    Use the module as a library::

        from fleet_mgmt.auth import build_client, load_credentials
        from fleet_mgmt.labels import LabelAction, create_labels, delete_labels, update_labels

        with build_client(load_credentials()) as client:
            create_labels(client, ["canary"], description="Early rollout ring")
            results = update_labels(client, ["web-01", "web-02"], ["canary"], LabelAction.ADD)
            failures = [result for result in results if not result.ok]
            delete_labels(client, ["canary"])
"""

from __future__ import annotations

import argparse
import json
import sys
from dataclasses import dataclass
from enum import StrEnum
from typing import TYPE_CHECKING
from urllib.parse import quote

import httpx

from fleet_mgmt.auth import (
    FleetAPIError,
    InvalidTokenError,
    MissingCredentialsError,
    RateLimitedError,
    build_client,
    load_credentials,
    raise_for_status,
)

if TYPE_CHECKING:
    from collections.abc import Iterable, Sequence

_HOSTS_IDENTIFIER_PATH = "/api/v1/fleet/hosts/identifier/{identifier}"
_HOST_LABELS_PATH = "/api/v1/fleet/hosts/{host_id}/labels"
_LABELS_PATH = "/api/v1/fleet/labels"
_LABELS_SUMMARY_PATH = "/api/v1/fleet/labels/summary"
_LABEL_BY_ID_PATH = "/api/v1/fleet/labels/id/{label_id}"

_CREATE = "create"
_DELETE = "delete"


class LabelAction(StrEnum):
    """Which label operation to perform on a host.

    Attributes:
        ADD: Assign the given labels to a host.
        REMOVE: Unassign the given labels from a host.
    """

    ADD = "add"
    REMOVE = "remove"


class LabelError(Exception):
    """Base class for errors raised by this module."""


class HostNotFoundError(LabelError):
    """Raised when no Fleet host matches a given identifier."""


@dataclass(frozen=True, slots=True)
class LabelResult:
    """The outcome of creating or deleting one label.

    Attributes:
        name: The label name the caller supplied.
        label_id: The Fleet label ID, or ``None`` if the operation failed
            before an ID was known.
        error: A human-readable failure reason, or ``None`` on success.
    """

    name: str
    label_id: int | None
    error: str | None

    @property
    def ok(self) -> bool:
        """Whether the label was created or deleted.

        Returns:
            ``True`` if there is no recorded error.
        """
        return self.error is None


@dataclass(frozen=True, slots=True)
class HostResult:
    """The outcome of applying a label action to one host.

    Attributes:
        identifier: The identifier the caller supplied for this host.
        host_id: The resolved Fleet host ID, or ``None`` if resolution
            failed.
        error: A human-readable failure reason, or ``None`` on success.
    """

    identifier: str
    host_id: int | None
    error: str | None

    @property
    def ok(self) -> bool:
        """Whether the label action succeeded for this host.

        Returns:
            ``True`` if there is no recorded error.
        """
        return self.error is None


def read_identifiers(lines: Iterable[str]) -> list[str]:
    """Parse host identifiers from a sequence of lines.

    Blank lines and lines starting with ``#`` are skipped. Duplicate
    identifiers are removed, keeping the first occurrence's position.

    Args:
        lines: Lines of text, such as from a file or standard input.

    Returns:
        The unique, ordered list of host identifiers.
    """
    seen: dict[str, None] = {}
    for line in lines:
        identifier = line.strip()
        if not identifier or identifier.startswith("#"):
            continue
        seen.setdefault(identifier, None)
    return list(seen)


def resolve_host_id(client: httpx.Client, identifier: str) -> int:
    """Resolve a host identifier to a Fleet host ID.

    Args:
        client: An authenticated client, as returned by
            :func:`fleet_mgmt.auth.build_client`.
        identifier: A hostname, hardware serial, UUID, node key, or
            osquery host ID.

    Returns:
        The numeric Fleet host ID.

    Raises:
        HostNotFoundError: If no host matches the identifier.
        InvalidTokenError: If the server responds with HTTP 401.
        RateLimitedError: If the server responds with HTTP 429.
        FleetAPIError: If the server responds with any other non-2xx status.
        httpx.HTTPError: If the request itself fails (e.g. connection error).
    """
    path = _HOSTS_IDENTIFIER_PATH.format(identifier=quote(identifier, safe=""))
    response = client.get(path, params={"exclude_software": "true"})

    if response.status_code == httpx.codes.NOT_FOUND:
        message = f"no host matches identifier {identifier!r}"
        raise HostNotFoundError(message)

    raise_for_status(response)
    return int(response.json()["host"]["id"])


def apply_labels(
    client: httpx.Client,
    host_id: int,
    labels: Sequence[str],
    action: LabelAction,
) -> None:
    """Add or remove labels on a single host.

    Args:
        client: An authenticated client, as returned by
            :func:`fleet_mgmt.auth.build_client`.
        host_id: The Fleet host ID to modify.
        labels: The manual label names to add or remove.
        action: Whether to add or remove the labels.

    Raises:
        InvalidTokenError: If the server responds with HTTP 401.
        RateLimitedError: If the server responds with HTTP 429.
        FleetAPIError: If a label does not exist, is not a manual label,
            or the server responds with any other non-2xx status.
        httpx.HTTPError: If the request itself fails (e.g. connection error).
    """
    method = "POST" if action is LabelAction.ADD else "DELETE"
    path = _HOST_LABELS_PATH.format(host_id=host_id)
    response = client.request(method, path, json={"labels": list(labels)})
    raise_for_status(response)


def update_labels(
    client: httpx.Client,
    identifiers: Iterable[str],
    labels: Sequence[str],
    action: LabelAction,
) -> list[HostResult]:
    """Apply a label action to a set of hosts, continuing past per-host errors.

    Each host is resolved and updated independently. A per-host failure
    (the host is not found, or the label is unknown or not a manual label)
    is recorded in that host's result and does not stop the run. An error
    that would affect every remaining host (an invalid token, a rate
    limit, or a network error) is raised immediately instead, since
    continuing would only repeat the same failure for each remaining host.

    Args:
        client: An authenticated client, as returned by
            :func:`fleet_mgmt.auth.build_client`.
        identifiers: Host identifiers to update, in order.
        labels: The manual label names to add or remove.
        action: Whether to add or remove the labels.

    Returns:
        One :class:`HostResult` per identifier, in the given order.

    Raises:
        InvalidTokenError: If the server responds with HTTP 401.
        RateLimitedError: If the server responds with HTTP 429.
        httpx.HTTPError: If a request fails (e.g. connection error).
    """
    results: list[HostResult] = []
    for identifier in identifiers:
        try:
            host_id = resolve_host_id(client, identifier)
            apply_labels(client, host_id, labels, action)
        except (HostNotFoundError, FleetAPIError) as error:
            results.append(HostResult(identifier, None, str(error)))
        else:
            results.append(HostResult(identifier, host_id, None))
    return results


def create_label(client: httpx.Client, name: str, description: str = "") -> int:
    """Create one manual label.

    Args:
        client: An authenticated client, as returned by
            :func:`fleet_mgmt.auth.build_client`.
        name: The new label's name.
        description: An optional description shown in the Fleet UI.

    Returns:
        The new label's Fleet ID.

    Raises:
        InvalidTokenError: If the server responds with HTTP 401.
        RateLimitedError: If the server responds with HTTP 429.
        FleetAPIError: If a label with this name already exists (409), the
            name is invalid (422), or the server responds with any other
            non-2xx status.
        httpx.HTTPError: If the request itself fails (e.g. connection error).
    """
    body = {"name": name, "description": description, "label_membership_type": "manual"}
    response = client.post(_LABELS_PATH, json=body)
    raise_for_status(response)
    return int(response.json()["label"]["id"])


def fetch_label_ids(client: httpx.Client) -> dict[str, int]:
    """Fetch the ID of every label the token can see, keyed by name.

    Args:
        client: An authenticated client, as returned by
            :func:`fleet_mgmt.auth.build_client`.

    Returns:
        A mapping of label name to Fleet label ID.

    Raises:
        InvalidTokenError: If the server responds with HTTP 401.
        RateLimitedError: If the server responds with HTTP 429.
        FleetAPIError: If the server responds with any other non-2xx status.
        httpx.HTTPError: If the request itself fails (e.g. connection error).
    """
    response = client.get(_LABELS_SUMMARY_PATH)
    raise_for_status(response)
    return {str(label["name"]): int(label["id"]) for label in response.json()["labels"]}


def delete_label(client: httpx.Client, label_id: int) -> None:
    """Delete one label by its Fleet ID.

    Args:
        client: An authenticated client, as returned by
            :func:`fleet_mgmt.auth.build_client`.
        label_id: The Fleet label ID to delete.

    Raises:
        InvalidTokenError: If the server responds with HTTP 401.
        RateLimitedError: If the server responds with HTTP 429.
        FleetAPIError: If the label is builtin (422), no longer exists
            (404), or the server responds with any other non-2xx status.
        httpx.HTTPError: If the request itself fails (e.g. connection error).
    """
    response = client.delete(_LABEL_BY_ID_PATH.format(label_id=label_id))
    raise_for_status(response)


def create_labels(
    client: httpx.Client,
    names: Iterable[str],
    description: str = "",
) -> list[LabelResult]:
    """Create manual labels, continuing past per-label errors.

    A per-label failure (the name already exists or is invalid) is recorded
    in that label's result and does not stop the run.

    Args:
        client: An authenticated client, as returned by
            :func:`fleet_mgmt.auth.build_client`.
        names: Label names to create, in order.
        description: A description applied to every created label.

    Returns:
        One :class:`LabelResult` per name, in the given order.

    Raises:
        InvalidTokenError: If the server responds with HTTP 401.
        RateLimitedError: If the server responds with HTTP 429.
        httpx.HTTPError: If a request fails (e.g. connection error).
    """
    results: list[LabelResult] = []
    for name in names:
        try:
            label_id = create_label(client, name, description)
        except FleetAPIError as error:
            results.append(LabelResult(name, None, str(error)))
        else:
            results.append(LabelResult(name, label_id, None))
    return results


def delete_labels(client: httpx.Client, names: Iterable[str]) -> list[LabelResult]:
    """Delete labels by name, continuing past per-label errors.

    Label IDs are fetched once up front. A per-label failure (no label has
    that name, or the label is builtin) is recorded in that label's result
    and does not stop the run.

    Args:
        client: An authenticated client, as returned by
            :func:`fleet_mgmt.auth.build_client`.
        names: Label names to delete, in order.

    Returns:
        One :class:`LabelResult` per name, in the given order.

    Raises:
        InvalidTokenError: If the server responds with HTTP 401.
        RateLimitedError: If the server responds with HTTP 429.
        FleetAPIError: If the label list itself cannot be fetched.
        httpx.HTTPError: If a request fails (e.g. connection error).
    """
    label_ids = fetch_label_ids(client)
    results: list[LabelResult] = []
    for name in names:
        label_id = label_ids.get(name)
        if label_id is None:
            results.append(LabelResult(name, None, f"no label named {name!r}"))
            continue
        try:
            delete_label(client, label_id)
        except FleetAPIError as error:
            results.append(LabelResult(name, label_id, str(error)))
        else:
            results.append(LabelResult(name, label_id, None))
    return results


def _build_arg_parser() -> argparse.ArgumentParser:
    """Build the command-line argument parser for this module.

    Returns:
        A configured ``argparse.ArgumentParser``.
    """
    parser = argparse.ArgumentParser(
        prog="fleet-labels",
        description="Create or delete Fleet manual labels, or add or remove them on hosts.",
    )
    subparsers = parser.add_subparsers(dest="command", required=True)
    for action in LabelAction:
        action_parser = subparsers.add_parser(
            action.value,
            help=f"{action.value.capitalize()} labels on the given hosts.",
        )
        action_parser.add_argument(
            "--label",
            dest="labels",
            action="append",
            default=[],
            required=True,
            metavar="NAME",
            help="A manual label name. Repeat to specify more than one.",
        )
        action_parser.add_argument(
            "hosts",
            nargs="*",
            metavar="HOST",
            help="A host identifier (hostname, hardware serial, or UUID).",
        )
        action_parser.add_argument(
            "--file",
            metavar="PATH",
            help="Read additional host identifiers from PATH, one per line. Use - for stdin.",
        )
        _add_json_flag(action_parser)

    create_parser = subparsers.add_parser(_CREATE, help="Create manual labels.")
    create_parser.add_argument("names", nargs="+", metavar="NAME", help="A label name to create.")
    create_parser.add_argument(
        "--description",
        default="",
        help="A description applied to every created label.",
    )
    _add_json_flag(create_parser)

    delete_parser = subparsers.add_parser(_DELETE, help="Delete labels by name.")
    delete_parser.add_argument("names", nargs="+", metavar="NAME", help="A label name to delete.")
    delete_parser.add_argument(
        "--yes",
        action="store_true",
        help="Confirm the deletion. Required, because deleting a label cannot be undone.",
    )
    _add_json_flag(delete_parser)
    return parser


def _add_json_flag(parser: argparse.ArgumentParser) -> None:
    """Add the shared ``--json`` output flag to a subcommand parser.

    Args:
        parser: The subcommand parser to extend.
    """
    parser.add_argument(
        "--json",
        action="store_true",
        help="Print results as JSON instead of plain text.",
    )


def _read_identifiers_from_file(path: str) -> list[str]:
    """Read host identifiers from a file path or standard input.

    Args:
        path: A filesystem path, or ``-`` to read from standard input.

    Returns:
        The unique, ordered list of host identifiers found in the file.
    """
    if path == "-":
        return read_identifiers(sys.stdin)
    with open(path, encoding="utf-8") as handle:  # noqa: PTH123
        return read_identifiers(handle)


def _validate_args(parser: argparse.ArgumentParser, args: argparse.Namespace) -> list[str]:
    """Check the parsed arguments and collect the items the command acts on.

    Exits through ``parser.error`` (status 2) on a usage error.

    Args:
        parser: The parser, used to report usage errors.
        args: The parsed arguments.

    Returns:
        The unique, ordered host identifiers for ``add``/``remove``, or
        label names for ``create``/``delete``.
    """
    if args.command in {_CREATE, _DELETE}:
        names = read_identifiers(args.names)
        if not names:
            parser.error("no label names given")
        if args.command == _DELETE and not args.yes:
            parser.error("deleting labels cannot be undone; pass --yes to confirm")
        return names

    identifiers = list(args.hosts)
    if args.file:
        identifiers.extend(_read_identifiers_from_file(args.file))
    identifiers = read_identifiers(identifiers)
    if not identifiers:
        parser.error("no hosts given; pass HOST arguments or --file")
    return identifiers


def _run_command(
    client: httpx.Client,
    args: argparse.Namespace,
    items: list[str],
) -> list[HostResult] | list[LabelResult]:
    """Run the chosen subcommand against the Fleet server.

    Args:
        client: An authenticated client.
        args: The parsed arguments.
        items: Host identifiers or label names, from :func:`_validate_args`.

    Returns:
        The per-host or per-label results, in order.
    """
    if args.command == _CREATE:
        return create_labels(client, items, args.description)
    if args.command == _DELETE:
        return delete_labels(client, items)
    return update_labels(client, items, args.labels, LabelAction(args.command))


def _print_text_report(results: list[HostResult] | list[LabelResult]) -> None:
    """Print one line per result, followed by a summary line.

    Args:
        results: The per-host or per-label results to report, in order.
    """
    for result in results:
        if isinstance(result, HostResult):
            subject, detail = result.identifier, f"host {result.host_id}"
        else:
            subject, detail = result.name, f"label {result.label_id}"
        if result.ok:
            print(f"ok   {subject} ({detail})")
        else:
            print(f"FAIL {subject}: {result.error}")
    succeeded = sum(1 for result in results if result.ok)
    print(f"{succeeded} succeeded, {len(results) - succeeded} failed")


def _print_json_report(
    args: argparse.Namespace,
    items: list[str],
    results: list[HostResult] | list[LabelResult],
) -> None:
    """Print a JSON summary of the command and its results.

    Host commands report the labels applied and one entry per host. Label
    commands report one entry per label.

    Args:
        args: The parsed arguments.
        items: Host identifiers or label names the command acted on.
        results: The per-host or per-label results, in order.
    """
    succeeded = sum(1 for result in results if result.ok)
    entries = [_json_entry(result) for result in results]
    labels = items if args.command in {_CREATE, _DELETE} else args.labels
    print(
        json.dumps(
            {
                "action": args.command,
                "labels": labels,
                "succeeded": succeeded,
                "failed": len(results) - succeeded,
                "results": entries,
            },
        ),
    )


def _json_entry(result: HostResult | LabelResult) -> dict[str, object]:
    """Convert one result into its JSON report entry.

    Args:
        result: A per-host or per-label result.

    Returns:
        ``identifier``/``host_id`` for a host result, or ``name``/``label_id``
        for a label result, plus ``ok`` and ``error``.
    """
    if isinstance(result, HostResult):
        return {
            "identifier": result.identifier,
            "host_id": result.host_id,
            "ok": result.ok,
            "error": result.error,
        }
    return {
        "name": result.name,
        "label_id": result.label_id,
        "ok": result.ok,
        "error": result.error,
    }


def main(argv: Sequence[str] | None = None) -> int:
    """Run the ``fleet-labels`` command-line tool.

    Args:
        argv: Command-line arguments, excluding the program name. Defaults
            to ``sys.argv[1:]``.

    Returns:
        A process exit code: ``0`` if every host or label succeeded, ``5``
        if one or more failed, ``2`` for missing credentials or a usage
        error, ``3`` for an invalid or expired token, ``4`` for a
        rate-limited request, and ``1`` for any other API or network error
        that stopped the run.
    """
    parser = _build_arg_parser()
    args = parser.parse_args(argv)
    items = _validate_args(parser, args)

    try:
        credentials = load_credentials()
    except MissingCredentialsError as error:
        print(f"error: {error}", file=sys.stderr)
        return 2

    try:
        with build_client(credentials) as client:
            results = _run_command(client, args, items)
    except InvalidTokenError as error:
        print(f"error: {error}", file=sys.stderr)
        return 3
    except RateLimitedError as error:
        print(f"error: {error}", file=sys.stderr)
        return 4
    except (FleetAPIError, httpx.HTTPError) as error:
        print(f"error: {error}", file=sys.stderr)
        return 1

    if args.json:
        _print_json_report(args, items, results)
    else:
        _print_text_report(results)

    return 0 if all(result.ok for result in results) else 5


if __name__ == "__main__":
    raise SystemExit(main())
```

</details>

<details markdown="1">
<summary>View the <code>fleet-labels</code> skill</summary>

````markdown
---
name: fleet-labels
description: Create or delete Fleet Device Management manual labels, and add or remove them on hosts, via the fleet_mgmt.labels module. Use when asked to create, make, delete, or remove a Fleet label, or to tag, label, add a label to, or remove a label from one or more Fleet hosts/devices.
model: haiku
context: fork
---

# Fleet labels

Use the `fleet_mgmt.labels` module to create or delete **manual** labels,
and to add or remove them on a set of Fleet hosts. It cannot create or
assign dynamic (query-based) labels, and Fleet refuses to delete builtin
labels such as `All Hosts` or `macOS`.

Requires the same `FLEET_URL` / `FLEET_API_TOKEN` environment variables as
the `fleet-auth` skill. Use that skill first if the token needs checking.

## Commands

Add a label to one or more hosts (accepts hostname, hardware serial, or UUID):

```
uv run fleet-labels add --label canary web-01 web-02
```

Remove labels (repeat `--label` for more than one):

```
uv run fleet-labels remove --label canary --label beta web-01
```

Read hosts from a file (one identifier per line, `#` for comments) or stdin:

```
uv run fleet-labels add --label canary --file hosts.txt
cat hosts.txt | uv run fleet-labels add --label canary --file -
```

Create manual labels (one `--description` applies to all names given):

```
uv run fleet-labels create canary beta --description "Early rollout ring"
```

Delete labels by name. `--yes` is required: deletion cannot be undone and
removes the label from every host that has it. Confirm with the user before
passing `--yes` unless they explicitly asked for the deletion.

```
uv run fleet-labels delete canary beta --yes
```

Add `--json` to any command for machine-readable output.

## Exit codes

| Code | Meaning                                                                 |
| ---- | ----------------------------------------------------------------------- |
| 0    | Every host or label succeeded                                           |
| 5    | One or more failed, run completed anyway                                |
| 2    | Missing credentials, no hosts/labels given, or `delete` without `--yes` |
| 3    | Token is invalid or expired                                             |
| 4    | Rate limited by the Fleet server                                        |
| 1    | Other API or network error; run stopped early                           |

A per-item failure (host not found, unknown label, dynamic/builtin label,
duplicate name on create) does not stop the run — every host or label is
still attempted, and the failures are listed in the output. Only an invalid
token, a rate limit, or a network error stops the run early, since those
would fail every remaining item anyway.

## JSON output shape

```json
{
  "action": "add",
  "labels": ["canary"],
  "succeeded": 1,
  "failed": 1,
  "results": [
    { "identifier": "web-01", "host_id": 12, "ok": true, "error": null },
    {
      "identifier": "web-02",
      "host_id": null,
      "ok": false,
      "error": "no host matches identifier 'web-02'"
    }
  ]
}
```

`create` and `delete` use the same top-level shape, with label entries:

```json
{
  "action": "delete",
  "labels": ["canary", "nope"],
  "succeeded": 1,
  "failed": 1,
  "results": [
    { "name": "canary", "label_id": 21, "ok": true, "error": null },
    {
      "name": "nope",
      "label_id": null,
      "ok": false,
      "error": "no label named 'nope'"
    }
  ]
}
```

## Using it as a library

```python
from fleet_mgmt.auth import build_client, load_credentials
from fleet_mgmt.labels import LabelAction, create_labels, delete_labels, update_labels

with build_client(load_credentials()) as client:
    create_labels(client, ["canary"], description="Early rollout ring")
    results = update_labels(client, ["web-01", "web-02"], ["canary"], LabelAction.ADD)
    failed = [r for r in results if not r.ok]
    delete_labels(client, ["canary"])
```

## What to tell the user on failure

- **Label rejected as dynamic/builtin:** Fleet only allows manual labels to
  be assigned this way. Tell the user to check the label's type in the
  Fleet UI, or create a new manual label.
- **Duplicate on create (409):** a label with that name already exists.
  Use it as-is or pick another name.
- **Builtin label on delete (422):** Fleet never allows deleting builtin
  labels.
- **`no label named ...` on delete:** names are exact and case-sensitive.
- **Host not found:** the identifier didn't match any host's hostname,
  serial, or UUID. Suggest double-checking the identifier in the Fleet UI.
- **403 / permission error:** the token's role must be admin, maintainer,
  technician, or gitops — observers cannot modify labels. Creating and
  deleting labels may need admin or maintainer.

To try any of this without touching a real server, use the `fleet-sandbox`
skill.
````

</details>

## Expanding from here

Now that you've seen how to do this with labels, think through what other things you might need to manipulate frequently.

You could do the same to get vulnerabilities, upate software, manage policies, etc.

If you are relying on an AI agent to remove the complexity of managing these features, it would make sense that you would want deterministic outcomes, especially if you are operating in a regulated environment.

## Where to start

Take one task you have asked an agent to do more than three times.

Ask your best model what a script for it would need to handle. Do not let it write code yet. Then have a mid tier model
write the script and a skill that calls it. Then set `model: haiku` and `context: fork` in that skill's frontmatter and
see if anything breaks.

If it works, you just moved a recurring task off your most expensive model permanently.

If it breaks, you learned that the task still has ambiguity in it, and you now know exactly where to look.

Either outcome is a win.

## What's next?

I think the next logical step is to have these run locally.

I have tinkered with the idea of creating a local HTTP server that translates prompts to API calls and back so that we
can interact with the local Apple Foundation models.

Unfortunately, my M2 Pro MacBook Pro has a limit of 4,096 tokens at one time. Apparently, beginning with the M3-series,
Apple has increased the limit.

In the meantime, I'm quite happy with this workflow, and I'm hopeful that it helps you too.
