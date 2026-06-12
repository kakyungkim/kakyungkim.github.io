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

I changed that two days in.

## A gate needs someone standing at it

The idea wasn't wrong. The problem was that someone had to show up at the gate every single day.

Two days of running it made the pattern clear: the newsletter would finish in the evening, and I'd get to it later, or push it to the next day. News from today is most useful today.

The rule "nothing goes out until I confirm" was slowing itself down.

## Flipping the direction of trust

Now a cloud routine runs at 18:30 KST every day — collecting news, publishing the blog post, sending a Telegram message when it's done.

"Nothing goes out until I confirm" became "things go out, and I pull them down if something looks off."

Taking down an HTML file is something I can always do. Approving every issue takes willpower every day. Two days was enough to know which one would last.

## The cloud doesn't touch local files

One thing tripped me up during the switch.

The routine ran fine and pushed to GitHub, but there were no new files on my laptop. I assumed it had failed and ran the pipeline manually. That produced two copies of the same day's output.

The cause was simple: the cloud only pushes to `origin/main`. My local machine doesn't update until I run `git pull`.

Now when something looks off, I check `git log origin/main` first, and verify the remote state before I run anything manually.

## One Telegram channel

I made a dedicated Telegram channel for publish notifications. When an issue goes out, a message arrives with three headline lines and a link.

It sounds minor, but without it I'd often forget an issue had even published. A day with no message means something went wrong — which turns out to be a useful monitoring mechanism.

Full automation doesn't mean you stop paying attention. It means you pay attention to different things. Less "did it publish?" and more "was what it published any good?"

---

You can read all published issues at the [econ-radar archive](https://kakyungkim.github.io/econ-radar/).
