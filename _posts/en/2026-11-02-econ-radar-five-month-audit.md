---
layout: single
title: "econ-radar at Five Months: Auditing a Self-Publishing Newsletter"
date: 2026-11-02 13:00:00 +0900
lang: en
ref: econ-radar-five-month-audit
featured: true
categories: [AI & Tooling]
tags: [econ-radar, Claude Code, AI Agents, Newsletter, Automation, Fact Checking, Building in Public]
permalink: /en/2026/11/02/econ-radar-five-month-audit/
excerpt: "I audited the economy newsletter that has published itself every day since June. In October alone, five blog pages never went live while the deploy script reported success each time, and a July article's KOSPI 8,000 showed up in an October issue (the actual close was 7,003.74)."
---

{% include lang-switch.html %}

Hi, I'm Ka-Kyung Kim, a bioinformatics researcher working across clinical data, genomics, and machine learning.

Five months into running econ-radar, a Korean-language economy newsletter that publishes itself every evening, I sat down and audited it. In October alone, five blog pages never went live even though the deploy script printed "published" every time, and an October 3 issue quoted a July article's KOSPI 8,000 (the actual October 2 close was 7,003.74). I added a check that compares these market figures against real closing prices.

econ-radar runs as a Claude Code cloud routine. A team of agents handles collection, analysis, editing, style review, fact-checking, and rendering, produces one issue each evening, and sends it to the blog, Telegram, and email. I wrote about [the original design](/en/2026/06/10/econ-radar-agent-harness/), [the switch to automatic publishing](/en/2026/06/12/econ-radar-auto-publish/), and [the first 20 days](/en/2026/07/01/econ-radar-20-day-evolution/). After that I mostly assumed it was fine and stopped looking.

The audit was done on November 2, 2026. I cross-checked the cloud routine's run logs, the GitHub Actions run logs, and the blog repository's commit history. Market references were the 10-year Treasury yield and the won-dollar rate from [FRED](https://fred.stlouisfed.org/series/DGS10), and daily KOSPI closes from Yahoo Finance.

## econ-radar by the numbers

| Item | Value |
|---|---|
| Publishing period | 2026-06-09 to 2026-11-01 |
| Daily issues | 143 (4 days not generated; one later backfilled) |
| Weekly trend reports | 22 (W24 to W44) |
| Subscribers | 18 by email, 13 on Telegram |

It's a small readership. Telegram posts get one or two views each. Email opens showed as zero, which turned out to be because Buttondown [turns open tracking off by default](https://buttondown.com/blog/opt-in-analytics) for new accounts. I'm only now starting to measure who actually reads it.

![Two-panel figure. Panel a shows October 2026 day by day: October 1 was not generated; October 2, 14, 15, 28, and 29 went out by Telegram and email but the blog page was missing; and from October 6 on, the watchdog sent a "missing" alert every day. Panel b compares the figures printed in early-October issues with the actual October 2 closes: KOSPI 8,000 versus 7,003.74, 1,530 versus 1,345.75 won per dollar, and a 4.49% versus 5.28% 10-year Treasury yield.](/assets/images/econ-radar-five-month-audit/reliability_en.png)

## Watchdog false alarms

In early October one daily issue simply didn't exist. The cloud routine hit a usage limit and exited after a second, yet the run was logged as a success. So I added a watchdog: every night at 10 p.m. it checks that the day's issue and blog page exist and sends me a private Telegram message if they don't.

From October 6 on, that watchdog warned me every single day. Every time I checked, the page was there. The GitHub Actions history showed why: the job scheduled for 10 p.m. actually started between 2:00 and 6:40 a.m. the next day. It checked the date at run time, so it was looking for tomorrow's issue, which didn't exist yet.

The worse part is what the noise hid. The blog pages really were missing on October 14, 15, 28, and 29, and the watchdog, busy checking the wrong day, missed all four. It now checks the date twelve hours before its run time, and I re-ran it on GitHub with the run time of one of the false alarms to confirm.

## A deploy script that reported success without publishing

For the days with missing blog pages, the cloud run logs ended like this:

```
✅ 커밋 완료            (commit done)
Everything up-to-date
🚀 공개 완료 (1~2분 뒤 라이브)   (published, live in 1–2 minutes)
```

The cloud session checks out the repository in detached HEAD state, pinned to a commit rather than a branch. In that state the script commits the new page and runs `git push origin main`, which pushes the old local `main` instead of the new commit. Git says "Everything up-to-date," the script exits cleanly and prints "published," and the session trusts that output and reports the issue as live, URL included. paper-radar, my weekly paper digest, uses the same script and was missing four weeks for the same reason.

The fix has two parts. The script now pushes the current commit with `git push origin HEAD:main`, and after pushing it checks that the page actually exists on the remote, failing if it doesn't. I first reproduced the false success with the old script in a detached-HEAD test, then used the fixed script to publish the four missing paper-radar weeks.

## July numbers in an October issue

The October 3 issue's investment item read "KOSPI around 8,000, won-dollar 1,530 to 1,560." The linked source was dated July 3. The actual October 2 closes were 7,003.74 and 1,345.75 won. The October 4 issue said the 10-year Treasury yield had "fallen to 4.49% after the jobs shock"; that day it actually rose, to 5.28%. And the October 9 issue's "10-year breaks 5.35%" was an intraday high written up as if it were a close (the highest close was 5.31%, on October 5).

The fact-checking agent was supposed to catch this, and the run logs explain why it didn't. The cloud session can't reach domains outside its allowlist, so the fact-check step could barely open any source articles. On one day all nine source domains failed to resolve, and the result still read "10 figures checked, 0 errors." A zero-error count means nothing if none of the sources could be opened.

## Checking market figures against real prices

So I built a check that doesn't depend on opening the source article. It pulls KOSPI, 10-year Treasury, and won-dollar figures out of each issue and compares them with the range of actual closes over the previous 30 days. It also flags any market-level item that cites an article more than three weeks old. Sentences that shouldn't be compared, such as brokerage index targets, other countries' bonds, and article titles in source lists, are skipped.

| Check | Result |
|---|---|
| Oct 3 and Oct 4 issues, before the fix | Flagged KOSPI 8,000, 1,530 won, and 4.49% |
| Same issues, after the fix | 0 flags |
| All September to November issues | 2 stale source links (real problems) |

The check runs once inside the cloud routine and again in GitHub Actions after the issue lands in the repository, this time with live market data, and it messages me privately if anything is flagged. Market data sites the cloud session can't reach are reachable from GitHub's runners. It's the same move I made in June, when I shifted Telegram delivery from the cloud session to GitHub Actions.

## Making the pages findable

The audit also showed that every daily page was titled with nothing but a date ("econ-radar — 2026-11-01"). There was no search snippet, no link preview, and no structured data for search engines and AI assistants to read, all things [I added to my blog posts last month](/en/2026/10/23/agent-cleanup-review-gate/).

- Titles now lead with the day's main story ("DNA 치료백신의 첫 FDA 허가 \| econ-radar 2026-11-01").
- New pages and the 161 existing ones now carry a description, Open Graph preview tags, and NewsArticle structured data.
- Each daily page now has an email signup button next to the Telegram one.
- Topic notes that had split over spacing and hyphens (five for ADC, six for GLP-1, and so on) were merged into 13 canonical notes, which now feed the topic hubs below.

I haven't measured any effect yet. Page analytics went in today, so I'll compare in a few weeks.

## What I learned

- **A success message is not a result.** On October 28 and 29, the script's "published" and the session's "issue is live" were both wrong. Something has to check that the file is on the remote and the page actually loads.
- **Watchdogs can be wrong too.** Alerts that fire every day get ignored, and the real failures got buried under them. Once, at least, check by hand what your watchdog is actually looking at.
- **Check whether the verifier can open its sources at all.** If it can't, it can't compare, and an unchecked figure shouldn't count as "no error." Anything a machine can fetch, like market prices, is better compared by machine from somewhere with network access.

## Subscribe

econ-radar is a Korean-language economy newsletter with a heavy tilt toward pharma, biotech, and AI. Each day it picks five stories from trusted Korean and international business press and reads them through three lenses: industry structure, investment, and the market (rising companies and demand). Foreign articles are translated into Korean. It doesn't recommend buying or selling; it gives you information and scenarios to decide with.

- **Email:** [buttondown.com/kakyungkim](https://buttondown.com/kakyungkim), every evening after 6:30 p.m. KST, with a weekly trend report on Monday mornings
- **Telegram:** [t.me/econradar](https://t.me/econradar)
- **Archive:** [kakyungkim.github.io/econ-radar](https://kakyungkim.github.io/econ-radar/)

When I find mistakes like these, I fix them and keep a record. If a number looks off to you, please let me know.

## Next

I pulled 3,603 items out of the daily issues into a table and used it to open 12 [topic hubs](/econ-radar/topics/) that follow SK hynix, HBM, ADCs, GLP-1, and more over time, for readers who come looking for a topic rather than a date (the hubs are in Korean). Next I want to tag each item with company and event type (deal, approval, trial readout, earnings) so the table can feed a book and papers. paper-radar has the same root problem, since the cloud session can't open preprint abstracts either, so I've started moving its candidate collection to GitHub Actions.
