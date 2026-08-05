---
layout: single
title: "I Checked My Own Manuscript With Its Own Rule"
date: 2026-08-06 09:00:00 +0900
lang: en
ref: checking-my-own-manuscript
categories: [AI & Tooling]
tags: [Claude Code, AI Agents, Reproducibility, Manuscript Review, Harness, GPTers]
permalink: /en/2026/08/06/checking-my-own-manuscript/
excerpt: "I reconciled all 120 numbers in my manuscript against committed results. For 27 of them I couldn't trace where the value came from."
---

{% include lang-switch.html %}

[In the previous post](/en/2026/08/05/remeasuring-a-papers-claim/) I re-measured a number from someone else's paper. That left me with a question: if I've learned how to re-measure someone else's number, shouldn't I run the same check on my own?

This is that story.

## What the paper is about

I'm writing a manuscript on using LLMs for somatic variant interpretation. You find a change in a gene in tumor tissue and decide whether it drives the cancer. The answer determines which drug a patient gets, so this runs in hospitals.

Here's the failure mode. Ask an LLM to draft a clinical report and the prose comes out fine. But a citation supporting a claim may point to a paper that doesn't exist. From the manuscript:

> In a clinical report, a fabricated reference is a patient-safety and medico-legal hazard, because the assertion it appears to support cannot be checked.

So the system's headline property isn't accuracy. It's **faithfulness**.

> A usable decision-support draft must instead be auditable line by line and **must abstain rather than fabricate.**

Abstention means declining to answer when the evidence is thin. Like leaving a question blank instead of guessing. Your score doesn't go up, but you don't submit a wrong answer either.

The system does this. Given 10 variants with no supporting evidence at all, it **abstained on all 10**.

## The manuscript follows the same rule

The prose does it too. There are sentences throughout that decline to overclaim.

| Sentence in the manuscript | What it concedes |
|---|---|
| `not claimed as a novel classifier` | The rule engine isn't new work |
| `remains uncontrolled` | An uncontrolled factor, stated as uncontrolled |
| `not powered for an accuracy claim` | Sample too small to claim accuracy |

These sentences cost you. Writing "this isn't novel" invites reviewers to score you down. They're there because a system built on refusing to overclaim is worthless if its author overclaims.

## Where it started: one line from a reviewer

The manuscript has review-harness runs on record. The harness reads the draft like a journal referee and returns points and scores.

The second run gave reproducibility a **perfect 5**. The justification:

> Offline-from-snapshots, 90 tests, scripts named per result; **spot-checked** headline numbers reconcile with committed JSONs.

The first half is good news. It runs offline from stored snapshots, there are 90 tests, and every result names the script that produced it.

The last clause is what caught me. **Spot-checked** — a few headline numbers were sampled and found to match.

And there was the picture. **Telling students to leave the blank blank, and never checking whether you did it on your own answer sheet.**

## What I built

A checker that reconciles every number in the manuscript against committed result files. Not a sample. All of them.

```
extract every number from the manuscript
        ↓
extract every number from the committed results
        ↓
look each manuscript number up in that pool
        ↓
report only the ones with no match
```

The comparison set is 20 committed files: 16 benchmark results and 4 cohort descriptors. Committed means the version is pinned, so the same file can be reopened later.

Two adjustments to keep it fair. Rounding counts as a match, so a source value of 0.8432 matches 0.843 in the text. And conventional constants like 0.05 are excluded, since a significance threshold has no business being in a results file.

## Results

```
Unique numbers in manuscript   120
  traceable                     93 (78%)
  no source found               27 (22%)
```

Those 27 appear **41 times** in the text.

## What the 27 are

Careful here. **It does not mean 27 numbers are wrong.** Opening them one by one, they fall into three groups.

| Group | Count | Examples |
|---|---|---|
| Performance metrics (derived) | 21 | 0.443, 0.671, 0.714 |
| Percentages computed in prose | 5 | 21.4, 25.0, 58.9 |
| Sample size | 1 | n = 56 |

The **21 metrics** are computed from raw data. Accuracy is correct-over-total, so of course it isn't sitting in a results file in that form. The real problem is elsewhere: **where that computation happened isn't recorded**, not in the manuscript and not in the repository.

The **5 percentages** bother me more. The value 21.4 appears five times, and you can't tell from the text what was divided by what. A ratio without a denominator can't be sized. 21 out of 100 and 3 out of 14 are both 21.4%, and they don't mean the same thing.

## A human referee and a machine landed on the same spot

The first review raised five must-fix points. Two of them were exactly here.

| What the referee found by reading | What the checker found by counting |
|---|---|
| Attach a confidence interval to every figure | 21 metrics carry no interval |
| State the direction of the provisional adjudication | 21.4 appears five times with no denominator |

A confidence interval says roughly where the true value sits. Written bare, 0.843 looks precise; with a small sample the honest range might be 0.76 to 0.93.

**One found it by reading, the other by counting.** Completely different methods, same conclusion.

## What I learned

**One: the abstention rule went into two places and skipped a third.**

| Where | When evidence is missing |
|---|---|
| The system's variant calls | it abstains |
| The manuscript's claims about itself | it concedes |
| **The manuscript's numbers** | **they just sit there, unmarked** |

Why did numbers get skipped? **Because numbers look sourced.** Three decimal places reads as precisely computed. You interrogate a sentence; you rarely interrogate a figure. I suspect that's also why a referee awarded full marks for reproducibility after checking a sample.

**A format that doesn't invite suspicion passes review.**

**Two: 22% is not an error rate. It's a traceability-failure rate.**

It doesn't mean 27 numbers are wrong. It means there's no path to confirm they're right. Blur those two and you get "my paper has a 22% error rate," which is false.

This is the trap in building a checker. The program distinguishes found from not-found. What "not found" *means* is not something the program decides. **The meaning of a checker's verdict has to be defined by whoever built it.**

**Three: a rule appears only where you felt you needed one.**

A fabricated citation becoming a patient-safety problem felt urgent. So that place got an abstention mechanism. The provenance of a number in my own paper never felt that urgent, so it got nothing. Writing a principle down doesn't make it travel to the next room.

## Where this leaves me

After two weeks, my original question has changed.

**Review isn't the machinery that keeps an AI from being wrong. It's the machinery that decides in advance what gets rejected automatically, what gets compared against a known answer, and what gets flagged as unknown.**

The third column is the one most often left empty, because marking something unknown makes your numbers look worse.

## Next

- Write the **computation scripts** for the 21 metrics. Right now there are values without a process.
- Attach **denominators** to the percentages: `21.4% (12/56)`, not `21.4%`.
- Add **abstention marks to the manuscript** itself. If a number's derivation isn't documented yet, say so.

The checker is committed to the repository so it can be rerun. Each time I revise, I can watch whether 27 goes down.

The paper is unpublished, so no scientific results appear in this post. What's here is the review process only.
