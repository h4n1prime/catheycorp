---
title: "Building Winston: How I Set Up an AI Agent That Debugged Itself"
date: 2026-01-31T14:43:00-06:00
draft: false
categories: ["technology"]
tags: ["ai", "openclaw", "automation", "self-hosting", "llm"]
description: "A Saturday morning building an AI assistant from scratch — local models, blog pipelines, efficiency audits, and a humbling self-debugging moment."
ShowToc: true
TocOpen: false
---

Last Saturday morning I decided to build something I'd been thinking about for a while: a personal AI agent that actually *does things*. Not a chatbot. Not a wrapper around ChatGPT. A full-stack assistant that manages infrastructure, writes content, monitors my email, and pushes code to GitHub.

This is the story of how that went — including the part where it broke in a hilariously ironic way.

## The Stack

The foundation is [OpenClaw](https://github.com/openclaw/openclaw), an open-source framework for building AI agents that connect to messaging platforms, run tools, and manage their own memory. I named mine Winston. Dry wit, rock-solid disposition, British butler energy minus the accent. 🪨

The interesting part isn't the chatbot layer — it's the hybrid model architecture underneath.

### Three Models, One Agent

Running everything through Claude Opus 4.5 would work great and cost a fortune. So I set up a tiered system:

- **Anthropic Claude Opus 4.5** — Main conversations with me. The heavy hitter for complex reasoning and writing.
- **Ollama (Qwen 2.5 14B)** — Running locally on a Mac Mini. Handles sub-agent tasks, heartbeat checks, and basic operations. Zero API cost.
- **Google Gemini** — Research tasks via the CLI auth plugin. Good at web search synthesis.

The idea is simple: route each task to the cheapest model that can handle it well. My direct conversations need Claude's quality. But checking my email every 30 minutes? A local 14B model handles that fine.

## The Blog Pipeline

Part of the project was building an automated blog workflow. Not just "AI writes a post" — a multi-agent pipeline where each stage uses the right model:

1. **Research Agent** (Gemini) — Searches the web, gathers sources, builds a research brief
2. **Writer Agent** (Claude) — Drafts the post from the research
3. **Editor Agent** — Polishes and humanizes the text
4. **Me** — Review and approve
5. **Publisher Agent** (Ollama) — Copies to Hugo, commits, pushes to GitHub

The site itself is Hugo with the PaperMod theme, deployed via GitHub Actions to GitHub Pages. Push to main, site rebuilds automatically. Five categories: technology, day in the life, cooking, gaming, and college football.

The whole thing took about 20 minutes to scaffold — Hugo site, GitHub repo, Actions workflow, DNS configuration, and the skill definition for the blog pipeline.

## The Efficiency Audit

Here's where it gets interesting.

After building everything, I wanted to make sure I wasn't bleeding tokens unnecessarily. So I did what any reasonable person would do: I had Winston spawn a sub-agent to audit *itself*.

The efficiency auditor (running on the cheaper Claude Sonnet model) analyzed every file that gets injected into the system prompt on every API call. The findings were brutal:

- **AGENTS.md** (the main instruction file) was ~1,800 tokens. Injected on *every single turn*.
- **SOUL.md** and **IDENTITY.md** had overlapping content — two files doing one file's job.
- **TOOLS.md** was mostly placeholder text, still getting injected every turn.
- Total bootstrap overhead: **~2,500 tokens per turn**.

At roughly 50 conversations per day, that's 125,000 wasted tokens daily just on boilerplate. The auditor produced condensed versions of every file and cut the bootstrap overhead by 80% — from ~2,500 tokens to ~500 tokens per turn.

Projected savings: **~$46/month**. Not life-changing money, but the principle matters. Every token in your system prompt multiplies across every interaction.

## The Humbling Part

Right after the efficiency audit completed, something broke. Winston started fumbling every heartbeat check — the periodic background task that checks email, calendar, and other routine items.

Every heartbeat response was nonsensical. The same error over and over: `"path required"`. Winston was trying to read HEARTBEAT.md (a simple markdown file in the workspace) but kept using the wrong tool — `memory_get` (designed for searching semantic memory) instead of `Read` (the basic file reader).

The irony wasn't lost on me. I'd just built an agent that could:
- Scaffold a Hugo blog in 20 minutes
- Create and deploy a GitHub Pages site
- Spawn sub-agents to audit its own efficiency
- Configure hybrid model routing across three providers

...but couldn't read a file it had written itself.

It took me asking "where is your soul.md file located?" for Winston to snap out of it. It used the `Read` tool to find SOUL.md, immediately realized the mistake, read HEARTBEAT.md correctly, and diagnosed its own bug in real time.

> "I've been fumbling these heartbeats badly. The `memory_get` tool is for searching memory files, not reading workspace files. I should've been using `Read` with the full path this whole time. That's why every heartbeat response was garbage."

Self-debugging. Not because I told it what was wrong — but because the right context made the error obvious.

## What I Learned

**Model routing matters more than model selection.** The smartest model in the world is wasteful if you're using it to check whether you have new email. Tier your models by task complexity.

**Bootstrap bloat is a silent killer.** Every character in your system prompt files costs you on every single API call. Audit ruthlessly. Consolidate. Cut the fluff.

**Agents are only as good as their tool selection.** Winston had the right tools available the entire time. It just picked the wrong one repeatedly — a failure mode that's very human, honestly.

**Self-auditing works.** Having an agent analyze its own setup isn't just a parlor trick. The efficiency audit found real savings and produced actionable rewrites. Use cheaper models for meta-work about the system itself.

**The best debugging happens in context.** Winston couldn't figure out its heartbeat bug in isolation. But the moment it needed to read a different file for a different reason, the correct pattern clicked and the bug was obvious. Context is everything.

## What's Next

The blog pipeline is live. The efficiency improvements are queued up to implement. Winston checks my email, tracks the news, and will eventually manage content across five blog categories.

The real test is whether this setup survives daily use — whether the hybrid model routing actually saves money without sacrificing quality, whether the blog pipeline produces posts worth reading, and whether Winston keeps learning from its mistakes.

If you're reading this, the pipeline worked. This post was drafted, reviewed, approved, and published through the exact system described above.

Welcome to CatheyCorp. 🪨
