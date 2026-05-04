---

title: "Stop Prompting, Start Orchestrating"
date: 2026-05-03 14:08:00 -0400
toc: true
toc_sticky: true
categories:
  - AI Tooling
tags:
  - Agentic Coding
  - Claude Code
  - Gemini CLI
  - LLM Harness
  - Skills-based Workflows
  - Deterministic Agents
excerpt: "Agentic coding succeeds through tighter reins rather than louder prompts.
Learn how a 20-line config file turns Claude or Gemini from chatty co-pilot into a deterministic, test-running, skill-bound teammate."

---

## "AI is Dead, Long Live the Harness"

No, AI isn't actually dead, but as a standalone term, it's relatively meaningless.

Everything is "AI" now, which means the word no longer tells you anything useful about how the system actually works. In
software development, the shift isn't happening in the models. The shift is in how we use them.

We have entered the age of the harness, and therefore the age of controlled execution.

A harness provides a full execution environment, extending far beyond the utility of a simple tool.

It gives a model:

- Read access to your codebase, configs, and history
- Write access to propose and apply changes
- Execution to run tests, scripts, and commands
- Feedback loops to verify results

Without it, you're chatting.

With it, you're collaborating. It becomes a partner that understands your codebase, your standards, and your intent.

## Defining terms

For this post, I want to define a few terms so everyone is on the same page.

| Term | Definition |
|------|------------|
| AI | A marketing term for a collection of technologies. It's too vague to be useful, so I'll avoid it. |
| LLM / Model | The underlying engine. If you go to Anthropic, Gemini, or ChatGPT in a browser, you are interacting with a model (Claude 4.6 Sonnet, Gemini 3.1 Flash, GPT-5.5). |
| Agent | The operational unit that uses a model to perform work. It can plan, act, and iterate by reading code, making decisions, executing tools, and evaluating results. The model provides reasoning; the agent applies it. |
| Agentic coding | A workflow where an agent executes multi-step tasks without requiring step-by-step human input. |

### Visualizing the Agentic Loop

This diagram clarifies how these components interact:

{% include figure image_path="/assets/images/posts/AgenticCodingLoop.png" alt="Conceptual diagram of the Agentic
Coding Loop" caption="Figure 1: The 'Agentic Coding Loop' where the Harness orchestrates context and action, and the
Model provides reasoning." %}

## Context is Everything

Early tools like Copilot worked with a narrow slice of context. They could usually only access a single file. Therefore,
the model optimized for local correctness. It literally didn't know any better.

The result was predictable. Suggestions looked right, but didn't work when integrated back into the codebase.

This was the "context gap." Developers weren't just writing code, they were constantly explaining their codebase to a
tool that couldn't see it.

A CLI-based harness is a total shift. It runs on your machine. When you ask questions, it does its own homework. It
explores your project tree, scans your git history, and reads your `pyproject.toml`, `package.json`, or `Gemfile` to
understand how your project works.

That extra context makes interactions *potentially* meaningful, if it's used well.

## The Context Junk Drawer: More isn't always better

If context is the fuel, it's tempting to think that more is always better. If 10 files is good, 100 must be better,
right?

Actually, no. Flooding a session with context often makes the agent less effective.

Modern models have massive context windows, but their "attention" is finite. When you flood the harness with log files,
build artifacts, and stale documentation, the important stuff gets lost. It's like trying to read a technical manual in
a loud room. You may catch the main idea, but you'll likely miss the subtle details.

This is one of the fastest ways to degrade output quality.

> **Context Rule:**
> Treat context like a scalpel, not a net. If the agent is drowning in noise, the outcome is prioritization failure.
{: .notice--danger }

Models don't "understand" context, they prioritize it. When everything is included, prioritization breaks down.

At that point, failure isn't randomness, it's prioritization failure.

To keep your agent sharp, be surgical. Only put in what's necessary to get the job done. If your agent is drifting, stop
drowning it in noise.

This isn't always something you explicitly control. The harness is often gathering context on your behalf by running
searches, scanning files, and pulling in results.

That makes it even more important to be intentional about what the agent *should* look at, not just what it *can* look
at.

## Configuration is the Contract

To stop getting generic responses, you need to encode expectations directly into the system.

> **The Golden Rule of Constraints:**
> If you find yourself correcting the same behavior twice, that behavior is ambiguous. Ambiguity is not a prompt
> problem; it is a configuration failure. The constraint belongs in your `CLAUDE.md`, `GEMINI.md`, or `AGENTS.md`.
{: .notice--warning }

In this workflow, configuration is just as important as code. These tools work best when you treat them like a new team
member who needs clear, consistent documentation.

### Configuration Files

Most harnesses look for a Markdown file that defines how the agent should behave. The filenames vary
(`CLAUDE.md`, `GEMINI.md`, `AGENTS.md`), but the idea is the same.

Think of it as a contract that defines how the agent is allowed to behave.

You can define rules at multiple levels. Personal preferences that hold true for every project live in your home
directory. Rules that are specific to a single codebase live in the root of the project.

Your global file might say:

```markdown
- prefer small, composable functions
- always add type hints
- avoid unnecessary abstraction
- prefer guard clauses over nested logic to keep the code flat and readable
```

Your repo file might say:

```markdown
- use this logging library
- follow this error handling pattern
- structure modules this way
- validate changes with pre-commit: `pre-commit run --all-files`
```

The agent merges both.

### What actually belongs in these files?

Don't overthink the format. Focus on constraints that reduce ambiguity:

- Coding standards and patterns
- Architectural principles
- Tooling expectations (linters, formatters, scripts)
- Things you *never* want it to do

The goal isn't documentation for humans, it's removing degrees of freedom for the agent.

Keeping these rules in your repo ensures every session starts with the same expectations, but expectations alone aren't
enough. You still need a way to enforce how work gets done.

That's where skills come in.

## Determinism over "Vibes"

LLMs can be inconsistent. Ask for the same fix twice and you may get two different approaches.

The way out of that isn't better prompting, it's better structure.

> Prompting still matters, but structure matters more.
{: .notice--info }

Instead of asking for outcomes, define **skills**.

A skill is a repeatable workflow.

1. Find the failing test
2. Reproduce the issue
3. Apply a fix
4. Verify the result

The key difference is this: the model is no longer deciding *what to do next*. The workflow is.

> **Skill Definition:**
> Don't use natural language to ask for a known workflow. Instead, codify that workflow as a 'Skill.' You are
constraining LLM reasoning with proven system behavior.
{: .notice--info }

You're defining the process instead of hoping the model infers it.

### Skills in Practice

If this still feels abstract, here’s what it looks like in a real tool.

In Claude Code, a skill is a specific folder with a `SKILL.md` file that defines a workflow.

For example, instead of asking:

> "Can you review this code?"

You define a skill:

```markdown
# Code Review Skill

When asked to review code:

1. Run tests first
2. Check for type safety issues
3. Validate error handling patterns
4. Ensure functions follow single-responsibility
5. Suggest improvements with diffs
```

Every time you invoke that skill, the process is consistent, and more consistent, less variable output leads to more
deterministic outcmes.

Different tools will implement this differently, but the concept is the same.

### Pro tip: Define skills to call your scripts

The most reliable setup is when skills delegate to code you already trust.

Do you have a script that checks links, builds artifacts, or validates output? Use it.

Don't ask the model to reinvent that logic on each request. Give it a skill that knows how to run the script.

**The Script (`scripts/check_links.sh`):**

```bash
#!/bin/bash
bundle exec htmlproofer ./_site --check-html --disable-external
```

**The Skill Definition:**
> "When I ask you to check for broken links, run `scripts/check_links.sh` and tell me which files have 404 errors."

This combines the reasoning of an LLM with the reliability of a script you already trust.

## Stop grepping, Start Understanding

I'm using traditional tools like `grep` and `find` less than I used to, but I haven't stopped using them.

We are seeing a shift in who operates the tools, even as the tools stay the same.

In a harness-driven workflow, the agent is often the one running searches. It will scan your codebase, open files, and
try to build context before taking action.

In small projects, that's fine.

In large codebases, it can get expensive — fast.

Every search, every file read, every chunk of context costs tokens.

In large codebases, the default failure mode isn't bad code generation, it's unbounded exploration.

The agent keeps searching, opening files, and trying to infer intent. Yoe end up paying more for the agent to figure out
what to do and less for actual execution.

This is where the distinction starts to matter:

- Text search (`grep`, `find`) is cheap, fast, and predictable, but limited
- Structured context (MCP, indexing, custom tools) is richer, but more expensive and more complex

MCP and similar approaches give the agent a higher-level view of your code:

- where something is defined
- how it's referenced
- how pieces relate to each other

They can dramatically reduce the number of exploratory steps, but it's not free.

There's overhead in building and maintaining that context, and in many cases, a simple script or targeted search will
get you to the answer faster and cheaper.

The mistake is treating MCP as a default.

Use it when the problem requires understanding relationships across the codebase.

Otherwise, give the agent a narrower path:

- a script to run
- a known file to inspect
- or a constrained search

The goal isn't to give the agent more visibility, it's to give it just enough to act without wandering.

This is another form of constraint.

The less the agent has to explore, the more predictable its behavior becomes.

Constraints, skills, and structure aren't just about better results, they're about stopping that exploration early.

## The New Metric: Articulation

Lines of Code was never a great metric, but now it's actively misleading.

Ambiguity has replaced typing speed as the primary bottleneck, and ambiguity compounds across every step the agent
takes.

If your request can be interpreted multiple ways, the agent *will* pick one and *you* will own the result.

That shifts the value of a developer:

- Can you describe the problem clearly?
- Can you define what "done" actually means?
- Can you recognize when the output is wrong?
- What about when it looks right?

The people who do this well aren't just "good with AI." They're precise communicators.

They can take a vague idea and turn it into something an agent can execute without guessing.

And just as importantly, they can validate the result.

When an agent hands you 200 lines of code, can you prove it works? Can you design the tests that keep it honest? That's
where seniority is moving. Defining, constraining, and verifying is the process. More code is the result.

Conversely, ambiguous definitions lead to long, back-and-forth conversations, multiple iterations, and code that is hard
to follow.

In other words, the better you define the problem, the less the agent has to guess, and the less it guesses, the more
*deterministic* your workflow becomes.

## Where to start

Start small.

Create a `CLAUDE.md` or `GEMINI.md` file in your project. Add a few rules about how you structure code or handle errors.

Then take one repetitive task and turn it into a skill.

You'll notice the shift immediately. The output becomes more consistent. You'll see less guesswork and more execution.

That's the real change happening right now.

The developers who thrive in this model won't be the ones who can generate the most code.

They'll be the ones who can:

- articulate the problem clearly
- define the end state precisely
- and validate the result without hesitation

The harness doesn't replace those skills, it amplifies them.

The age of the harness is here.

Are you still prompting, or are you orchestrating?

## More information

- [Agent Skills](https://agentskills.io/home)
- [The Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf)
- [Gemini Agent Skills](https://geminicli.com/docs/cli/skills/)
- [Codex Agent Skills](https://developers.openai.com/codex/skills)
