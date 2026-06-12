---
layout: single
title: "I Handed My Daily Econ Newsletter to a Team of AI Agents — Building econ-radar"
date: 2026-06-10 09:00:00 +0900
lang: en
ref: econ-radar-agent-harness
featured: true
categories: [AI & Tooling]
tags: [Claude Code, AI Agents, Agent Harness, Newsletter, Building in Public]
permalink: /en/2026/06/10/econ-radar-agent-harness/
excerpt: "I wanted to keep up with pharma, biotech, and AI every day — but doing it by hand never lasted more than a few days. So I handed the collecting, analyzing, editing, and reviewing to a team of AI agents, and kept only the judgment for myself."
---

{% include lang-switch.html %}

Hi — I'm Ka-Kyung, a bioinformatics researcher who spends most days somewhere between clinical data, genomics, and machine learning.

I wanted to keep up with where pharma, biotech, and AI are heading — every single day. But skimming trustworthy sources, organizing them, and interpreting them by hand doesn't survive more than a few days. It isn't a willpower problem; it's a volume problem. So I reframed the question — from "how do I use AI?" to "how do I engineer this repetitive work into a system?" I handed the gathering and organizing to a **team of AI agents with separated roles**, and kept only the judgment and the review for myself. What came out is econ-radar — a small agent harness.

## What econ-radar is

econ-radar is a personal daily that scans trusted business press and organizes it through three lenses — industry structure, investing, and companies to watch — weighted toward pharma, biotech, and AI. Each day's output accumulates in an Obsidian vault, designed to grow later into trend reports and longer pieces.

I also slipped in a small corner that pulls a few business-English expressions from each day's articles, with example sentences and audio — mostly because I wanted to keep studying English myself. It's a tool I built for my own use, so these little personal wants show up throughout.

The key isn't "one clever prompt." It's a **team of agents with separated roles**.

## No single agent does everything

- A **scout** gathers the news — facts only, no interpretation, foreign articles translated into Korean.
- Two **analysts** work in parallel: one on industry & investing, one on companies to watch.
- An **editor** weaves them into a single newsletter.
- A **style critic** rewrites awkward sentences right before publishing.
- A **renderer** turns it into readable HTML,
- and a **curator** files it into the vault with tags and links.

Each step leaves its output as a file, and the next step reads that file. Not letting the work live only inside a chat — that small rule is what makes the harness trustworthy.

## What I learned building it

### 1. "This reads awkward" has to become a rule

At first the output kept sounding like AI. "Write it more naturally" only worked for a single turn. So I collected sentences from real Korean business articles and pinned them as a style sample — concrete rules like *events in past tense* and *amounts with their local-currency conversion*. Then the style critic fixes every draft against that sample. Turning a subjective complaint ("this feels off") into an objective asset (samples + rules) — harness engineering turned out to be mostly that, repeated.

### 2. The human only judges and approves

Collection, analysis, and drafting are automated — but anything that goes out the door waits for my approval. The point of automation isn't to remove the person; it's to leave only the *decisions* for the person.

### 3. Separate fact from opinion

Each item carries an objective "Key Point" (why it matters) and a subjective "Insight" (my read), split by color. Facts stay trustworthy; opinions stay in my voice. Blur the two and both get weaker.

### 4. Don't summarize and stop — accumulate

Summaries alone cut clicks to the source and leave me with nothing. So each day stacks into the vault and links by topic, becoming a knowledge asset that grows over time.

## This actually started in research

The "agent harness" mindset reached me first on the research side. In a team study, I proposed and extended a way to read papers not just as an objective summary but through two lenses — academic and industry — and to render the result on the web so figures and equations show up inline. That's where it clicked that repetitive knowledge work can be turned into a harness. econ-radar is that same idea, moved fully into a personal project.

## See it live

You can see econ-radar in action here:

- [econ-radar archive](https://kakyungkim.github.io/econ-radar/)
- [a sample issue](https://kakyungkim.github.io/econ-radar/2026-06-10.html)

It's still an experiment, but I run it daily and keep refining the format and the analysis structure.

## Next

It's early. I'm running it daily and tuning the voice and structure, and a public archive is on the way. I'll keep writing here about the build and what it teaches me.
