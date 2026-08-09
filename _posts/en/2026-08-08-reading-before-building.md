---
layout: single
title: "I Read Someone Else's Code Before Writing My Own"
date: 2026-08-08 09:00:00 +0900
lang: en
ref: reading-before-building
categories: [AI & Tooling]
tags: [Claude Code, AI Agents, Open Source, Design Decisions, PRD, Building in Public]
permalink: /en/2026/08/08/reading-before-building/
excerpt: "I read two open-source form builders to ground the design of a small MVP. Then I had the spec reviewed by someone other than its author, and four blocking defects came back."
---

{% include lang-switch.html %}

I set out to build something like Google Forms. You make a form, share a link, collect answers and file attachments. That's the whole scope. The target was to finish it inside a six-hour workshop, so the feature list had to shrink hard.

The usual move here is to scaffold a project and start drawing screens. I didn't do that. I read two open-source products that have been solving this exact problem for years.

## The problem

Building anything means a steady stream of decisions. Do I split the question list into its own table or store it as one JSON blob? Do I persist whether a form is closed, or compute it at read time? When do I validate an uploaded file?

Decide each of these by instinct and you can't explain them later. And a decision you can't explain is a decision you can't revise with confidence.

Products that already have users have made every one of those calls. The hard-to-reverse ones leave marks in the code. Reading those marks lets you borrow tuition someone else already paid.

## What I read

I picked two. Both are form builders, both are open source, both still get commits.

| | OpnForm | HeyForm |
|---|---|---|
| Backend | PHP, Laravel, REST | Node.js, NestJS, GraphQL |
| Frontend | One Nuxt app | React admin app + separate renderer package |
| Database | Relational, 112 migrations | MongoDB, 26 Mongoose schemas |
| Repo layout | Split into api and client | pnpm workspace, 7 packages |

Different languages, different databases, different API styles. That's what made the comparison worth doing. **Where the two diverge, it's usually taste or context. Where the two converge, you're probably looking at the answer to the problem.**

## How I read them

I wrote down twelve angles first, then went through one angle across both repos before moving to the next. Author flow, respondent flow, editor structure, renderer structure, question type definitions, data model, state handling, submission validation, file upload, auth and permissions, testing.

I deliberately avoided sweeping one repo end to end and reconciling later. Do that and your comparison criteria drift while you read.

## Where they went opposite ways

The sharpest difference was the relationship between the editor and the respondent-facing screen.

OpnForm reuses a single renderer and switches behavior by mode. There are eight modes, and each one returns an object saying whether to validate, whether to show admin controls, whether to actually submit. The editor preview mounts **the same component** as the public form, in preview mode.

HeyForm goes the other way. The builder canvas has 25 per-type components, and the respondent renderer has 28 more. The switch that maps a question type to a component exists twice, once on each side. Files like `FakeRadio` and `FakeSelect`, which only mimic an input, make the split obvious.

It looked like duplication until I found the reason. The renderer ships as its own npm package. To embed a form on someone else's site you have to break its coupling with the admin app. They bought deployment independence and paid for it in maintenance.

A six-hour MVP has no embed requirement, so I took OpnForm's side.

Six of these forks came up: whether form state lives in one column or scattered across a settings object, whether drafts and published content are stored separately, when files get validated, whether a file belongs to a response or to a form.

## Where they agreed

This was the real payoff.

**Neither normalizes the question list.** OpnForm puts it in a JSON column, HeyForm in an array of objects. Two opposite database families, same call. Reordering questions, changing types, and adding choices are frequent operations, and normalizing means touching many rows every time.

**Neither trusts the question definition that arrives with the request.** A client can submit answers alongside a definition claiming "this question is optional and its max length is 100,000." Both products discard that and re-fetch the form by id to get their validation basis.

**The stored definition decides what a value means.** I saw this in HeyForm's payment path. It ignores the amount and currency the respondent sent and recomputes them from the published form settings. There's even a comment explaining why.

**Closed-ness is computed, not stored.** Whether the state is closed, whether the deadline has passed, whether the submission cap has been reached: all evaluated at read time.

**File type is verified by content, not by extension.** HeyForm pairs each MIME type with an allowed extension list and a magic-byte check. OpnForm reads the leading bytes to detect the real type, and rather than blocking SVG outright it strips the scripts out.

## The one line that paid off most

OpnForm's authorization policy class has a method that reads roughly like this:

> Can this form be answered? Only if it isn't closed, hasn't hit its submission cap, and is publicly visible.

It accepts a nullable user. The entire treatment of anonymous respondents sits in that one line. And it translates almost directly into a Supabase row-level security policy: a single SQL condition saying inserts are allowed only when the form is published.

Scatter permission checks through application code and you will forget one while adding a new route. Put it in the database and every route hits the same wall.

## Then I had the spec reviewed

I turned the reading into documents: two per-angle analyses, a comparison table, and an adoption-decisions file. The last one ends with **a table linking each decision to the source file it came from.** Six months from now, "why did we do it this way" needs somewhere to land.

Then I wrote the product requirements from those decisions, and **separated the author from the reviewer.** The reviewer only gets the original brief and the decisions file, and reads the requirements against them.

Four blocking defects came back. Two of them stung.

**Four technical constraints were missing entirely.** Next.js App Router and Turborepo were in the brief and appeared nowhere in the document. Neither did mobile responsiveness. The persona section said the respondent "often opens this on a phone," but that isn't a requirement.

**A security guarantee undercut itself.** One section stated flatly that response inserts are enforced by row-level security in the database. Another section, the API summary, assumed that same path uses a service role key. A service role key bypasses row-level security completely. Implemented as written, the policy would have been decorative.

The acceptance criterion said this: "verify that a direct insert bypassing row-level security is blocked for the same reason." Bypassing it, and being blocked by it. The sentence contradicts itself, and I couldn't see it while writing.

## What the second round turned up

I applied the fixes and sent it back to the same reviewer. This time **the reviewer caught something it had missed in round one.**

Files are uploaded before the response is submitted, and the submit request carries the storage path along with the answers. That path is also a client-controlled value. If the server takes it at face value, a respondent can attach a file that was uploaded to somebody else's form.

I had written "don't trust values that come with the request" all over the document, and then left one place where the principle wasn't applied. Knowing a principle and applying it exhaustively turned out to be two different things.

## What I took away

**Three things.**

First, when two products built on different stacks make the same call, that call is worth trusting. Reading only one, I couldn't have told its taste apart from the problem's answer.

Second, the author shouldn't be the reviewer. Authors read what they meant. If it's in your head, it reads as if it's on the page. I did not find those four missing constraints myself.

Third, writing down what you haven't decided beats deciding by default. The document currently carries twelve open items, each with a note on why it's still open. Decide them silently and you lose the ability to tell a decision from a guess.

One of them was a repo tooling choice where the brief and my own prior analysis pointed in opposite directions. Nothing in the documents justified picking a side, so I didn't. I asked instead.

## Next

On to the architecture. The first things to settle are four security boundaries: what privilege reads a public form, which key handles response submission, how the file path gets verified, and how long a signed link stays alive.

The reading time paid for itself. A fair number of these I would have missed on instinct alone.

---

*This analysis and the documents around it were done with AI agents in the loop. They helped most with reading and organizing the repositories, and with reviewing the resulting documents from a perspective other than the author's. Both reference repositories were read only and left unmodified.*
