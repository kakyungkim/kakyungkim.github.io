---
layout: single
title: "When your agent keeps repeating the same news — designing for freshness and dedup"
date: 2026-06-19 09:00:00 +0900
lang: en
ref: agent-staleness-dedup
categories: [AI & Tooling]
tags: [Claude Code, CCR, AI Agents, Newsletter, Building in Public, Automation, Agent Design]
permalink: /en/2026/06/19/agent-staleness-dedup/
excerpt: "The agent doesn't know what it wrote yesterday. Without being told, it'll run the same drug through deep analysis again today."
---

{% include lang-switch.html %}

A few days into running an agent that generates a daily newsletter, you notice a pattern: today's in-depth topic is the same drug, the same company as yesterday. A three-day-old article shows up in today's newsletter with no special marking.

The agent isn't lazy. It has inertia.

## Why this happens

Agents rebuild context from scratch each run. There's no sense of "I already covered this yesterday, I should pick something else today." Without explicit instructions, the agent picks whatever stands out in today's search results — even if it's the same subject as yesterday.

Freshness is the same problem. Web search results mix in articles from days ago. If the agent doesn't check the date, it just uses them.

I hit this directly with econ-radar. The auto-generated newsletter's in-depth topic (Today's Topic) was tebipenem, an antibiotic I'd already covered in depth the day before. It had already been sent to the blog and Telegram by the time I noticed.

## The fix: tag items at collection time

The key insight: filter at the **collection stage**, not the analysis or editing stage. If the news-scout agent tags each item, every downstream agent can act on those tags.

### The tag system

```
[background]  — article published more than 48 hours ago
[duplicate]   — already covered as an independent item in the previous newsletter, no new developments
[follow-up]   — same story, but with new developments (approval result, updated numbers, position change)
(no tag)      — fresh and new
```

I don't discard tagged items. They may still be useful as one or two sentences of context. They just don't get promoted to headline status.

### The editing rule

One rule added to the newsletter-editor prompt:

> `[background]` and `[duplicate]` items do not appear as independent entries in Today's Top 5, Today's Topic, or Deep Dive. They may only appear as 1–2 background sentences explaining a fresh item. `[follow-up]` items get brief treatment with the follow-up context made explicit.

When a section has nothing fresh, the newsletter now says "no significant signal today for this section" rather than filling the space with stale material.

### Preventing Today's Topic repetition

The in-depth topic section needs its own rule. It's the most prominent part of the newsletter — readers notice immediately when the same subject comes up two days in a row.

> Before selecting Today's Topic, read the last **3 days** of `vault/daily/` files and check the Today's Topic title in each. If it's the same drug, company, or event, choose something else.

Adding this to the agent prompt is all it takes. The prerequisite: past newsletters must be accessible to the cloud agent. In econ-radar, that means the vault is tracked in git and included in the repo the CCR checks out — which is what [tracking the vault in git]({{ site.baseurl }}/en/2026/06/12/econ-radar-auto-publish/) enabled.

## Why 48 hours

I started with 14 days. Figured anything within two weeks counted as recent. Wrong.

In economic and biotech news, things move fast enough that a two-day-old article often already has a follow-up. Cutting to 48 hours eliminated the "why is this in today's newsletter?" reaction.

The right number depends on the domain. A weekly paper digest might use 7 days; a monthly trend report might use 30. The number matters less than first defining: "what does 'fresh' mean for this audience?"

## The two principles

**Put quality rules at the collection stage.** You can filter at the editing stage, but tagging at collection means every downstream stage benefits automatically. It also keeps individual agent prompts shorter and less likely to conflict.

**Make past output readable by the agent.** To check "what did I write yesterday," yesterday's file has to exist somewhere the agent can reach. That's the practical reason to keep the vault in git and in the repo the cloud routine checks out.
