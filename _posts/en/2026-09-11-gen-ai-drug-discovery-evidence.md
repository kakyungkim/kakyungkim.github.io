---
layout: single
title: "How Far Generative AI Has Come in Drug Discovery, and How Far You Can Trust It"
date: 2026-09-11 09:00:00 +0900
lang: en
ref: gen-ai-drug-discovery-evidence
categories: [Life Science, Data Science]
tags: [Generative AI, Drug Discovery, Protein Design, BRIC View, Data Leakage, Reproducibility, Regulation]
permalink: /en/2026/09/11/gen-ai-drug-discovery-evidence/
excerpt: "I published a trend report on generative AI in biotech and drug discovery. One question runs through all of it: how far can you actually trust the output?"
---

{% include lang-switch.html %}

I published a report with BRIC View on generative AI in biotech and drug discovery. This post is
the plain-language version, written so you can follow it without a background in the field. One
question runs through the whole report: generative AI is said to be changing biology, but how far
can you trust what comes out of it?

The answer depends on where you look. Protein design has reached the point where results are
confirmed in the lab. Small molecules are standing at the threshold of the clinic. Genome-scale
design is still inside the computer. The rest of this post walks through that gap.

## What it takes to produce one drug

Bringing a new drug to market means screening tens of thousands of candidate compounds down to
one, then testing it in animals and then in people. That takes well over a decade and costs on
the order of a billion dollars. Even among candidates that reach first-in-human dosing, roughly
one in ten becomes an approved drug.

Hence an old question: can we pick the candidates faster, and pick them better?

## Where generative AI fits

The same class of model that writes sentences in ChatGPT also generates proteins and compounds.
Earlier AI was mostly asked to predict: will this molecule be toxic? Generative AI answers a
different request: draw me something new with these properties. The work moved from searching to
designing.

That takes material to learn from. UniProt holds over 227 million protein sequences. The
AlphaFold structure database holds over 214 million predicted structures. ChEMBL holds over 20.3
million bioactivity measurements. Public resources like these are the training material.

## Protein design has reached experimental confirmation

The clearest results have come from protein design. RFdiffusion designs, from scratch, proteins
that bind to a chosen target. These designs were not left on paper. They were expressed,
purified, and imaged, and the designed shape matched the measured structure to within 0.63 Å of
backbone RMSD. A single atom is roughly 1 Å across, so the error was smaller than one atom.

The success rate, though, was about 19%. Design five and one works. That is a large improvement
over earlier methods, and the 2024 Nobel Prize in Chemistry went to protein structure prediction
and computational protein design.

## The funnel narrows toward the back

Generating candidates inside a computer is cheap and plentiful. Passing through the lab and then
through clinical trials, the number that survives drops sharply while the evidence demanded of
each one gets stronger.

The numbers trace that shape. Among AI-discovered drugs, 21 of 24 advanced out of Phase I, which
mainly tests safety. That is about 88%, well above the industry average of 40 to 65%. In Phase
II, where the question becomes whether the drug actually works, only 4 of 10 advanced. At 40%,
that does not beat the industry average.

The most advanced case sits in the same place. Rentosertib, a TNIK inhibitor aimed at idiopathic
pulmonary fibrosis, met its primary safety endpoint in a Phase 2a trial of 71 patients and
entered Phase 3 in July 2026. Efficacy showed a favorable trend, but only as a secondary
endpoint, in arms of roughly 18 patients each. Confirmation is still ahead.

## How a scorecard gets inflated

Sometimes the scorecard itself deserves suspicion. If the practice problems show up on the exam,
the score looks good, and the same thing happens in data. Material used for training leaks into
the evaluation set.

This has been documented in at least 294 papers across 17 fields and sorted into eight distinct
types. With leakage, benchmark scores come out high and then fail to reproduce when someone
measures again elsewhere. I ran into the same problem a few days ago when I measured it myself on ADMET models, written
up [here in Korean](/kr/2026/09/08/admet-split-audit/).

## Risk grows with capability

As generative capability grows, so does the room for misuse. When researchers inverted the
objective of a drug discovery model, telling it to seek toxicity instead of avoiding it, roughly
40,000 toxic molecules were designed in under six hours. Nothing was synthesized or verified;
these are computational results.

There are moves in the other direction. Evo 2, a genome generative model, deliberately excluded
viral genomes that infect eukaryotes from its training data, blocking dangerous sequences at the
design stage rather than after the fact.

## Where the rules stand

Regulation is catching up. The US FDA issued a draft framework for judging whether AI-generated
data is credible enough to support regulatory decisions. The EU set obligations for high-risk AI
under the AI Act and pushed the main compliance dates to December 2027 and August 2028. The USPTO
concluded that AI cannot be named an inventor and that a significant human contribution is
required. In Korea, the MFDS issued a guideline for LLM-based digital medical devices in June
2026, listing seven hazard categories along with validation metrics.

What is still missing is a separate standard for how evidence produced by generative models
should be weighed in drug review.

## Where this started

Last spring I wrote a BRIC View report on how bio big data is being pooled and expanded. Once
that was done, the next question was sitting right there: what is actually being built on top of
that data?

Starting the survey, I found no shortage of material listing technologies and cases. What was
missing was a separation of how strong the evidence is at each stage. So instead of widening
coverage, I followed one question through.

## Read it

The full report is available at BRIC View, in Korean.

- [BRIC View 2026-T24, "Generative AI in Biotech and Drug Discovery: Technology, Cases, and Ecosystem Outlook"](https://www.ibric.org/s.do?JpKhtZtuZJ)

Every number in this post comes from the report, and the primary sources are in its reference
list.

## Next

The part of the report that took longest was separating which claims rest on what grade of
evidence. Wanting to do that with my own hands, I measured how the choice of data split changes
reported performance in ADMET models, and wrote that up separately. Next I want to widen the task
and model set and work out why some of my results diverged from prior work.

The sentence the report closes on is this. In the era of generative AI, competitiveness in
biotech will rest less on how fast you can generate something, and more on how credibly you can
demonstrate that what you generated holds up.
