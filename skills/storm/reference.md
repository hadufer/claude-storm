# STORM — Reference (exact protocol, prompts, conventions)

This file is the source of truth for the `/storm:storm` pipeline. It encodes the
real Stanford STORM algorithm (verbatim defaults and prompt intents from
`github.com/stanford-oval/storm`, NAACL 2024 / arXiv:2402.14207) and the
4-prompt adaptation by Nav Toor (@heynavtoor).

---

## 1. STORM defaults (mirror these)

| Parameter | STORM default | Used here as |
|-----------|---------------|--------------|
| Generated perspectives | 3 | 5 (Nav Toor lenses), adapt to topic |
| Default persona | "Basic fact writer" (always first) | always include |
| Interview rounds / persona (`max_conv_turn`) | 3 | depth: 2 / **3** / 4 |
| Search queries / question (`max_search_queries_per_turn`) | 3 | up to 3 |
| Search results kept / query (`search_top_k`) | 3 | top ~3 |
| References retrieved / section (`retrieve_top_k`) | 3 | ~top-3 per section (expand if needed) |
| Parallelism (`max_thread_num`) | 10 | dispatch in parallel, **≤ ~10 at once** |
| Conversation history cap | 2500 words | keep packets dense |
| Conversation info cap | 1000 words | — |
| Section info cap | 1500 words | — |
| Lead/summary length | ≤ 4 paragraphs | — |

STORM module order: **Knowledge Curation → Outline Generation → Article
Generation → Article Polishing**. The original paper reported multi-perspective
articles were ~25% more organized and ~10% broader in coverage than
single-prompt baselines — that gain is exactly what the perspective fan-out buys.

---

## 2. The personas (Phase 1)

Always include the generalist; default the rest to the five Nav Toor lenses,
adapted per topic.

- **Basic fact writer** — *"focusing on broadly covering the basic facts about
  the topic."* (STORM's default persona, always present.)
- **The Practitioner** — works with this daily; practical realities academics
  miss.
- **The Academic** — what the peer-reviewed evidence actually says; where it
  contradicts popular belief.
- **The Skeptic** — strongest counterargument; what proponents ignore.
- **The Economist / Incentives** — who profits; what incentives shape the
  narrative and the research.
- **The Historian** — historical parallels; how similar patterns played out.

STORM's own persona discovery works by surveying the tables of contents of
related reference articles and inventing editors that match those structures.
**Operationalize it (Phase 1, standard/deep):** run 1–2 quick searches for
survey/overview articles, skim their section structure, and derive 1–2
topic-specific personas from recurring specialist sections (e.g. a "Clinician"
for a disease, a "Regulator" for a policy) before mapping the rest onto the Nav
Toor lenses. Skip the survey for `quick` depth.

---

## 3. Interview protocol (Phase 2) — what each `storm-researcher` does

Per round (repeat for ROUNDS):

1. **Ask one persona-guided question** — never repeat a prior question; go
   deeper each round. (STORM's `AskQuestionWithPersona`: *"You are an
   experienced Wikipedia writer … you have specific focus when researching the
   topic … Ask good questions to get more useful information. Please only ask a
   question at a time and don't ask what you have asked before."*)
2. **Question → up to 3 search queries.** (STORM's `QuestionToQuery`: *"You want
   to answer the question using Google search. What do you type in the search
   box?"*)
3. **Retrieve** the queries (WebSearch + WebFetch the strongest hits); keep ~top
   3 per query.
4. **Answer grounded in the snippets only**, every sentence cited `[n]`. (STORM's
   `AnswerQuestion`: *"You are an expert who can use information effectively …
   Make your response as informative as possible, ensuring that every sentence
   is supported by the gathered information … If no appropriate answer can be
   formulated, respond with 'I cannot answer this question based on the available
   information.'"*) **No grounding ⇒ no claim.**

The packet schema each researcher returns is defined in
`agents/storm-researcher.md`.

---

## 4. Citation globalization algorithm (after Phase 2, and again after Phase 5)

STORM writes each section's citations **locally** (`[1..n]` per source block),
then `post_processing()` renumbers them into a single global, URL-deduplicated
reference list. Reproduce this:

1. Collect every source from every researcher packet (and later, every writer
   addition): `{title, url, publisher?, date?, key_facts}` — `key_facts` are the
   grounded snippet(s) the researcher cited from that source, carried along so
   section writers receive facts, not bare URLs. Writer additions arrive in the
   reserved `[Nk]` namespace, scoped per writer.
2. **De-duplicate by canonical URL** (strip tracking params, trailing slashes,
   `#fragments`; treat `http`/`https` and `www.` variants as the same).
3. Assign each unique URL one **global** index, numbered in order of first
   appearance: `[1], [2], …, [N]`.
4. Build a map `localCitation(agent, k) → globalIndex` (this also covers each
   writer's `[Nk]` additions, scoped per writer so they never collide). When you
   stitch the **findings, contradiction map, synthesis, article sections and
   peer review** together, rewrite every inline citation to its global index.
5. **Drop unused** sources from the final References list; keep only globals that
   are actually cited in the final document.
6. Render inline citations as concatenated brackets with no separators:
   `…published at NAACL 2024.[1][3]`.
7. The **References** section lists `[n] Title — URL` (clickable), in global
   order.

---

## 5. Nav Toor's 4 prompts (the method this 1-prompt run automates)

Kept verbatim for fidelity — the pipeline performs all four, but grounded in real
sources instead of pure simulation.

**Prompt 1 — Multi-Perspective Scan:** simulate 5 expert perspectives
(Practitioner, Academic, Skeptic, Economist, Historian); for each give the core
position in 2 sentences, the strongest supporting evidence, and the one thing
they'd tell you that no other perspective would. → **Phase 1–2.**

**Prompt 2 — Contradiction Map:** where do 2+ perspectives directly contradict
(list the clashing claims); which has the strongest/weakest evidence; the one
question that would resolve the biggest contradiction; what EVERY perspective
agrees on (likely true); what NONE addressed (the field's blind spot). →
**Phase 3.**

**Prompt 3 — Synthesis:** one-paragraph CEO summary (nuance, not headline); 5 key
findings ranked by reliability (note supporting/challenging perspectives); the
hidden connection visible only across all perspectives; the actionable insight
for the reader's role; the frontier question that would change everything. →
**Phase 6b.**

**Prompt 4 — Peer Review:** confidence scores 1–10 per finding (justified);
weakest link + what would verify it; bias check (which voice dominated); missing
6th perspective; overall grade a Stanford professor would give + what to fix. →
**Phase 7a.**

---

## 6. Final report template (Phase 7b)

Produce the deliverable in this exact order, in the resolved language:

```markdown
# <Topic> — STORM Research Report

> Generated with the STORM method (multi-perspective grounded research).
> Depth: <quick|standard|deep> · Perspectives: <n> · Sources: <N> · Date: <date>

## Summary
<≤4 cited paragraphs — the standalone lead.>

## Synthesis Briefing
**Executive summary (60s):** <one paragraph.>

**Key findings (ranked by reliability):**
1. **<finding>** — Reliability <x>/10. Supported by <perspectives>; challenged by
   <perspectives>. [n][n]
2. ...

**Hidden connection:** <...>

**Actionable insight (for <role>):** <...>

**Frontier question:** <...>

## <Article body — outline sections with inline [n] citations>
### ...
### ...

## Contradiction Map
- **Conflicts:** <perspective A claims … [n] vs perspective B claims … [n]>
- **Strongest / weakest evidence:** <...>
- **Resolving question:** <...>
- **Universal agreement (likely true):** <...>
- **Blind spot (nobody addressed):** <...>

## Peer Review & Reliability Scorecard
| Finding | Confidence (1–10) | Why |
|---|---|---|
| ... | ... | ... |

- **Weakest link:** <claim + what would verify it>
- **Bias check:** <which voice dominated>
- **Missing perspective:** <6th angle>
- **Overall grade:** <grade> — <what a Stanford professor would fix>

## References
[1] <Title> — <https://url>
[2] <Title> — <https://url>
...
```

Save as `storm-<kebab-topic>.md` in the working directory. Offer an HTML export
(self-contained, print-to-PDF friendly) on request.

---

## 7. Honesty rules

- Grounded by default; mark `[unverified]` for anything web search couldn't
  confirm; never fabricate sources/quotes/numbers/dates.
- Surface real disagreement among sources rather than smoothing it over.
- Note recency/staleness of pivotal evidence.
- The peer-review section must be genuinely critical — it is the fix for STORM's
  documented lack of self-critique (source bias, fact misassociation).
