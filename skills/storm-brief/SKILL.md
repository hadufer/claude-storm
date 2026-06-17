---
name: storm-brief
description: >-
  The fast 5-minute STORM briefing — Nav Toor's exact 4-prompt method
  (multi-perspective scan → contradiction map → synthesis → peer review) run in a
  SINGLE shot from the model's own knowledge, with no web search and no
  subagents. Use when the user wants a quick multi-perspective deep-think /
  research briefing on a topic and does not need live sources. For grounded,
  cited, web-researched reports use /storm:storm instead.
argument-hint: <topic> [--role <your role>] [--lang fr|en|...]
allowed-tools: Write, Read
model: inherit
---

# STORM Brief — the 4-prompt method in one shot

This is Nav Toor's "Stanford STORM Method" run as a single prompt. It produces a
multi-perspective research briefing in ~one pass, **from your own knowledge** —
fast, but **not source-grounded**. For real web research with citations, use
`/storm:storm`.

**Topic / arguments:** `$ARGUMENTS`

Parse `$ARGUMENTS` as the **topic** plus optional `--role <role>` (the reader's
role, used for the actionable insight) and `--lang <code>` (default: the language
of the query). If empty, ask only for the topic.

Run all four phases below in order, in one response, and write the entire
briefing in the resolved language. Be intellectually honest: because this run has
**no live sources**, treat every claim as your best recollection and flag
anything uncertain — the peer-review phase exists precisely to expose that.

## Phase 1 — Multi-Perspective Scan

Simulate **5 distinct expert perspectives**, adapting the lenses to the topic:

1. **The Practitioner** — works with this daily. What do they know that
   academics miss? What practical realities are usually ignored?
2. **The Academic** — what does the peer-reviewed evidence actually say? Where
   does it contradict popular belief?
3. **The Skeptic** — the strongest counterargument; what proponents conveniently
   ignore.
4. **The Economist** — follow the money; who profits from the current narrative;
   what financial incentives shape the research.
5. **The Historian** — what historical parallels exist; how those played out.

For **each** perspective give:
- core position in 2 sentences,
- the strongest evidence supporting it,
- the one thing they'd tell you that no other perspective would.

## Phase 2 — Contradiction Map

1. Where do 2+ perspectives directly contradict? List each conflict with the
   specific clashing claims.
2. Which perspective has the strongest evidence? Which the weakest? Why?
3. The one question that, if answered, would resolve the biggest contradiction.
4. What does EVERY perspective agree on? (Likely true — even opponents confirm
   it.)
5. What did NO perspective address? (The blind spot in the whole field — often
   the most valuable finding.)

## Phase 3 — Synthesis Briefing

1. **One-paragraph summary** — brief a CEO who has 60 seconds and needs nuance,
   not just the headline.
2. **5 key findings, ranked by reliability** — for each, note which perspectives
   support it and which challenge it.
3. **The hidden connection** — one non-obvious link visible only across all 5
   perspectives at once.
4. **The actionable insight** — what someone in the reader's role (`--role`, or
   ask/infer it) should actually DO differently. Be specific.
5. **The frontier question** — the one question that would change everything
   about how we understand this topic.

## Phase 4 — Peer Review

1. **Confidence scores (1–10)** for each of the 5 key findings, each justified.
2. **Weakest link** — the claim you're least confident in and what info would
   verify it.
3. **Bias check** — which perspective might be overrepresented; did one voice
   dominate?
4. **Missing perspective** — a 6th angle that could change the conclusions.
5. **Overall grade** — what grade a Stanford professor would give this briefing
   and what they'd tell you to fix.

## Finish

Present the full briefing in chat. If the user asks, save it to
`storm-brief-<kebab-topic>.md`. Then suggest: *"Want the source-grounded version
with real citations? Run `/storm:storm <topic>`."*
