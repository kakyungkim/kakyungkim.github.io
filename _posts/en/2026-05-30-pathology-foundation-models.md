---
published: false  # 보관 중 (킵) — 게시 시 true로
layout: single
title: "What Pathology Foundation Models See in an H&E Slide"
date: 2026-05-30 10:00:00 +0900
lang: en
ref: pathology-foundation-models
categories: [Computational Pathology, Data Science]
tags: [Foundation Models, Digital Pathology, H&E, Deep Learning, Biomarkers]
permalink: /en/2026/05/30/pathology-foundation-models/
excerpt: "Routine H&E slides are pink-and-purple snapshots of tissue. A new generation of foundation models suggests they also carry a surprising amount of molecular signal."
---

{% include lang-switch.html %}

For more than a century, the H&E stain has been the workhorse of pathology. Hematoxylin paints nuclei blue-purple, eosin washes the cytoplasm and stroma pink, and a trained pathologist reads the resulting architecture to diagnose disease. It is cheap, fast, and available for almost every tumor that has ever been biopsied.

The interesting question for a data scientist is this: **how much of a tumor's molecular state is already written into that pink-and-purple image?** A growing body of work — and a new generation of "foundation models" for pathology — suggests the answer is *more than you might expect*.

## Morphology is a readout of molecules

The link is not magic. Genotype shapes phenotype, and phenotype shapes how tissue looks under a microscope. Tumors with defective mismatch repair (dMMR / microsatellite instability) tend to show characteristic features — tumor-infiltrating lymphocytes, mucinous or medullary patterns. Certain driver alterations associate with recognizable growth patterns. Pathologists have used these visual cues for decades; the difference now is that we can ask a model to learn them at scale, from far more examples than any single expert will ever see.

This is why predicting molecular labels from H&E is plausible at all: the morphology is a noisy, indirect — but real — readout of the underlying biology.

## What a "foundation model" changes

Until recently, most computational pathology models were trained from scratch for one narrow task, on a few hundred or few thousand annotated slides. They worked, but they were brittle and data-hungry.

Foundation models flip the recipe. They are trained **self-supervised** on enormous collections of unlabeled tissue tiles — millions of small patches cut from whole-slide images — to learn a general-purpose visual representation. Publicly described examples include **UNI** and **CONCH** from the Mahmood Lab and **Virchow** from Paige. CONCH adds a language component, aligning image tiles with text so the model learns from captions as well as pixels.

The payoff is transfer. Once a model has a strong general representation of tissue, a downstream task needs far less labeled data: you freeze the encoder, turn each tile into an embedding, and train a small head on top.

## From a giant image to a single prediction

A whole-slide image is enormous — often billions of pixels — so you cannot feed it to a network whole. The typical pipeline looks like this:

1. **Tile** the slide into thousands of small patches (e.g. 256×256 pixels at 20× magnification), discarding background.
2. **Embed** each tile with the foundation model into a fixed-length vector.
3. **Aggregate** the tile embeddings into one slide-level prediction, usually with *multiple-instance learning* (MIL) or an attention mechanism that learns which regions matter.

The attention step is more than a technical detail: it lets you see *where* the model looked, which is essential for trusting — or debunking — a prediction.

## What they can and cannot do

It is worth being precise here, because the field attracts hype. H&E-based models have shown genuinely useful signal for some targets — microsatellite instability, a handful of recurrent mutations, and molecular subtypes in several cancers. For other targets the signal is weak or absent, and reported performance can drop sharply when a model trained at one hospital meets slides scanned, stained, and prepared at another.

So the honest framing is not "H&E replaces molecular testing." It is closer to **triage and prioritization**: a fast, nearly free pre-screen that can flag which cases deserve a confirmatory assay, or surface hypotheses worth testing — always validated against ground-truth molecular data, never asserted on image evidence alone.

## Why this is exciting anyway

Every biopsy already produces an H&E slide. If even part of a tumor's molecular profile can be inferred from an image you already have, that is enormous leverage — especially where sequencing is slow, expensive, or simply unavailable. The research challenge that remains is the unglamorous one: **generalization**. A model is only as good as its behavior on slides from sites it has never seen, with stains it was never trained on.

That is the part I find most interesting, and it is where careful data science — honest baselines, leakage checks, external validation — matters far more than the size of the model. More on that in future posts.
