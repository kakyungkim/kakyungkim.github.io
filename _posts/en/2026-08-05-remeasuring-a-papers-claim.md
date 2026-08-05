---
layout: single
title: "I Re-measured a Paper's Own Number"
date: 2026-08-05 09:00:00 +0900
lang: en
ref: remeasuring-a-papers-claim
categories: [AI & Tooling]
tags: [Claude Code, AI Agents, Literature Survey, Research Automation, GPTers]
permalink: /en/2026/08/05/remeasuring-a-papers-claim/
excerpt: "Can an LLM stand in for an expert judge? I tested it on public data, missed the target, and found the reason wasn't the model."
---

{% include lang-switch.html %}

I'm in a GPTers study group on reviewing AI output. Here's what I did in week one: I borrowed someone else's tool wholesale, and used the time it saved to re-measure a number another paper had published.

## The problem

Since 2025 there's been a flood of "AI does science on its own" papers. Co-Scientist, The AI Scientist, AI-Researcher. From the titles they look interchangeable. Open them and they do quite different things: one generates hypotheses, one writes code, one drafts manuscripts.

I kept reading them and kept forgetting them. Finish one, start the next, and the first one blurs.

One sentence in particular stuck with me: nine domain experts had evaluated the AI-generated hypotheses. So what happens if you put an LLM in one of those nine chairs? That's a question public data can answer.

## Not starting from scratch

The backbone here isn't mine. I took `research-survey`, a Claude Code plugin published by the study lead, and built on it.

It runs five stages:

```
topic → extract papers → summarize → triple-verify → insights/hypotheses → keep tracking
```

The third stage is the interesting one. **Triple-verify** checks the summary against the source three different ways. The most common failure when an LLM summarizes a paper is inventing something plausible that isn't in the text, and that stage is built to block it.

Building this myself would have meant days on the fetcher, the classifier, and the evidence-tracking rules. Borrowing it freed all of that, and I spent the freed time **testing the paper's claim**.

## What I did

Four steps.

**One, build the corpus.** Pull related work from arXiv, sort into two categories. Each refresh added more; 25 became 57.

**Two, turn the top seven into notes.** One rule: nothing goes in without a source. An LLM summary reads smoothly, which is exactly the problem — smooth prose is hard to audit. So every claim had to carry a page or figure number.

This is where the baseline came from. The paper's appendix records nine board-certified hematologists and oncologists **split into two groups**, scoring 78 hypotheses, with **agreement between the two groups at Spearman ρ = 0.745** (p < 0.001).

Worth unpacking. At 1 the two groups ranked everything identically; at 0 they were unrelated. So 0.745 is strong but imperfect. **The key point is that this is a ceiling.** If humans only agree with each other at 0.745, an LLM in the same seat can't meaningfully exceed it.

One caveat worth stating: this is agreement *between groups*, not between individuals. And the paper does not say **how the nine raters were split, nor how scores were combined within a group** — not the sizes, not the basis for the split, not whether both groups rated all 78 hypotheses. That matters. Averaging within a group cancels individual noise and pushes the figure up; consensus pushes it up further; splitting the 78 between groups would mean the correlation was computed on an overlap of unknown size. **Whether 0.745 can serve as a ceiling depends on which of these it was.** I use it as a reference point, not as a settled ceiling.

**Three, put the LLM in the judge's chair.** It scored hypotheses on five criteria: novelty, feasibility, specificity, non-triviality, grounding. The subjects were eight drugs evaluated for repurposing in acute myeloid leukemia, and I compared the scores against the wet-lab results in the paper.

| Comparison | n | Spearman ρ | p |
|---|---|---|---|
| LLM scores vs wet-lab (binary) | 8 | 0.447 | 0.267 |
| LLM scores vs wet-lab (continuous) | 11 | 0.642 | 0.033 |
| Between the two expert groups | 78 | 0.745 | < 0.001 |

I missed my 0.70 target. But 0.642 is **86% of the human ceiling**.

**Four, check the gap the paper claimed.** The same paper argued that public databases hold almost no negative results. I searched PLOS and PubMed six different ways.

| Source | Query | Hits |
|---|---|---|
| PLOS | `drug repurposing` + `negative result` | 248 (raw) |
| PLOS | strict filter (repurposing proper) | ~30 |
| PubMed | `"Negative Results"[pt] AND "Drug Repositioning"[MeSH]` | **0** |
| PubMed | `"drug repurposing"[tiab] AND (negative result*)` | 68 |

Together that's roughly **50 to 60** strict negative-result papers, which just clears the target. It cleared, but not easily. It took six separate queries, and **the standard classification field returned zero**. The publication type "Negative Results" exists in PubMed, and almost nobody tags anything with it. The collection process itself demonstrated the gap the paper described.

## What I learned

**One: of five criteria, one was doing the work.**

I examined the rubric itself to see which dimensions actually tracked the experimental outcome.

| Criterion | r_pb | p | Reading |
|---|---|---|---|
| **Grounding** | **0.707** | **0.050** | the only significant one |
| Specificity | 0.500 | 0.207 | right direction |
| Non-triviality | 0.302 | 0.468 | right direction |
| Novelty | **0.000** | 1.000 | **no discrimination at all** |
| Feasibility | **-0.146** | 0.730 | **runs backwards** |

Hypotheses with solid literature support also worked in the lab. Feasibility ran backwards: drugs withdrawn from market score low on feasibility, but withdrawal is often about side effects or economics rather than efficacy, so they can still work in a dish.

**I split the scorecard into five columns and one of them decided the verdict.** The other four create a sense of thoroughness without explaining anything.

**Two: reading a low number takes care.**

Read the table alone and it's simple. 0.642 missed the target, so LLMs can't replace experts yet.

Read the design and it changes. In the original paper, the experts **pre-filtered the obviously bad hypotheses** before scoring. Everything left was already decent.

Put it in grading terms. Score a whole class and first and last are far apart, so two graders agree easily. Score only the top ten and the gaps are small, so the rankings wobble. Not because the graders are bad, but because the range narrowed. Statisticians call this range restriction.

**A low measurement and a low capability are not the same thing.** Miss that distinction and you take the number as the conclusion.

**Three: borrowing a tool buys time for the question.**

Had I built the survey pipeline myself, I'd never have reached the experiment. Building tools and asking questions with tools are different jobs, and this time the second one was the point.

## Where this started

When I signed up, my question was how to trust what an AI produces. After week one I had a second one. **If I've learned how to re-measure someone else's number, shouldn't I run the same check on my own?**

## Next

- I plan to ask the original authors for the full appendix data. With all 78 expert scores I could rerun the experiment properly instead of on a sample of eight. The email is drafted but not yet sent.
- I plan to run three different models as independent judges on a fixed rubric and measure how much they agree, to check whether one model's quirks are driving the result.

[The next post](/en/2026/08/06/checking-my-own-manuscript/) covers what happened when I ran this on **my own manuscript**. In a paper about building a system that refuses to fabricate evidence, I found 27 numbers whose evidence I couldn't trace.

---

**References**

- Gottweis et al. *Accelerating scientific discovery with Co-Scientist*, arXiv:2502.18864
- Bisht et al. *Agentic AI Scientists Are Not Built For Autonomous Scientific Discovery*, arXiv:2605.08956
