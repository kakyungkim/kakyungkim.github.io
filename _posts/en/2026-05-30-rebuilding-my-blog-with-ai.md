---
layout: single
title: "I Rebuilt My Dead Blog With an AI Pair — Here's What Actually Broke"
date: 2026-05-30 09:00:00 +0900
lang: en
ref: rebuilding-my-blog-with-ai
categories: [AI & Tooling]
tags: [Jekyll, GitHub Pages, Claude Code, AI Pair Programming, minimal-mistakes]
permalink: /en/2026/05/30/rebuilding-my-blog-with-ai/
excerpt: "Months ago I gave up on this blog after the environment setup defeated me. This week I sat down with an AI coding agent and we found that almost nothing was wrong with my writing — and almost everything was wrong with my build."
---

{% include lang-switch.html %}

Hi — I'm Ka-Kyung, a bioinformatics person who spends most days somewhere between clinical data, genomics, and machine learning. I started this blog months ago, wrote a couple of posts, and then it quietly died. Not for lack of ideas, but because I could never get the thing to *build and deploy*. Every attempt ended in a red X on GitHub and a stack trace I didn't have the patience to decode.

This week I tried something different: I sat down with an AI coding agent and treated the broken blog as a debugging session instead of a chore. A few hours later it was live, bilingual, and running a proper theme. Here is what was actually wrong — because none of it was what I'd assumed.

## Symptom 1: the theme wouldn't download

My local builds died instantly with:

```
SSL_connect ... certificate verify failed (unable to get certificate CRL)
```

I had assumed this was "my Ruby is broken" and had even dropped a `cacert.pem` into the repo as a superstitious fix. The real cause was narrower: my config used `remote_theme`, which makes Jekyll **download the theme from GitHub on every build**, and my local Ruby's OpenSSL refused that connection.

The fix wasn't to wrestle with certificates. It was to stop downloading the theme at all — the same theme ships as a gem (`minimal-mistakes-jekyll`). One line in `_config.yml` and the network call simply disappeared. Lesson: when something fails, first ask *what is it even trying to do?* Mine was making a network request I didn't need.

## Symptom 2: CI failing in nine seconds

My GitHub Actions runs failed in about nine seconds — too fast to be a real build. The logs from back then had long since expired, so we couldn't read the original error. Instead of guessing, we reproduced the build locally until it was green, then looked at the workflow with fresh eyes.

The culprit was a leftover step that force-installed an old, pinned version of Bundler on top of the one the lockfile expected. It was redundant — the Ruby setup action already handled dependencies — and the version mismatch was a landmine. Deleting the step made the workflow both simpler and correct. Lesson: **redundant "just in case" steps are not free**; each one is a place to fail.

## Symptom 3: the posts vanished

The strangest one. After fixing the theme, the site built — but my actual blog posts were *gone* from the output. The home page was empty.

The cause was an old multi-language plugin I'd added to get Korean and English versions. On a modern Jekyll it silently dropped every post and mangled the URL structure. Rather than nurse a fragile dependency, we removed it and rebuilt bilingual support from plain Jekyll: posts carry a `lang` field, and each language has its own home page that filters to its own posts. Fewer moving parts, and now I can actually reason about how a URL is produced.

## What I took away from pairing with an AI

A few things surprised me about the process itself:

- **It read logs I would have skipped.** My instinct was to try random fixes; the agent's first move was always to reproduce and read the actual error. Most of my "mystery" failures had the cause printed right there.
- **It didn't trust the guide — and the guide was wrong.** One setup instruction pointed at a package that doesn't exist anymore. A confident wrong instruction had been quietly blocking me for months.
- **It insisted on verifying.** Nothing was declared fixed until a build was green locally *and* in CI. That discipline — "show me it works" — is exactly what I'd been skipping on my own.

None of these are AI-specific lessons, really. They're just good engineering habits that are easy to drop when you're frustrated and alone with a red X. The AI's main contribution was patience and a refusal to guess.

<div class="notice--info" markdown="1">
**In short**
1. My GitHub Pages blog sat unpublished for months — not for lack of writing, but because the build kept failing.
2. With an AI agent, the real culprits were mundane: a theme downloaded over a broken SSL connection, a redundant CI step, and an old plugin silently dropping every post.
3. Every fix was "do less," and the lessons were plain: read the logs, distrust the guide, verify before calling it done.
</div>

## What's next

The blog builds, deploys, and speaks two languages now — so I can finally get back to the part I wanted to do all along: writing. I'll be posting about the projects I build and the ideas I keep chewing on, somewhere in the space between data science and life science.

If you've got a half-dead side project gathering dust because the *setup* beat you, consider this your nudge. The writing was never the hard part. Thanks for reading.
