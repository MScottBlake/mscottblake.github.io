---
title: "Getting Started with Gemini CLI"
date: 2026-01-23 00:54:00 -0500
categories:
  - AI
tags:
  - Gemini
  - Gemini CLI
  - Claude Code
  - Coding Assistant
  - Agentic Workflows
  - MCP
  - Model Context Protocol
  - Cloud AutoPkg Runner
youtubeId: 1AF5pFGwRTM?list=PL4cUxeGkcC9h-AKdBSCRpqjD3y6T7Xgrb
---

In my [last post]({% post_url 2026-01-11-local-llms-on-spare-apple-silicon-a-cautionary-tale %}), I wrote about my ambitious but ultimately disappointing attempt to run local LLMs on my spare Apple Silicon hardware. While I remain optimistic about the future of local AI, that experience made one thing clear: for practical, day-to-day coding assistance _right now_, I needed to look elsewhere.

I have been following the _agentic workflow_ buzz for a while and decided to try it out. To be specific, when I say agentic workflow, I'm referring to tools that can act as a coding partner in the command line terminal, not just autocomplete a block of code.

I first looked at Anthropic's [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview), which seems like the current front-runner in the space. However, as a hobbyist, I couldn't justify the cost since it lacks a free tier with access to the tool.

## Gemini CLI Setup

I decided to give Gemini CLI a shot. To get up and running, I followed a [YouTube tutorial by Net Ninja](https://www.youtube.com/playlist?list=PL4cUxeGkcC9h-AKdBSCRpqjD3y6T7Xgrb). I found this series to be excellent. The videos were not too long, they were descriptive, and they went through a lof of content. I was able to familiarize myself with the setup and basic capabilities from these videos alone.
{% include youtubePlayer.html id=page.youtubeId %}

After playing around with it for a few days with minimal configurations, I have to say it works surprisingly well.

I've never used Claude Code, so I can't offer a direct comparison. However, if you are looking to experiment with tools like this, Gemini CLI is a strong contender and probably good enough.

## From Chatbots to Agents

Up until very recently, my AI usage was strictly limited to chat bots (like ChatGPT or Gemini via their web interface). My workflow usually involved using them during the design phase of a project, especially for complex architectural decisions or implementing patterns I wasn't familiar with.

For instance, when I was building [Cloud AutoPkg Runner]({% link _pages/tag-archive.md %}#cloud-autopkg-runner), I used a chat bot to help me design the cache plugin system because I didn't know how to build optional dependencies. I would go back and forth, refining the design until I truly understood what was needed. Often, I wouldn't even have the AI output code. The value was in helping me process the pros and cons before I wrote a single line. That "measure twice, cut once" approach saved me a significant amount of wasted time and effort.

### The Context Gap

The biggest downside to that chatbot-centric workflow was context.

It was often difficult and tedious to build up the necessary context the AI needed in order to produce quality responses. You have to paste in file contents, explain the directory structure, describe your constraints, and defend your coding styles repeatedly.

This is the hole that I think Gemini CLI fills perfectly.

## Why This Matters

Gemini CLI and other code agents run directly within the project directory, they have immediate access to the entire codebase. They can "see" the project structure, understand the existing coding styles, read the linter rules, and respect the project's constraints automatically. They remove the friction of context-switching and copy-pasting, allowing for a much more fluid and integrated development experience. The project rules can also be defined in a file that lives in the repository, so you don't even need to repeat your constraints through multiple coding sessions.

I'm approaching this primarily as a hobbyist, but I feel that proficiency with these tools is quickly becoming table stakes for developers. Even if you don't use them for everything, familiarizing yourself with AI coding workflows is a smart way to stay ahead of the curve. If you spend any time writing code for your job, it is worth your time to investigate these tools.

## Not Just "Vibe Coding"

There is a term floating around called _vibe coding_, which implies you can just vibe with the AI and it will magically build your app. While catchy, I think this is misleading.

These tools cannot do the work entirely by themselves. To get truly useful output, they need a human who would be able to do it themselves if they were given enough time.

An agent like Gemini CLI is a force multiplier, not a replacement for fundamental skill. If you don't understand the underlying code, architecture, or security implications, you won't be able to effectively guide the agent or verify its work. You need to know what "good" looks like to ensure the AI delivers it.

I am thinking about writing a follow-up post to share some of the rules that I have found useful. Let me know if this is something that you're interested in reading.

## Wrap Up

I'm excited to see where this goes and how it changes the way I build software. I am particularly interested in learning more about custom tools and MCP (Model Context Protocol) servers. As a Mac Admin, I think that is where I am likely to find the most value from these workflows. They are the "glue" that can tie different systems and APIs together into a cohesive, automated experience.

If you've been on the fence about trying terminal-based AI agents, I highly recommend giving Gemini CLI a spin. It might just change your workflow for the better.
