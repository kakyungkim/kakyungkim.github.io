---
layout: single
title: "I Gave Up Reading the News and Built a Team of AI Agents Instead — econ-radar Design Notes"
date: 2026-06-10 09:00:00 +0900
lang: en
ref: econ-radar-agent-harness
featured: true
categories: [AI & Tooling]
tags: [Claude Code, AI Agents, Agent Harness, Newsletter, Building in Public]
permalink: /en/2026/06/10/econ-radar-agent-harness/
excerpt: "I kept starting and stopping with economic news — not from lack of interest, but from the sheer volume of it. So I stopped trying to do it by hand and built a team of AI agents to do it for me."
---

{% include lang-switch.html %}

Hi — I'm Ka-Kyung, a bioinformatics researcher working across clinical data, genomics, and machine learning.

It never lasted. A few days in, and the rhythm would break. It wasn't a motivation problem — it was a volume problem. Too many sources, too much time spent just organizing before any actual thinking could happen.

What I gave up wasn't reading economic news. What I gave up was collecting and sorting it by hand every day. So I shifted the question — from "how do I use AI?" to "how do I turn this repetitive work into a system?" I handed the gathering and organizing to a team of agents with separated roles, and kept only the judgment and review for myself. That's how econ-radar came together.

## What econ-radar is

econ-radar is a personal daily that scans trusted business press and organizes it through three lenses — industry structure, investing, and market — with extra weight on pharma, biotech, and AI.

Everything goes into an Obsidian Vault. The goal isn't a one-and-done news summary. It's a knowledge asset that grows over time: something I can use to track how an industry has moved, or as raw material when I sit down to write a longer analysis.

I also slipped in a small personal corner — each day's output pulls a few business-English expressions from the articles, with example sentences and audio. This is a tool I built for my own use, so these kinds of personal wants show up throughout.

## Not one AI — a small team

From the start, I wasn't trying to write one clever prompt. I thought it would be more robust to split the work the way a team would.

- A **scout** gathers the news — facts only, no interpretation, foreign articles translated into Korean.
- Two **analysts** run in parallel: one on industry structure and investing, one on companies to watch.
- An **editor** weaves them into a single newsletter.
- A **style critic** catches awkward phrasing right before publishing.
- A **renderer** turns it into readable HTML.
- A **curator** files it into the vault with tags and links.

Each step writes its output to a file; the next step reads that file. Nothing lives only inside a chat window — all the intermediate work is recorded and traceable. That simple rule turned out to make the whole pipeline noticeably more reliable. When something breaks, it's easy to see where, and easy to improve just that one step.

## What I learned building it

### 1. "This feels off" has to become a rule

Early output kept sounding like AI. "Write it more naturally" would help for one turn, then drift back.

So I collected sentences from real Korean business articles and pinned them as a style sample — along with concrete rules like *use past tense for events* and *always include the local-currency amount*. Now the style critic corrects every draft against that sample.

Looking back, the real work of harness engineering was turning a subjective complaint into a reproducible rule. That loop repeated itself more often than anything else.

### 2. The point of automation is leaving only the judgments

Collection, analysis, and drafting are automatic. But anything that goes out the door still requires my sign-off. Automation isn't about removing the person — it's about leaving only the decisions that actually require one. The most valuable part of reading news isn't the gathering or the summarizing; it's deciding what matters and what to make of it.

### 3. Separate fact from opinion

Each item carries an objective **Key Point** (why it matters) and a subjective **Insight** (my read), split by color.

I mixed them early on. When I went back to reread, the line between what happened and what I thought about it had blurred. Keeping them visually separate makes both stronger.

### 4. Build to accumulate, not just to read

Summaries are easy to read today and hard to find next month. But if each day's output stacks in the vault with topic links, it becomes useful later — when writing a longer piece, or when tracking how an industry has evolved over months. The goal was information that compounds, not information that gets consumed.

### 5. I was only watching the sell side

The third lens started as "companies to watch" — which company is growing, where capital is flowing, what technology is getting attention.

After running it for a while, something felt missing. The same news story reads differently depending on whether you're asking *who's investing* versus *who's actually using this*. So I added a demand layer.

Pharma and biotech make this gap especially visible. The patient who takes the drug, the physician who prescribes it, and the insurer or government that pays for it all evaluate it by different criteria. A drug can be scientifically excellent and still fail if any one of those groups says no. I kept seeing that pattern confirmed in the news — a strong clinical result that stalled on reimbursement, a product that prescribed physicians rarely reached for.

Now company analysis includes a demand read alongside the supply-side view. It's been the most meaningful change I've made to the harness.

## This actually started in research

The agent harness idea came to me first on the research side. In a team study, I proposed reading papers through two lenses — academic and industry — and rendering the output on the web so figures and equations show up inline. Running that repeatedly is where it clicked: repetitive knowledge work can be decomposed into roles and steps. The judgment at the end still requires a person. Most of what leads up to it can be structured.

econ-radar is that same idea, moved into a personal project.

## See it live

- [econ-radar archive](https://kakyungkim.github.io/econ-radar/)
- [a sample issue](https://kakyungkim.github.io/econ-radar/2026-06-10.html)

The format is still changing — I keep finding things that don't work until I've actually run them for a few days. That's most of what I'm learning.

## Next

I'll keep writing here about running econ-radar — the things that broke, the decisions that turned out to matter, and what worked. If you're thinking through similar problems, hopefully some of it is useful.
