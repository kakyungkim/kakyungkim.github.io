---
layout: single
title: "A news harness built in a day, then grown over 20 days of running it every day"
date: 2026-07-01 09:00:00 +0900
lang: en
ref: econ-radar-20-day-evolution
categories: [AI & Tooling]
tags: [Claude Code, AI Agents, Newsletter, Building in Public, Automation, Agent Design, Harness]
permalink: /en/2026/07/01/econ-radar-20-day-evolution/
excerpt: "The skeleton took a day. The interesting part came after: 20 days where the frictions and failures of running it kept reshaping the design."
---

{% include lang-switch.html %}

Summarizing economic news at a consistent quality every day is surprisingly draining. You're careful for the first few days, then fatigue sets in and the depth wobbles with your mood. So I handed the job to a small team that runs every day instead of doing it myself. econ-radar is the personal knowledge harness that started that way. Every evening it organizes economic news through three lenses—industry structure, investment, and market—publishes automatically, and accumulates the record by topic so it can grow into trend reports, blog posts, and eventually a book.

Earlier posts in this series each covered one problem at a time: [dropping the approval gate and moving to fully automated publishing]({{ site.baseurl }}/en/2026/06/12/econ-radar-auto-publish/), and [designing for freshness and dedup so the agent stops repeating the same news]({{ site.baseurl }}/en/2026/06/19/agent-staleness-dedup/). This one steps back and walks through the whole 20 days in one breath.

The skeleton itself went up on June 9, 2026, in a single day. The interesting part is what came next. Over 20 days of running it almost daily, the frictions and failures of operation kept reshaping the structure. The first issue was shallow and read faintly like it was written by AI. This is the arc from that start to where it is now: every Monday morning a weekly digest goes out automatically across three channels. If you've ever actually operated any automation, you've probably hesitated at the same spots.

<figure style="margin:2em 0;text-align:center">
  <a href="{{ site.baseurl }}/assets/decks/econ-radar-harness/" target="_blank" rel="noopener">
    <img src="{{ site.baseurl }}/assets/images/econ-radar/deck-cover.png" alt="Cover of the econ-radar harness slide deck" style="width:100%;max-width:720px;border-radius:12px;box-shadow:0 10px 34px rgba(49,46,129,0.20)">
  </a>
  <figcaption style="font-size:0.9em;color:#64748b;margin-top:0.7em">
    The slide deck I used to present this harness (June 27). <a href="{{ site.baseurl }}/assets/decks/econ-radar-harness/" target="_blank" rel="noopener">View full slides →</a> · <a href="{{ site.baseurl }}/assets/decks/econ-radar-harness-2026-06-27.pdf" target="_blank" rel="noopener">Download PDF</a>
  </figcaption>
</figure>

## Start: skeleton in a day, but the first issue was shallow

What I set up on day one was three layers. Layer 1 is the daily pipeline: it collects and analyzes news, edits it into a single newsletter, then renders and ships it. Layer 2 is an Obsidian vault that accumulates each day's record into topic notes (MOCs) via tags and links. Layer 3 is authoring: it extracts patterns from the accumulated record to grow trend reports, blog posts, and a book. I fixed the operating rules early: prefer trustworthy business press as sources, translate foreign articles into Korean, and treat investment as information and scenarios rather than buy/sell advice.

<figure style="margin:2em 0;text-align:center">
  <img src="{{ site.baseurl }}/assets/images/econ-radar/deck-3layers.png" alt="econ-radar three-layer structure: daily pipeline, vault accumulation, authoring" style="width:100%;max-width:820px;border-radius:10px;border:1px solid #e2e8f0">
  <figcaption style="font-size:0.9em;color:#64748b;margin-top:0.6em">The three layers. Layer 1 runs daily, Layer 2 connects each day's record by topic, and Layer 3 grows the accumulated record into trend reports and blog posts.</figcaption>
</figure>

The first run worked well enough. 21 collected items passed through two analysts, became a newsletter, produced HTML and a push message, and updated five topic notes. The problem was the output. The first piece of feedback was that it lacked "the density of skimming a business daily." There were only three headline items, so it had no breadth, and it just listed events—it read like a news list rather than an analytical brief. The investment section had only conclusions, no two-sided bull/bear view. The prose, too, had a translated feel with AI cliches mixed in.

## Week 1: turning vague complaints into rules on disk

Here the first pattern that ran through the whole harness appeared. Vague complaints like "the density is weak" or "this reads like AI" got transcribed, each time, into rules and gates written to disk.

For the weak-density complaint, I pinned concrete criteria into the collection and editing steps: target 4–6 items per section, attach a one-line "why it matters" to every headline item, and make the investment section write both bull and bear. For the "reads like AI" complaint, I set up a baseline file of samples pulled from real business articles, and attached a review agent (style-critic) that looks only at prose right before publishing. That's the seat that catches cliches like "not A but B" or "in a sense."

The third lens also switched twice around this time. It started as careers (the future of work), but fresh material for daily accumulation was scarce, so I changed it to "companies to watch." Soon after, my job-study group gave feedback: "the output leans toward the investor's side; look from the demand side too." The critique was that the three lenses essentially only asked "who makes money" and left out "who buys and uses it, and why." So I layered a Demand lens on top, binding companies to watch (the sell side) and demand (the buy side) into one lens. I even wrote into the baseline that in pharma/bio, demand splits three ways—patients, prescribers, and payers.

Keeping all these baselines in files rather than in someone's head is the point. The prose samples, the depth checklist, the demand-lens breakdown—all of it lives in memo files in the vault. It's a device to make the agents look at the same yardstick every time.

## Auto-publish: from "nothing goes out until I check it" to "it goes out, and I pull it if it's wrong"

On June 12, just a few days in, I made a big decision. I dropped the human-approval gate and switched to fully automated publishing. The policy moved from "don't send until confirmed" to "send, and cancel if something's off." Every evening at a set time, it now builds the issue, posts to the blog, and notifies via Telegram—hands off. (I covered the reasoning and the safeguards behind this switch separately in [an earlier post]({{ site.baseurl }}/en/2026/06/12/econ-radar-auto-publish/).)

But then safety had to come from somewhere else. On June 13, at issue #4, I critiqued the whole harness and applied four priorities at once. For the problem that the gate only checked prose while wrong numbers slipped through, I added a fact-checking gate (fact-checker): it reconciles up to ten key figures against the source and blocks any figure without a source before publishing. Going into auto-publish, that was the defense line I needed most. For the empty Layer 3, I scheduled a cloud routine to auto-generate the weekly digest; for the problem that the cloud can't read the past, I tracked the entire vault in a private repo so it can connect multiple weeks. I found the render step was the most expensive at about 93K tokens a run, yet it barely needed judgment—so I swapped it for a deterministic script and cut the per-issue token cost.

<figure style="margin:2em 0;text-align:center">
  <img src="{{ site.baseurl }}/assets/images/econ-radar/deck-cost.png" alt="Per-issue token cost by step; the most expensive render step was cut to near zero with a script" style="width:100%;max-width:820px;border-radius:10px;border:1px solid #e2e8f0">
  <figcaption style="font-size:0.9em;color:#64748b;margin-top:0.6em">Per-issue tokens by step. The most expensive step—render—needed the least judgment. Freezing it into a script removed that chunk.</figcaption>
</figure>

That last decision sharpens the second pattern. Steps that need no judgment don't go to the LLM; they get frozen into scripts, and only the steps that judge stay with the model. The fact that the most expensive step needed the least judgment was a finding worth keeping.

## Automation breaks on its first real run: rehearse before you ship

After moving to auto-publish, what I learned most is that "send it automatically" almost never works on the first try.

Telegram notifications stopped arriving after auto-publish. The workflow finished "successful" while the send step alone was silently skipped. The cause was that the send-or-not logic compared only the last single commit. When one push bundles several commits—the auto-generated one and the curation one—the file carrying the send signal sat in an earlier commit and got missed by the detection. Only after widening the comparison to the whole push did sends go through normally.

Adding the email channel produced an even sharper case. On the evening of June 25, the first immediate send was rejected with HTTP 400. Telegram was fine; only email failed. It turned out the send service requires a specific protective header for immediate sends only, and since I'd tested only drafts and scheduled sends, that requirement never surfaced. The safety catches on an external send API reveal themselves only on a real immediate send, not on drafts or schedules. Automation, as usual, breaks on its "first real run."

<figure style="margin:2em 0;text-align:center">
  <img src="{{ site.baseurl }}/assets/images/econ-radar/deck-production-lessons.png" alt="Table of failures hit while adding channels, and the lessons" style="width:100%;max-width:860px;border-radius:10px;border:1px solid #e2e8f0">
  <figcaption style="font-size:0.9em;color:#64748b;margin-top:0.6em">The walls I hit while adding channels. Each channel is a different runtime, so the same output has to be re-verified in the delivery environment.</figcaption>
</figure>

So when I brought up the Layer 3 weekly publish, I ran the whole thing end-to-end in CI before sending. That's how I caught the Korean-filename problem in advance. I'd first named the weekly HTML with Korean, like `2026-W26-동향.html`; it opened fine locally, but on the live URL the Korean risked breaking during encoding. Only after stripping Korean from the filename did I confirm a healthy response live. The same check also caught a bug where an environment variable in a conditional branch wasn't visible to the next step, so the blog send would never have run. Without rehearsing before publishing, those are spots where the first auto-send would have quietly failed again.

One more lesson attaches here. "Fixed it, but live is unchanged" happened often—deploy-workflow lag compounded by browser and CDN caches. Working locally doesn't mean it works in the actual delivery environment. Each channel is a different runtime, so live has to be checked separately.

## Compare and complement rather than switch wholesale

The third pattern came from trying to rework the collection step. Since scraping news by web search every day burns a lot of tokens, my hypothesis was that switching to RSS collection would be faster and cheaper. Instead of drifting toward whatever looked better, I built the same day's issue three ways, end to end, and compared. Existing web search (A), RSS only (B), and web search plus RSS as a union (C).

RSS-only (B) collected in about a second with clean dedup, but that day's big bio news was 0 of 279 candidates. Key trade-press sources were blocked, leaving a coverage hole, and since RSS summaries were thin I ended up re-fetching full text anyway—so the token savings I'd expected didn't materialize. By contrast, C—web search complemented by RSS—had the widest coverage: web search caught the big news while RSS consistently reinforced domestic and trade-press items. Comparing the three issues on real data, the clear call was to complement web search with RSS rather than replace it wholesale.

"Complement after comparing rather than switch" is the same grain as adding the demand lens as one more layer on the existing analysis, running it for a few days, instead of building a new agent. Before ripping things out, put them side by side and judge by effect.

## Knowledge in a standard format: one folder, read three ways at once

Layer 2, the asset side, also got one cleanup. On June 24, I moved the vault from Obsidian wikilinks to an open standard called OKF (Open Knowledge Format). It's Google Cloud's write-up of the pattern of stacking knowledge in markdown and links for an LLM to read; the key is that a file path becomes a concept's identifier and links are written as absolute-path markdown. The only thing I changed was the link syntax, but as a result one vault folder serves three uses without conflict. In Obsidian you navigate by graph and backlinks; an agent just `cat`s and queries it; and with git you version and deploy it. I moved 1,228 wikilinks at once to satisfy all three.

There's still an unsolved chore here, though. The migration is done, but the conventions the cloud publishing routine references still point at the old format, so it builds in the old shape every day and reverts the migration. Conventions and baselines have to live inside the repo, not globally, to stick—a lesson I confirmed repeatedly as I added channels. In fact I moved the prose rules into the repo for the same reason, keeping a copy the review agent is required to read.

## The last blank: Monday morning's weekly digest

The seat that stayed empty longest was Layer 3's public publishing. The routine to auto-generate the weekly digest had been scheduled since June 13, but with no way to ship it, three weeks—W24 through W26—had piled up on disk without ever being published. The daily went out on three channels every day while the weekly didn't go out at all.

This week I opened that path. I wrote a new script to render the weekly digest as HTML, attached the same three-channel send path as the daily (blog, Telegram, email), then scheduled a workflow to send it Monday morning at 8. The cadence: daily every evening, weekly Monday morning. Framed as "recap last week plus what to watch this week," it aims to be read once as the week opens. The Korean-filename bug above was caught precisely while pre-verifying this publish.

The agent team also broadened by layer around this time. On top of the eight Layer-1 members that run daily, I added four specialists called only when there's deeper work: a designer for the output's visuals and responsive UI, a librarian who periodically tidies the Obsidian graph and guards link integrity, a reviewer who checks prose, facts, and structure in one pass before a blog post or book ships, and someone who repurposes trends and posts into short social pieces. The daily crew stays light; specialists get called only for deep work. Day one's design—see the work not as one person's single prompt but as a team with divided roles—differentiated one step further here.

<figure style="margin:2em 0;text-align:center">
  <img src="{{ site.baseurl }}/assets/images/econ-radar/deck-team.png" alt="The eight-agent Layer-1 pipeline and four on-demand specialists by layer" style="width:100%;max-width:860px;border-radius:10px;border:1px solid #e2e8f0">
  <figcaption style="font-size:0.9em;color:#64748b;margin-top:0.6em">The eight-agent Layer-1 pipeline with two gates (prose and facts), plus four by-layer specialists called only when needed.</figcaption>
</figure>

## What ran through the 20 days

I started out to fill an empty Layer 3, but what actually got filled wasn't a single publishing path. It was the verification and rules that run that path safely, and the organization that divides the work. Looking back over 20 days, the structure changed almost every day, and every change started from a friction that surfaced while operating it. Turn vague complaints into rules on disk as they arise; when automation doesn't work in one shot, rehearse before sending; before switching wholesale, compare side by side and choose to complement; divide the work into a layered agent team. Four patterns repeated, differing only by date.

So automation is less about wiring up a Send button and more about finding, in advance, the spots that break on the first real run. A harness doesn't end at "done"—it keeps a backlog turning. The points you critique in yourself become the next things to do, in a structure that doesn't stop.

---

*You can also get econ-radar's evening economic-news digest on the Telegram channel [@econradar](https://t.me/econradar) or by [email newsletter](https://buttondown.com/kakyungkim). The full deck is available as [slides]({{ site.baseurl }}/assets/decks/econ-radar-harness/) and [PDF]({{ site.baseurl }}/assets/decks/econ-radar-harness-2026-06-27.pdf).*
