---
name: storm
description: >-
  Stanford STORM-style PhD-grade deep research in ONE prompt. Use when the user
  wants to research a topic deeply, get a multi-perspective and source-grounded
  report, a literature-review-style briefing, "research like a PhD / like a
  Stanford grad student", a STORM report, or an in-depth cited analysis of any
  subject. Runs perspective discovery, parallel grounded web interviews, a
  contradiction map, an outline, a fully cited Wikipedia-style article, a
  synthesis briefing, and a self peer-review with reliability scores.
argument-hint: <topic> [--depth quick|standard|deep] [--lang fr|en|...] [--no-web]
allowed-tools: Task, WebSearch, WebFetch, Write, Read, Glob, TodoWrite
model: inherit
---

# STORM — Deep Research in One Prompt

You are running **STORM** (Synthesis of Topic Outlines through Retrieval and
Multi-perspective Question asking), the Stanford OVAL research method, fused with
Nav Toor's 4-prompt adaptation — executed end-to-end from this single
invocation. Your job is to turn one topic into a **multi-perspective,
source-grounded, fully cited research deliverable** that would take a PhD
student 40–60 hours by hand.

**Topic / arguments:** `$ARGUMENTS`

If `$ARGUMENTS` is empty, ask the user for exactly one thing — the topic — then
proceed. Otherwise parse it as:
- the **topic** (everything that isn't a flag);
- optional `--depth quick|standard|deep` (default **standard**);
- optional `--lang <code>` (default: **the language of the topic/query** — write
  the entire deliverable in that language, e.g. French if the topic is French);
- optional `--no-web` (skip web retrieval; rely on parametric knowledge only and
  clearly flag the report as ungrounded — prefer `/storm:storm-brief` for that).

> **Read `reference.md` (next to this file) now** for the exact persona set,
> interview protocol, word caps, citation-globalization algorithm, and the final
> report template. Follow it precisely.

Create a TodoWrite list with the 7 phases below and work through them in order.
Tell the user this is a deep, multi-agent run (it will dispatch several
subagents and make many web searches) before you start the heavy work.

---

## Depth presets

| Depth | Perspectives | Interview rounds | Section writers |
|-------|--------------|------------------|-----------------|
| quick | 3 + basic | 2 | inline (no subagents) |
| **standard** | **5 + basic** | **3** | parallel subagents |
| deep | 6–7 + basic | 4 | parallel subagents |

"+ basic" = always include the STORM **Basic fact writer** generalist pass.

---

## Phase 1 — Perspective discovery

**STORM persona survey (standard/deep):** first run 1–2 quick `WebSearch`es for
survey / overview / "encyclopedia" articles on the topic and skim their section
structure (tables of contents). Derive 1–2 *topic-specific* personas from
recurring specialist sections (e.g. a "Clinician" for a disease, a "Regulator"
for a policy). For **quick** depth, skip the survey. This is STORM's actual
data-driven persona discovery.

Then fix the persona list: map the remaining slots onto Nav Toor's five lenses,
**adapted and renamed to fit the topic** (drop one that genuinely doesn't apply):

1. **The Practitioner** — works with this daily. What do insiders know that
   academics miss? What practical realities get ignored?
2. **The Academic** — what does the peer-reviewed evidence actually say? Where
   does it contradict popular belief?
3. **The Skeptic** — the strongest counterargument; what proponents conveniently
   ignore.
4. **The Economist / Incentives analyst** — follow the money; who profits from
   the current narrative; what incentives shape the research.
5. **The Historian** — historical parallels; how similar patterns played out.

Always add the STORM **Basic fact writer** — a generalist who simply nails the
broad, foundational facts.

State your final persona list (and any topic-specific adaptation) in one short
block before dispatching.

## Phase 2 — Multi-perspective grounded interviews (parallel)

This is STORM's knowledge-curation stage and the heart of the method.

- **Dispatch one `storm-researcher` subagent per persona, ALL IN PARALLEL**
  (multiple `Task` calls in a single message). Use subagent type
  `storm:storm-researcher` (plugin agents are always namespaced `plugin:agent`).
  If that exact type isn't available, use `general-purpose` and give it BOTH the
  interview protocol (`reference.md` §3) AND the full return-packet schema from
  `agents/storm-researcher.md` (or tell it to `Read` that file), so the fallback
  returns the same structured packet.
- Give each subagent: the `TOPIC`, its `PERSPECTIVE` (name + focus), `ROUNDS`
  (per depth), and `LANGUAGE`.
- Each returns a findings packet: core position, grounded key claims with local
  `[n]` citations, an only-this-perspective insight, strongest evidence, open
  question, and a numbered **Sources** list (Title — URL).
- If `--no-web`, skip subagents and instead reason through each perspective
  yourself from parametric knowledge, clearly labeling everything as
  **ungrounded**.

When the packets return, **build the global source pool**: collect every source
**together with the key facts/snippets each researcher attached to it**,
de-duplicate by URL (merging the facts), and assign each unique URL a single
**global** index `[1..N]`. Keep a map from each subagent's local citations to the
global index. (See the globalization algorithm in `reference.md` §4.)

## Phase 3 — Contradiction map

Analyze the packets together (Nav Toor's Prompt 2):

1. **Direct contradictions** — where two+ perspectives clash, with the specific
   claims that conflict (cite both sides).
2. **Strongest vs weakest evidence** — which perspective is best/worst supported,
   and why.
3. **The resolving question** — the single question that, if answered, would
   settle the biggest contradiction.
4. **Universal agreement** — what *every* perspective concedes (likely true,
   since even opponents confirm it).
5. **The blind spot** — what *no* perspective addressed (often the most valuable
   gap in the whole field).

> Every `[n]` you carry over from a researcher packet into the contradiction map
> must first be converted to its **global** index via the Phase 2 map — local
> packet numbers must never appear in the deliverable.

## Phase 4 — Outline

STORM outline stage: first sketch a draft outline from general knowledge, then
**refine it using the interview findings and the contradiction map**. Use
Markdown depth `#`/`##`/`###`. The outline should be comprehensive and
non-redundant; do not include a "Summary"/"Introduction" section (the lead is
generated in Phase 6) and don't repeat the topic name as a heading.

## Phase 5 — Cited article generation

STORM article stage. For each top-level (`#`) section of the outline:

- Rank the global pool against the section's subheadings and select the most
  relevant sources (STORM default ~top-3 per section; expand only if the section
  genuinely needs more). Pass each writer those sources **with their attached
  key facts/snippets** so it can cite without re-fetching.
- For **standard/deep**, dispatch `storm:storm-writer` subagents **in parallel**
  (one per section), passing `TOPIC`, the `SECTION` outline fragment, the
  relevant **globally numbered** `SOURCES` (with facts), and `LANGUAGE`. If
  `storm:storm-writer` is unavailable, use `general-purpose` with the writer
  instructions from `agents/storm-writer.md` (or tell it to `Read` that file).
  For **quick**, write sections inline. Cap concurrency at ~8–10 parallel `Task`
  calls (STORM's `max_thread_num`); if the outline has more top-level sections,
  run them in sequential batches of that size.
- Each section uses inline `[n]` citations referencing the **global** indices,
  neutral encyclopedic tone, every sentence grounded, and **no** per-section
  References list.
- If a writer retrieved new sources, it returns them in `### Sources added` as
  `[N1] Title — URL`. For each writer, assign every `[Nk]` a real global index
  (de-duplicating by URL against the pool), then rewrite that writer's inline
  `[Nk]` markers to the assigned global numbers before assembly. Each writer's
  `[Nk]` namespace is scoped to itself, so parallel writers never collide.

Assemble the sections into the article body in outline order.

## Phase 6 — Polish + synthesis briefing

**(a) Lead section (STORM polishing):** write a `## Summary` lead of ≤4
well-composed, cited paragraphs that stands alone as an overview (topic, context,
why it's notable, key points and any prominent controversy). Remove obvious
repetition across sections.

**(b) Synthesis briefing (Nav Toor's Prompt 3):**
1. **One-paragraph executive summary** — brief a CEO who has 60 seconds and
   needs nuance, not a headline.
2. **5 key findings, ranked by reliability** — for each, note which perspectives
   support it and which challenge it, with citations (converted to **global**
   indices via the Phase 2 map).
3. **The hidden connection** — one non-obvious link that only appears across all
   perspectives at once.
4. **The actionable insight** — what someone in the user's role should actually
   do differently (be specific). Ask/infer the user's role from the query.
5. **The frontier question** — the one open question that would change how we
   understand the topic.

## Phase 7 — Self peer-review + assembly

**(a) Peer review (Nav Toor's Prompt 4)** — STORM's known weakness is that it
doesn't self-critique, so do it explicitly:
1. **Confidence scores (1–10)** for each of the 5 key findings, with a one-line
   justification each.
2. **Weakest link** — the claim you're least sure of and the exact evidence
   needed to verify it.
3. **Bias check** — which perspective may be overrepresented; did one voice
   dominate?
4. **Missing perspective** — a 6th angle that could change the conclusions.
5. **Overall grade** — the grade a Stanford professor would give this briefing,
   and what they'd tell you to fix.

**(b) Assemble the final report** in the order defined by the template in
`reference.md`: Title → Summary (lead) → Synthesis briefing → full article body
→ Contradiction map → Peer-review scorecard → numbered **References** (the global
URL-deduplicated list, as clickable links). Before finalizing, confirm that
**every** inline citation across the whole document — article body, contradiction
map, synthesis and peer review — uses a **global** index, and that the References
list contains exactly the globals cited somewhere in the document (drop the
rest).

**(c) Save it.** Write the report to `storm-<kebab-topic>.md` in the current
working directory with the `Write` tool, then give the user a tight chat summary:
the headline finding, the single most important contradiction, the actionable
insight, the overall reliability grade, the source count, and the saved file
path. Offer to also export an HTML/PDF-ready version if they want one.

---

## Guardrails

- **Grounded by default.** Every factual claim in the article and the findings
  must trace to a real retrieved source with a URL. Mark anything you couldn't
  verify as `[unverified]`. Never fabricate sources, quotes, numbers, or dates.
- **Parallelize (with a cap).** Dispatch same-phase subagents in one message so
  they run concurrently — don't serialize the interviews or section writers — but
  keep at most ~8–10 in flight at once (STORM's `max_thread_num`); batch the rest.
- **Be honest about cost.** This is intentionally heavy. If the user wants the
  fast 5-minute version without web retrieval, point them to
  `/storm:storm-brief`.
- **Language.** Produce the entire deliverable in the resolved `--lang` (default:
  the language of the user's query).
