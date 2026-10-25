---
layout: single
title: "Review Agents That Never Ran, and an Agent Cleanup"
date: 2026-10-23 00:30:00 +0900
lang: en
ref: agent-cleanup-review-gate
featured: true
categories: [AI & Tooling]
tags: [Claude Code, AI Agents, Subagents, Code Review, Plugins, Building in Public]
permalink: /en/2026/10/23/agent-cleanup-review-gate/
faq:
  - q: "Where and how should I attach a review agent in Claude Code?"
    a: "You can call it as a subagent in the same session, keep a separate review-only workspace, or attach reviewers to a multi-agent tool or the Agent SDK. Creating one per purpose tends to pile up similar reviewers that rarely get called; fixing the order of calls in the structure is what led to real calls."
  - q: "What kinds of errors do review agents miss in real work?"
    a: "Errors planted in a sentence show up when compared against the source, but errors from the calculation process and processing order, such as applying a log transform twice or comparing a transformed value against the raw file, are hard to see from the result sentences alone."
  - q: "What should I watch for when using review agents in a specialized field?"
    a: "In fields with little public data, LLMs miss more logic errors, so build a domain expert's review criteria into the harness and keep the reviewer apart from the writing context."
excerpt: "Every time I wrote a review agent, I assumed review was now covered. Counting from session logs, only 16 of 97 agent definitions had ever been called. Based on that record, I cut the always-on agents from 32 to 13 and made review verdicts get written to a file."
---

{% include lang-switch.html %}

I'm Ka-Kyung Kim, a bioinformatics researcher working across clinical data, genomics, and machine learning.

When I counted my Claude Code session logs, only 16 of 97 agent definitions had ever been called, and 11 of 14 review-role definitions could edit files. Based on that, I cut the always-on global agents from 32 to 13, made the review agents read-only, and had review verdicts written to a file that a gate script checks.

After a few months of working with Claude Code, my agent definitions had piled up. I wrote a paper-critique agent for manuscripts, a fact-checking agent for reports, and a style-checking agent for blog posts. Each time I added a review agent, I told myself that review was now in place.

A lecture in a study group in GPTers (a Korean AI practitioners' community), cohort 24, argued for separating production from review, and I wanted to check whether my setup actually did that. Over three weeks I used session logs to measure whether the review agents were really called, what permissions they held, and whether their verdicts led to edits. The measurement covered every local session log on this Mac as of October 22, 2026 (the earliest dated August 9); cloud-scheduled runs leave no local log and are not included. For how subagents and plugins behave, I relied on the official Claude Code docs on [subagents](https://code.claude.com/docs/en/sub-agents) and [plugins](https://code.claude.com/docs/en/plugins). This post covers those measurements and how I reorganized my agent setup based on them. It builds on the case I presented in the study group, and I've included the commands and the rollback procedure so others can follow along.

## Claude Code Agent Definitions: 16 of 97 Ever Called

![Three bar charts. Of 97 agent definitions, 16 were ever called; of 14 review-role definitions, 11 had Edit or Write access; always-on global agents went from 32 to 13 after the cleanup](/assets/images/agent-cleanup-review-gate/audit_en.png)

First I found every `.claude/agents/*.md` under my home folder, grouped them by name, and counted Agent tool calls in the local session logs.

| Item | Value |
|---|---|
| Agent definitions | 97 unique names (160 files) |
| Global definitions loaded into every session | 32 |
| Agent calls | 526 |
| Of those, generic agents called without a definition | 404 (77%) |
| Definitions called at least once | 16 of 97 |
| Calls to review-role definitions | 11 |

There are more files than names because the same definition sits in several backup and copy folders. Agents for projects that run daily on cloud schedules, such as the economics briefing and the paper briefing, leave nothing in local logs and showed up as 0 calls. Reading that as "unused" would be wrong.

Even so, most review definitions had no call record at all. Similar reviewers kept appearing project by project, and once created, they were almost never called.

## What Called Review Agents Had in Common

Some places did call review steadily. The multi-agent tool I learned about in the study group creates a team (the tool calls it a department) for a job, attaches a reviewer alongside the author automatically, and injects role guidance at each step. Of the 54 times the tool injected role guidance, 29 were steps that call the reviewer. In a project where I defined a writing agent and a checking agent as a pair in one folder from the start, the checker followed the writer 6 times out of 11 writing runs.

The two records count different units, so I didn't compare them directly. But the places where reviewers got called shared one trait: the order of calls was fixed by structure. When I wrote "call review if needed," I was the one who judged the need, and I was the one who forgot.

## Write Access on Review Agents

The lecture recommended an arrangement where the reviewer only issues a verdict and the producer does the fixing. By that logic, a review agent should have no tool for editing files.

Of the 14 review definitions, counted over the same scope since week one, 11 had the Edit or Write tool. Fortunately none of those 11 had ever been called, and the two that had actually been called were read-only. There were also 0 cases of a reviewer editing a file it was checking. Nothing had gone wrong, but write permission was open with no check on it.

I also looked at whether verdicts led to edits. The share of cases where a file was edited within 30 minutes of calling review was 27% for named review agents, 40% for review delegated to a generic agent, and 50% for Codex review. After calls that weren't reviews, which serve as the baseline, 40% also had an edit within 30 minutes. Being reviewed didn't make me edit more than usual. This method can't tell whether an edit within 30 minutes was caused by the review, though. I concluded that verdicts and their follow-through have to be tracked separately.

## A Subagent Output Token Counting Error

While measuring, my own numbers became subject to review too. The biggest error was in the measurement script.

I had measured that the main session produces 95.7% of output tokens even when work is delegated to subagents, and I had written that number into a global rule as its basis. Then, during another measurement, I noticed that the output token count was far too low for the amount of text the subagents had written. Opening the log, I found that a single response is stored as several lines while streaming, and my script was counting only the intermediate value from the first line. After I switched to the per-message maximum and re-measured, the figure was 69.9%, and subagent output grew from about 200 thousand to 2.21 million tokens.

The tool I used to audit review agents was itself getting no review. I fixed the number in the rule file at the same time.

## Reorganizing the Claude Code Agent Setup

I changed the setup based on the measurements. I followed three principles: don't delete definitions, back up before changing and test the rollback first, and check the names that skills call before moving anything out of the way.

| Group | Criterion | Result |
|---|---|---|
| Global, always on | Called often, or called by several skills | 13 |
| Plugin bundles | Used with specific skills only, and occasionally | Three: paper, PRD, design system (14 agents, 14 skills) |
| Archive | 0 calls, and no skill calls them by name | 7 |
| Project-specific | Only meaningful in that repository | Left untouched |

The original plan kept 7 global agents. A name search showed that moving the research and methodology agents, which the hackathon work and paper work share, into a plugin would break the hackathon skills, so I kept 6 in global. The analysis agent started as an archive candidate, but I found that the paper-writing skill calls it, so I kept it.

### Making Three Review Agents Read-Only

I created fact-checker and style-critic as new global agents and aligned their verdict format with the existing paper-critic. All three are limited to read-only tools.

```json
{"verdict": "ACCEPT|REVISE|BLOCK|ESCALATE",
 "justification": "one or two sentences",
 "evidence": [{"claim": "...", "ref": "file:line or URL", "verified": true}],
 "issues": [{"id": "F1", "where": "file:line", "problem": "...", "fix_hint": "..."}],
 "missing": ["what could not be verified"]}
```

I left out a score field. To record that evidence was checked, the agent must attach a file and line.

While cleaning up, I found one more reason for the zero calls. My style rules file told me to call style-critic, but that definition existed only inside one project folder. The file told me to call an agent that couldn't be called from any other folder.

### Moving Occasional Agents into Local Plugins

I use paper, PRD, and design-system work once every few weeks, yet their agents sat in every session's list. I made these three into local plugins, keep them off by default, and turn them on only when needed.

```
~/.claude/local-plugins/                  # local marketplace
  .claude-plugin/marketplace.json
  plugins/paper/
    .claude-plugin/plugin.json
    agents/*.md
    skills/<skill>/SKILL.md
```

```bash
claude plugin validate ~/.claude/local-plugins
claude plugin marketplace add ~/.claude/local-plugins
claude plugin install paper@kk-local
claude plugin disable paper@kk-local   # keep it off by default
claude plugin enable  paper@kk-local   # turn on when needed; run /reload-plugins in open sessions
```

Once inside a plugin, names get the plugin name as a prefix (`write-paper` becomes `/paper:write-paper`). So I had to update every place that used the old names, inside and outside the bundles, and I fixed 175 places inside the bundles and 6 outside with a script. I hit two pitfalls along the way.

- I missed names with a Korean particle attached directly, as in "readiness-reviewer가". The regex `\w` includes Korean characters, so no word boundary was detected. I fixed it by defining the boundary as only Latin letters, digits, and hyphens.
- When I ran the move script in zsh, nothing moved. zsh does not split `${list//,/ }` on spaces, so the loop ran only once. I reran it in bash.

After the move, I verified in a fresh session. With the plugin on, `prd:prd-author` showed up in the list and the call succeeded. With it off, the same question returned nothing.

### A Review Verdict Ledger and a Gate Script

The week-one measurement confirmed that adding more definitions doesn't make review get called. So I had each verdict written as one line to `.review/verdicts.jsonl` in the same folder as the deliverable, and made a gate script check that a passing verdict exists for the current content. Each verdict carries a hash of the text, so if the text changes after the verdict, it has to be reviewed again.

```bash
python3 review_gate.py post.md --need style-critic            # warn only
python3 review_gate.py post.md --need style-critic --strict   # block if not passed
```

There are four states: passed, no verdict, text changed after the verdict, and unclosed issues. I wrote 7 checks first, watched all 7 fail, and then implemented the script. The first time I ran it, it warned that my own case post had no verdict. For now it only warns for two weeks, and after that I plan to decide whether to switch to blocking mode.

### Cleaning Up the Stalled Resident Jobs

The multi-agent tool I used before handed off work by typing a message into a worker window and sending Enter. The message went in but the Enter often didn't register and things stalled, and on one day the recovery that unclogs the stuck queue ran 511 times. Running through my plan limit quickly while using Codex as a worker was another reason I let it go.

Yet three resident processes of that tool had been running for two months. They were frozen and made no model calls, but if they came back, they would send a nudge message to every stalled window, and each nudge re-reads the entire session (about 330 thousand tokens). I turned them off by unregistering them from launchd and kept the config files, so one command can bring them back.

I borrowed the review design from that tool: the reviewer only issues a verdict and does not fix, the verdict format and evidence notation are fixed, and Claude and Codex are mixed so they don't miss the same mistakes together. Only the handoff mechanism changed, to the Agent tool and a single `codex exec` run, neither of which needs keystrokes.

## Backing Up and Restoring `~/.claude`

`~/.claude` is configuration shared by every session, so a bad edit affects other work too. I made a backup before changing anything and tested the restore script first.

```bash
cd ~/.claude
tar -czf backups/20261022_agents_restructure.tgz agents rules skills settings.json
```

When Korean file names are compressed and extracted, the normalization form changes (NFC versus NFD), and `diff -r` reports everything as different. After normalizing the names to NFC and comparing contents, 0 of 505 files differed.

The restore script reverts only what I changed this time, and leaves alone any file another job created after the backup. By default it only shows what it would do, and it can revert partially.

```bash
bash restore.sh                          # print the to-do list only
bash restore.sh --apply                  # revert everything
bash restore.sh --apply --only plugins   # plugins only (archive, critics, refs, jarvis also work)
```

Each run compresses the state at that moment first, so after a restore I can also return to the pre-restore state. I tested on a copy of `~/.claude` with fake `claude` and `launchctl` commands. My first version deleted the whole skills folder and overwrote it with the backup, which would also wipe out skills that other sessions created after the backup. Testing exposed that problem, so I rewrote it to revert only what I changed.

## Review Agent Q&A

After I presented this in the study group, three of the questions connected to the topic of this post.

### Where to Attach Review Agents

The first question was where and how to attach review agents. You can call them as subagents in the same Claude session, set up a separate workspace that only does review, or attach several reviewers to a multi-agent tool or the Agent SDK. I answered that I had been creating one on the spot for each purpose inside my workspace, with requests like "make me a critic for papers" or "make me a reviewer that checks whether the numbers are wrong." That answer contained the reason why definitions piled up to 97 and why similar reviewers kept appearing project by project.

### Errors Review Agents Miss in Real Work

The second question was why the review errors were so many. The person asking had deliberately planted eight kinds of errors, such as swapped cause and effect, into a report drawn from raw data, tested it 24 times across three agent setups, and said the reviewers found all of them. The errors I met in real work were of a different kind. Examples were applying a log transform twice, comparing a log-transformed value against an untransformed source file, and cases where the processing I said I would do differed from the file I actually processed. Some were claims that overstated partial results. An error planted in a sentence shows up when you compare against the source, but errors from the calculation process and processing order are hard to see by reading only the result sentences.

### Domain Review Criteria and Separating the Writing Context

The study leader added two points. LLMs find logic errors better in fields with a lot of public material and miss more in specialized fields with little data, so in those fields you have to build a domain expert's review criteria into the harness. Also, if a reviewer has already seen the context of how something was written, it may pass the work to fit that context, so the reviewer should be kept apart from the writing context. That is also why I keep my paper-review agent separate rather than merging it into the critique agent. The review agent reads only the finished manuscript and does not see the analysis process.

## What I Learned

**A review agent gets called when the order of calls is fixed.** Writing a verdict record and having a gate check it led to real calls more than defining one more agent did.

**Reviewers only issue verdicts.** If they have no permission to fix, verdicts and edits don't get mixed, and I can count separately whether a verdict was actually applied.

**If the ground truth is in a file, use a check. If not, use review.** Whether config and documents disagree, or whether a banned phrase remains, can be blocked with a script. Only things whose ground truth is not in a file, such as the strength of a claim or a causal interpretation, go to review.

**Measurement tools are subject to review too.** The number I used as the basis for a rule was undercounting subagent output by more than a factor of ten. When a number looked off, opening the raw log was the fastest way to find out.

## Next

- After two weeks of warnings, decide whether to switch the gate to blocking mode.
- Use Codex for review only, with a cap of 10 calls a day.
- After four weeks, measure again with the same script and compare the number of global definitions, the number of entries in the verdict ledger, and the share of verdict issues that got closed.
