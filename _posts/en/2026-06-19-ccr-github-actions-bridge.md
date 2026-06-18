---
layout: single
title: "When your cloud AI agent can't reach Telegram — bridging with GitHub Actions"
date: 2026-06-19 09:00:00 +0900
lang: en
ref: ccr-github-actions-bridge
categories: [AI & Tooling]
tags: [Claude Code, CCR, GitHub Actions, Telegram, AI Agents, Building in Public, Automation]
permalink: /en/2026/06/19/ccr-github-actions-bridge/
excerpt: "The CCR cloud sandbox blocks outbound HTTPS to Telegram. It took me a while to figure out why, and the fix turned out to be simpler than I expected."
---

{% include lang-switch.html %}

In [an earlier post]({{ site.baseurl }}/en/2026/06/12/econ-radar-auto-publish/), I said econ-radar publishes automatically every day — including a Telegram notification to the channel. For a few days, the notifications just didn't come.

## I assumed it was a token problem

I'd written Telegram push code directly into the CCR (Claude Code Routine) prompt. Token, chat ID, `sendMessage` call. It worked locally. In the cloud, the connection failed entirely.

I checked the token. I checked the chat ID. Both looked right. After a few more days of debugging, I found the real issue: the Anthropic cloud sandbox that runs CCR blocks outbound HTTPS to `api.telegram.org`. This isn't a misconfiguration — it's a deliberate constraint on the environment.

## The fix: agent writes a file, someone else does the sending

If the agent can't make the call directly, hand it off to something that can. GitHub Actions runs on standard Ubuntu with normal internet access.

The flow looks like this:

```
CCR (cloud agent)
  → writes message to vault/push/YYYY-MM-DD.md
  → git push origin main

GitHub Actions (no network restrictions)
  → detects change in vault/push/*.md
  → calls Telegram API → sends to channel
```

The agent only writes a file and pushes. The git push acts as an event bridge.

## Implementation

### send_push.py

Reads the push file and sends it to Telegram. Token and chat ID come from environment variables only — never hardcoded.

```python
import os, sys, requests

BOT_TOKEN = os.environ.get("TELEGRAM_BOT_TOKEN", "")
CHAT_ID   = os.environ.get("TELEGRAM_CHAT_ID", "")

if not BOT_TOKEN or not CHAT_ID:
    print("[ERROR] environment variables not set")
    sys.exit(1)

date = sys.argv[1]
text = open(f"vault/push/{date}.md").read()

requests.post(
    f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage",
    json={"chat_id": CHAT_ID, "text": text, "parse_mode": "Markdown"}
)
```

### The GitHub Actions workflow

Only triggers when something in `vault/push/` changes.

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'vault/push/*.md'

jobs:
  send:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Detect date
        id: detect
        run: |
          BEFORE="${{ github.event.before }}"
          if [ "$BEFORE" = "0000000000000000000000000000000000000000" ]; then
            BEFORE="HEAD~1"
          fi
          DATE=$(git diff --name-only "$BEFORE" HEAD \
            | grep -E '^vault/push/[0-9]{4}-[0-9]{2}-[0-9]{2}\.md$' \
            | sed 's|vault/push/||; s|\.md||' \
            | sort -r | head -1)
          echo "date=$DATE" >> "$GITHUB_OUTPUT"

      - name: Send Telegram push
        if: steps.detect.outputs.date != ''
        env:
          TELEGRAM_BOT_TOKEN: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          TELEGRAM_CHAT_ID: ${{ secrets.TELEGRAM_CHAT_ID }}
        run: python3 scripts/send_push.py ${{ steps.detect.outputs.date }}
```

Add `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` to GitHub Secrets and you're done.

## A second bug I only found in production

It worked fine for a day. Then the notifications stopped again.

This time the cause was different. CCR was bundling multiple commits into a single `git push`:

```
b38e97f  econ-radar: 2026-06-16 auto-generated  ← vault/push file is here
8834dde  topics MOC update
027155e  semiconductor MOC update
4520dc9  topics MOC final sync                   ← HEAD
```

My original detection logic used `HEAD~1 HEAD` — comparing only the last commit. It looked at `4520dc9` vs `027155e`, found no `vault/push/` changes, and silently skipped the send step. The job showed "success" because no step actually failed.

The fix: use `github.event.before..HEAD` instead of `HEAD~1 HEAD`. This covers all commits included in the push, not just the last one. `fetch-depth: 0` is needed so git can diff that full range.

The root cause was that I tested with single-commit pushes. In real operation, the agent creates several commits before pushing. Worth checking how your agents actually structure commits before writing the detection logic.

## Finding a channel's chat_id

One more thing that tripped me up: `getUpdates` returns an empty array for channels. Use `getChat` instead:

```
https://api.telegram.org/bot{TOKEN}/getChat?chat_id=@channelname
```

The `result.id` field is what you want. For channels it's a negative number starting with `-100`.

## This pattern extends beyond Telegram

Anything CCR can't reach directly can be handled the same way — Slack, Discord, Notion, internal APIs. The agent writes a file, Actions reads it and makes the external call.

One sentence: the agent produces, Actions delivers.
