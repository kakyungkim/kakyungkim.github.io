---
layout: single
title: "I dropped manual approval after two days — going fully automatic with econ-radar"
date: 2026-06-12 22:00:00 +0900
lang: en
ref: econ-radar-auto-publish
categories: [AI & Tooling]
tags: [Claude Code, AI Agents, Agent Harness, Newsletter, Building in Public, Automation]
permalink: /en/2026/06/12/econ-radar-auto-publish/
excerpt: "I built it so nothing would go out without my approval. Two days later I dropped that, and it runs better."
---

{% include lang-switch.html %}

In [the first post]({{ site.baseurl }}/en/2026/06/10/econ-radar-agent-harness/), I said anything going out the door would wait for my approval. The newsletter could be ready on the cloud, but posting to the blog still required me to type `./deploy.sh`.

At the time, that felt like the obvious choice.

Two days later, I changed it.

## A gate needs someone standing at it

The idea wasn't wrong. The problem was that someone had to show up at that gate every single day.

Two days in, the pattern was obvious: the newsletter would finish in the evening, and I'd get to it later, or push it to the next day. News from today is most useful today.

The rule "nothing goes out until I confirm" wasn't protecting quality — it was just delaying publication.

## I switched to automatic publishing

Now a cloud routine runs every day at 18:30 KST. It handles everything from collecting news to publishing the blog post, then sends a Telegram notification when it's done.

The old model was **"nothing goes out until I confirm."** The new model is **"things go out — and if something looks wrong, I take it down or fix it."**

Taking down an HTML file is something I can always do. Approving every issue takes willpower every day. Two days was enough to know which one would last.

## The cloud doesn't touch local files

There was a small incident during the switch.

The routine ran fine and pushed to GitHub, but there were no new files on my laptop. I assumed it had failed and ran the pipeline manually. That produced two copies of the same day's output.

The cause was simple: the cloud only pushes to `origin/main`. My local machine doesn't update until I run `git pull`.

Now when something looks off, I check the remote state first. Checking `origin/main` before running anything manually has become a habit.

## What one Telegram channel turned out to be worth

I made a separate Telegram channel for publish notifications. When an issue goes out, a message arrives with a few headline lines and a link.

I thought it would be a minor convenience. It turned out to matter more than that.

Without it, I'd often forget an issue had even published. Now, a message means everything ran fine; no message means something went wrong somewhere. I get a natural status check without any monitoring dashboard.

## What changed after automating

It's easy to assume that full automation means you stop paying attention. That's not quite right. The attention just shifts.

Before, I was asking **"did it publish today?"** Now I occasionally ask **"was what it published any good?"** Managing whether it goes out became managing what goes out.

That's actually what I wanted from the start. I built econ-radar not to have something to post every day, but to have more time to actually read and think about the news.

---

You can read all published issues at the [econ-radar archive](https://kakyungkim.github.io/econ-radar/).
