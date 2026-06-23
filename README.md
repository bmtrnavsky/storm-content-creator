# STORM Content Creator

**Version:** 0.1.0
**License:** MIT

**This is a methodology reference and AI agent skill specification, not a runnable code implementation.** It provides operational instructions for executing the five-phase STORM research pipeline with human-in-the-loop expert interviews.

## What Is STORM?

Most people use AI incorrectly for research. They give a single prompt like "write a report on this topic" and get back a shallow, single-perspective response that might include hallucinations. The AI has no process for asking good questions, no mechanism for seeking diverse viewpoints, and no way to ground claims in real sources.

**STORM** (Synthesis of Topic Outlines through Retrieval and Multi-perspective Question Asking) is a research method developed at Stanford University's Oval Lab (Shao et al., NAACL 2024). It solves the shallow-research problem by using Gen AI to mimic how an investigative journalist works -- but at machine speed and scale:

1. **Gen AI discovers multiple perspectives** -- the AI maps out 6-8 distinct expert viewpoints the topic deserves, before any research begins
2. **Gen AI runs simulated expert interviews** -- the AI plays dual roles (interviewer and expert) in 6-8 separate multi-turn conversations, each grounded in real internet sources found via web search
3. **Gen AI synthesizes findings** -- after all simulated interviews are complete, the AI presents a structured brief to the human expert: "Research says X. From your experience, does this track? What is missing? What is wrong?"
4. **Human expert stress-tests** -- the human challenges the AI's conclusions, adds angles the AI missed, and corrects framing errors rooted in lived experience. The AI follows up, sharpens, and challenges back.
5. **Gen AI curates findings** -- the AI organizes all interview logs and human input into a clean, hierarchical outline with every claim mapped to a source
6. **Gen AI writes with evidence** -- the AI produces the article section by section, with every factual claim pointing back to a collected source
7. **Gen AI audits for blind spots** -- a moderator agent sweeps for unknown unknowns, source bias, and false connections

**The human expert does not discover perspectives, run interviews, or write the report.** Gen AI handles all of that. The human comes in at defined checkpoints to challenge the conclusions against lived experience. The human adds nuance, catches framing errors, and flags angles the AI missed. The AI does the heavy lifting. The human catches what the AI cannot: framing errors rooted in lived experience, domain-specific nuance that never makes it into published sources, and conclusions that are technically correct but practically wrong.

The result: research reports with comparable breadth and depth to Wikipedia articles, but grounded in the practitioner's own expertise and experience.

## What This Repo Adds

This is an adaptation of Stanford's STORM method for content creation with three key innovations:

1. **Human Expert Stress-Test** -- after Gen AI completes all 6-8 simulated interviews and synthesizes the findings, a real domain practitioner reviews and challenges the AI's conclusions. This is additive, not a replacement. The AI does the research. The human stress-tests it. No published STORM implementation includes this step.
2. **Three-Layer Search** -- Gen AI searches across three layers during interviews: web search for current facts, RAG (personal knowledge management) for cross-domain connections from prior notes, and session memory for the human's own prior thinking. The RAG layer makes the output distinctly personal rather than generic AI content.
3. **Model Tiering** -- fast, capable models for high-volume AI research stages (6-8 interviews), stronger models for reasoning stages (outline, writing, moderator). Manages cost without sacrificing quality where it matters.

Based on:
- [STORM Paper](https://arxiv.org/abs/2402.14207) -- Shao et al., Stanford Oval Lab, NAACL 2024
- [Co-STORM Paper](https://arxiv.org/abs/2408.15232) -- Collaborative extension, EMNLP 2024
- [Reference Implementation](https://github.com/stanford-oval/storm) -- 29.2k stars

---

## Table of Contents

- [Core Insight](#core-insight)
- [The Five-Phase Pipeline](#the-five-phase-pipeline)
- [Model Tiering](#model-tiering)
- [Search Stack Detail](#search-stack-detail)
- [When to Use Full vs Lightweight](#when-to-use-full-vs-lightweight)
- [Evaluation Data](#evaluation-data)
- [Known Failure Modes](#known-failure-modes)

---

## Core Insight

Direct prompting produces shallow, single-perspective questions. STORM solves this by mimicking an investigative journalist: discover multiple perspectives, interview experts, ground every claim in retrievable sources.

**Phase 2 runs two stages:** First, 6-8 simulated AI expert interviews (the original STORM method). Then, a human expert stress-test pass that challenges the AI findings against lived experience. The human does not replace the simulated interviews -- they add a layer on top.

---

## The Five-Phase Pipeline

### Phase 1: Perspective Discovery

**Role:** Senior Research Strategist

**Task:** Map 6-8 distinct expert perspectives the topic deserves.

**Output:** For each perspective: persona name, what they care about, sharpest questions they would ask.

**Key mechanism:** Survey related articles to discover diverse viewpoints automatically. Different perspectives produce different questions; generic prompts produce generic questions.

**Human checkpoint:** Present perspectives to the domain expert for approval before proceeding. Expert can reject, add, or modify. This early gate saves hours of wasted work on wrong angles.

---

### Phase 2: Expert Interview (Human-in-the-Loop Modification)

**Role:** Interviewer (AI) + Expert (Human)

**Task:** Multi-turn back-and-forth dialogue for one perspective at a time.

**Search stack (three layers):**

| Layer | Tool | Use Case |
|-------|------|----------|
| Web Search | `web_search` | Broad internet facts, current data, specific claims |
| RAG | Vector + full-text search over personal vault | Cross-domain connections from prior knowledge |
| Session Memory | `session_search` | Prior thinking on related topics (run FIRST to avoid redundancy) |

**Process:**

1. Run 6-8 simulated expert interviews (one per perspective). Each is a multi-turn dialogue where an AI interviewer asks questions and an AI expert answers, grounded in real sources from web search. This is the original STORM method.
2. After simulated interviews are complete, run a Brad interview pass. AI presents the synthesized findings from all simulated interviews to Brad as structured interview: "Research says X. From your experience, does this track? What is missing? What is wrong?"
3. Brad responds. AI follows up, sharpens, challenges. Brad can add entirely new angles the simulated interviews missed.
4. Repeat the Brad interview pass for each perspective where Brad has relevant domain experience.

**Hard rule:** If a claim cannot be backed by a real source AND the expert cannot confirm from experience, output `unverified`. Fabrication is strictly forbidden.

**Why this matters:** No published STORM implementation replaces the simulated expert with a real human practitioner. The human catches framing errors, adds nuance from lived experience, and stress-tests conclusions against reality.

---

### Phase 3: Curate and Outline

**Role:** Meticulous Editor

**Task:** Organize interview logs into a clean, hierarchical outline.

**Output:** Sections grouped by theme, duplicates removed, contradictions between sources explicitly flagged. Every section mapped to specific sources collected during interviews.

**Living outline:** The outline is not static. It evolves as interviews surface new information. Use insert and reorganize operations as research progresses. This mirrors Co-STORM's dynamic mind map concept.

---

### Phase 4: Grounded Writing

**Role:** Technical Writer

**Task:** Write report section by section following the outline exactly.

**Constraint:** Neutral, factual tone. Every claim points back to a collected source.

**Hard rule:** If a section is thin on data, write `needs more research` instead of padding with generic fluff.

---

### Phase 5: Moderator Pass

**Role:** Independent Moderator / Auditor

**Task:** Blind spot sweep for unknown unknowns -- critical questions no one thought to ask.

**Why this is the highest-leverage role:** The Co-STORM evaluation found that removing the moderator hurts performance more than reducing the number of experts. Single expert + moderator beats multiple experts without a moderator. This is the most counterintuitive, high-leverage finding in the system.

**Red-team constraints:** Audit for two named failure modes:

1. **Source bias transfer** -- leaning too heavily on a single biased source
2. **Over-association of unrelated facts** -- creating red herrings by connecting facts that are not actually related

**Output:** Flagged errors fixed before final output.

**Human checkpoint:** Present moderator findings to the expert for review. Expert approves fixes or adds direction.

---

## Model Tiering

The STORM reference implementation supports different models for different stages. This adaptation applies tiering to manage cost and speed:

| Pipeline Stage | Model | Rationale |
|----------------|-------|-----------|
| Phase 1: Perspective Discovery | DeepSeek 4 Flash | Fast, strong at structured research output |
| Phase 2: Expert Interview | DeepSeek 4 Flash | High-volume token burn; fast and capable |
| Phase 3: Curate and Outline | Owl Alpha (Sonnet 4.6) | Structured output, reliability |
| Phase 4: Grounded Writing | Owl Alpha | Voice matching for human hand-edit pass |
| Phase 5: Moderator/Auditor | Owl Alpha | Highest-leverage role; needs real reasoning strength |
| Final Polish | DeepSeek 4 Flash | Mechanical task (dedup, summary); speed only |

**Model terminology:** Owl Alpha, Sonnet 4.6, and DeepSeek 4 Flash are specific model identifiers in the Hermes/OpenRouter ecosystem. Owl Alpha = Sonnet 4.6 capability level. DeepSeek 4 Flash = Gemini Flash capability level or better, significantly faster.

**Fallback:** If the primary model hits a reasoning ceiling on the moderator role for complex cross-domain topics, escalate to a stronger model. Do not escalate as a reflex.

---

## Search Stack Detail

### Web Search

- Use for: broad internet facts, current data, specific claim verification
- Query formulation: use synthesis-forcing queries (`what does research say about X`) not just topic queries (`X`)
- Run during Phase 2 interviews to ground answers in real sources

### RAG (Personal Knowledge Management)

- Use for: surfacing vault knowledge, cross-domain connections from prior notes
- Query during interviews: `what has the expert already written about [concept]`
- This is the differentiator -- makes output distinctly personal rather than generic AI content
- Supports both hybrid (vector + full-text) and semantic-only search

### Session Memory

- Use for: checking prior thinking on related topics
- Run BEFORE fresh research in Phase 2 to avoid redundancy
- No API cost, often overlooked

---

## When to Use Full vs Lightweight

**Full STORM (all 5 phases):**
- Cornerstone content, major research pieces, definitive guides
- Requires expert time for Phase 2 interviews and Phase 5 review

**Lightweight STORM (Phases 1, 3, 5 only):**
- Regular blog posts, shorter pieces
- Skip the interview phase; use perspective discovery + curation + moderator pass only

**Decision criteria:**

| Criteria | Full | Lightweight |
|----------|------|-------------|
| Pillar-level piece | Yes | No |
| Crosses multiple domains | Yes (but see note) | No |
| Time-sensitive / news-driven | No | Yes |
| Expert available for checkpoints | Yes | No |

**Note on "crosses multiple domains":** Almost every S2BI post touches DC operations + BI + technology + strategy. That is the nature of Brad's cross-domain expertise. Apply this criterion pragmatically: if the domains are tightly integrated in one lived experience, it is still one domain. Use full STORM when the topic requires genuinely distinct domains that the expert has NOT connected before.

---

## Evaluation Data

Co-STORM evaluation (20 participants, human evaluation) against RAG chatbots:

| Metric | RAG Chatbot | STORM+QA | Co-STORM |
|--------|-------------|----------|----------|
| Report Relevance | 3.57 | 3.61 | **3.78** |
| Report Breadth | 3.50 | 3.61 | **3.79** |
| Report Depth | 3.26 | 3.43 | **3.77** |
| Report Novelty | 2.44 | 2.50 | **3.05** |
| Unique URLs | 2.94 | 2.89 | **6.04** |

Key findings:
- 78% preferred Co-STORM over RAG chatbots overall
- 67% rated it higher in Breadth and Serendipity
- Moderator removal hurts performance more than expert removal
- Single expert + moderator > multiple experts without moderator

---

## Known Failure Modes

From the Co-STORM paper:

1. **Source bias transfer** -- leaning too heavily on a single biased source
2. **Over-association of unrelated facts** -- connecting things that should not be connected

Both should be named checks in any moderator/auditor phase.
