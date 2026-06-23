# PKM vs PCM: A Comparative Analysis

**Research method:** STORM (Stanford, NAACL 2024) with Human Expert Stress-Test
**Expert:** Brad Trnavsky (Navy FMF Corpsman,Operations Supervisor, MISM candidate, AI systems builder)
**Date:** 2026-06-23
**Perspectives consulted:** 6 (PKM Practitioner, PCM Practitioner, RAG Engineer, Organizational Learning Theorist, AI Agent Architect, Skeptic)

---

## Executive Summary

Personal Knowledge Management (PKM) and Personal Context Management (PCM) are often treated as competing approaches to the same problem: how do you make what you know retrievable and useful? This research argues they are complementary systems operating at different time scales and serving different cognitive functions. The optimal architecture uses both.

PKM is about durable knowledge -- the compounding, linked, enriched understanding that persists and grows over months and years. PCM is about situational context -- the recent, relevant, conversational understanding that gets you through the current task.

The evidence from Brad's own living system (2,000+ Obsidian notes, Hindsight episodic memory, session search, pgvector RAG) shows that neither approach alone is sufficient. The retrieval stack needs both a knowledge layer (PKM) and a context layer (PCM), with distinct storage, retrieval, and decay characteristics for each.

---

## The Six Perspectives

### Perspective 1: The PKM Practitioner

**What they care about:** Long-term knowledge compounding, note enrichment, retrieval quality over time.

Brad's PKM system (the "Living Wiki") is a two-layer architecture built in Obsidian:

- **Layer 1: Capture and Structure.** AI agents seed notes from research and daily inputs. Notes are tagged, linked, and organized by theme. The vault has grown to 2,000+ notes across AI technology, personal development, leadership, and operational domains.
- **Layer 2: Enrichment and Retrieval Practice.** Brad enriches AI-seeded notes with personal experience, then the spaced retrieval practice system surfaces notes at calculated intervals (Day 1, 3, 7, 21) to re-encode them into long-term memory.

The core thesis of PKM: your knowledge architecture is either compounding or decaying. Notes without a retrieval mechanism become digital graves. The quality of output is bounded by the quality of the context you provide. When you ask an AI a question, it can only work with what it can find.

**Sharpest questions:**
- Is a note that has never been retrieved after 6 months actually knowledge, or just data?
- Does linking create real understanding or just the illusion of connection?
- How do you measure knowledge compounding in a PKM system?

### Perspective 2: The PCM Practitioner

**What they care about:** Recency, relevance, real-time context assembly for the current task.

Brad's PCM system operates through two mechanisms:

1. **Hindsight (Episodic Memory):** A separate FastAPI service backed by PostgreSQL 18 with pgvector. Stores discrete memories from past sessions. Three operations: Retain (store), Recall (retrieve), Reflect (synthesize patterns). Implements a Dual-Horizon Memory Model: short-term memories decay after 24 hours unless accessed 3+ times (promoted to long-term); long-term memories persist by category.

2. **Session Search:** Full-text search over past conversational sessions. Surfaces what Brad thought, decided, or discussed in prior conversations. Captures intent and reasoning chains that never made it into a note.

The SECI model (Nonaka & Takeuchi) explains why PCM matters at the personal level. Knowledge does not transfer -- it converts. The cycle runs: Socialization (tacit to tacit) -> Externalization (tacit to explicit) -> Combination (explicit to explicit) -> Internalization (explicit to tacit). Hindsight captures the externalization that would otherwise be lost. Session search captures the combination that happens across conversations.

**Sharpest questions:**
- How do you decide what gets retained in episodic memory versus what gets vaulted into PKM?
- Is session memory more valuable than vault memory for decision-making, or less?
- What decays faster: facts or the reasoning chains that produced them?

### Perspective 3: The RAG Engineer

**What they care about:** Embedding quality, chunking strategy, hybrid search, latency.

The technical architecture of Brad's retrieval stack:

- **Hybrid BM25 + Vector Retrieval with RRF Fusion:** Keyword search (BM25) runs simultaneously with semantic vector search. Results fuse via Reciprocal Rank Fusion. Consistently outperforms either alone.
- **HyDE (Hypothetical Document Embeddings):** For abstract queries ("what does Brad think about servant leadership"), a flash model generates a hypothetical answer, which is embedded and searched. This is closer in vector space to real answers than the raw question. Graceful degradation design: runs standard search AND HyDE simultaneously, so HyDE helps when correct and is harmless when wrong.
- **Chunking Strategy:** Accounts for 80% of variance in retrieval quality. Brad's implementation strips YAML as metadata, splits at H2/H3 boundaries, uses 15-20% overlap, stores both parent and child chunks.
- **Embedding Model Migration:** Moving from nomic-embed-text (MTEB 62.39, CPU-bound) to Qwen3 Embedding 8B (MTEB 70.58, API-call via OpenRouter). Re-embed cost: ~$0.20 for the entire vault.

The RAG engineer's verdict: PKM and PCM require different retrieval strategies. The vault (PKM) benefits from semantic search with HyDE -- you are looking for conceptual connections across a large knowledge base. Session memory (PCM) benefits from keyword and temporal search -- you are looking for what was discussed yesterday or last week.

**Sharpest questions:**
- Should vault notes and session memories use the same embedding model, or different ones?
- Does chunking strategy need to be different for conversational data (PCM) versus structured notes (PKM)?
- When does retrieval latency become the bottleneck: at 1,000 notes or 10,000?

### Perspective 4: The Organizational Learning Theorist

**What they care about:** Knowledge transfer, organizational memory, the personal-level equivalent of institutional learning.

At the organizational level, knowledge management fails when treat it as a storage problem rather than a conversion problem. The same is true at the personal level.

Brad's SECI spiral at the personal level:
- **Socialization:** Working alongside experienced operators (Navy corpsman field training, DC operations shadowing). The knowledge moves body to body.
- **Externalization:** Writing SOPs, journaling trades, writing after-action reports. What lived in the body becomes a document. The vault is an externalization machine. Every enriched note is externalization.
- **Combination:** Connecting a trading note with a Stoic resilience note with a cognitive bias note to surface a non-obvious connection. This is what AI agents do well -- cross-referencing across domains.
- **Internalization:** Applying documented methodology until it becomes instinct. Retrieval practice re-encodes the explicit note back into deep tacit knowledge.

The critical insight for PKM vs PCM: PKM primarily serves the Combination phase (synthesizing explicit knowledge), while PCM primarily serves the Externalization phase (capturing tacit knowledge from sessions before it is lost). Both are necessary because the SECI cycle cannot skip steps.

**Sharpest questions:**
- Is the PKM system primarily about making tacit knowledge explicit, or about recombining explicit knowledge into new insight?
- Does captured session memory (PCM) convert to vault knowledge (PKM), or do they remain separate stores?

### Perspective 5: The AI Agent Architect

**What they care about:** Context window management, memory architecture, agent state persistence.

Memory in AI systems uses a triple-based representation: Subject-Relation-Value, categorized into nine semantic types (personal, preference, task, relationship, skill, context, instruction, temporal, other). Each type has its own storage policy and decay rules.

The Dual-Horizon Memory Model maps to cognitive science:
- **Short-term:** Atkinson-Shiffrin buffer. 24-hour decay (Ebbinghaus curve). Promotion to long-term requires 3+ accesses.
- **Long-term:** Durable, category-based persistence. Personal, preference, and skill memories enter directly; task and context memories require promotion.

Truth maintenance handles contradictions:
- Entity normalization via Levenshtein distance (0.85 threshold)
- Latest-value-wins for single-valued relations
- Immediate retraction for negation relations

Minsky's K-lines (1986) describe memory as "a recipe for recreating the mental state" -- not stored information. The pgvector RAG system is a crude implementation of K-lines: embeddings capture which concepts were active in a document, and retrieval reactivates similar concepts in the current context.

**Sharpest questions:**
- Is the 24-hour decay threshold for short-term memory calibrated correctly, or should it vary by memory type?
- Should the PKM vault have explicit truth maintenance (like the agent memory system), or is inconsistency acceptable in a personal knowledge base?

### Perspective 6: The Skeptic

**What they care about:** Is the PKM/PCM distinction meaningful, or just two names for the same retrieval problem?

The skeptic's case: both PKM and PCM are retrieval systems. Both take an unstructured input (life, conversations), structure it (notes, memory triples), and retrieve it on demand. The primary difference is time scale and structure:

- PKM: longer time scale, highly structured (tags, links, YAML schema), enriched over time
- PCM: shorter time scale (24hr to session), less structured (conversational), less enriched

If you squint, PCM is just PKM with a faster decay rate and less deliberate organization. The real question is not "which approach" but "how do you build a retrieval system that operates effectively at multiple time scales simultaneously."

The skeptic's verdict: the distinction is useful for design purposes (different storage, different retrieval strategies, different decay rules) but artificial at the architectural level. You need one retrieval system with two horizons, not two separate systems.

---

## Synthesis: The Two-Horizon Personal Retrieval Model

Based on the six perspectives and stress-testing against Brad's lived experience, the evidence supports a unified model with two horizons:

### Horizon 1: Knowledge (PKM) -- The Compounding Layer

**Purpose:** Durable understanding that grows over time.
**Storage:** Structured notes in Obsidian vault with YAML schema, tags, and bidirectional links.
**Retrieval:** Semantic search with HyDE, hybrid BM25 + vector with RRF fusion.
**Decay:** Slow. Notes persist indefinitely. Relevance decays naturally as the knowledge becomes outdated or superseded -- but never disappears.
**Maintenance:** AI seeding + human enrichment. Spaced retrieval practice. Combination across notes by agents.
**Best for:** Conceptual understanding, domain frameworks, decision principles, lessons learned, cross-domain connections.

### Horizon 2: Context (PCM) -- The Situational Layer

**Purpose:** Recent, relevant understanding for the current task.
**Storage:** Hindsight episodic memory (structured triples in PostgreSQL/pgvector) + session search (full-text over past conversations).
**Retrieval:** Keyword and temporal search. Recency-weighted.
**Decay:** Fast. Short-term memories decay after 24 hours unless accessed 3+ times. Session memories persist but lose relevance within days to weeks.
**Maintenance:** Automatic capture from sessions. Hindsight Reflect runs consolidate patterns. Manual promotion to vault when context becomes knowledge.
**Best for:** Recent decisions, project state, what Brad thought about X last week, conversational context chains.

### How They Interact

The SECI spiral maps to the two-horizon model:

1. **Socialization** (tacit to tacit) produces insights during sessions -> captured by PCM.
2. **Externalization** (tacit to explicit) moves session insights to vault notes -> PCM promotes to PKM.
3. **Combination** (explicit to explicit) connects vault notes across domains -> PKM internal process, often triggered by agent queries.
4. **Internalization** (explicit to tacit) re-encodes PKM knowledge through retrieval practice -> PKM feeds back into better socialization.

The horizons are not separate systems. They are layers in a single retrieval architecture, connected by promotion (PCM -> PKM when context proves durable) and reference (PKM -> Horizon 2 when a vault note needs recent context).

---

## Key Findings

1. **PKM and PCM are complementary, not competing.** They operate at different time scales (months/years vs hours/days) and serve different cognitive functions (durable understanding vs situational awareness).

2. **The retrieval stack must be different for each horizon.** PKM benefits from semantic search with HyDE. PCM benefits from keyword and temporal search. One embedding model can serve both, but the retrieval strategy should differ.

3. **The SECI model explains why both are necessary.** The knowledge creation spiral cannot skip steps. PCM captures externalization that would otherwise be lost. PKM enables combination that creates new insight. Neither replaces the other.

4. **Decay rules should differ by horizon.** PKM notes persist indefinitely (knowledge does not expire). PCM memories decay (situational context becomes irrelevant). The 24-hour decay threshold in the Dual-Horizon model is calibrated for conversational memory but would be wrong for vault notes.

5. **The promotion mechanism (PCM -> PKM) is the critical link.** When a session insight proves durable or important, it gets externalized into a vault note. This is the bridge between the two horizons. Without it, valuable context is lost when PCM memory decays.

6. **The skeptic is partially right.** At the architectural level, the distinction is artificial -- it is one retrieval system with two horizons. But at the design level, the distinction matters because storage, retrieval strategy, and decay rules genuinely differ.

---

## Open Questions

1. Should vault notes have explicit truth maintenance (like the agent memory system), or is inconsistency acceptable in a personal knowledge base?
2. What is the optimal promotion threshold from PCM to PKM? Currently it is a manual decision. Could it be automated?
3. Does the embedding model upgrade (nomic to Qwen3) affect PCM and PKM retrieval quality differently?
4. How should the two horizons be weighted when both return results for the same query? Should PKM results always outrank PCM, or should it depend on the query type?

---

## Sources

### Vault Knowledge (PKM Layer)
- Living Wiki Architecture (Brad's Obsidian vault design)
- PKM Knowledge Systems wiki (2000+ notes, Zettelkasten, PARA, CODE framework)
- RAG Architecture notes (hybrid BM25+vector, HyDE, chunking strategy)
- Agent memory governance (triple-based representation, truth maintenance)
- SECI model at personal level (Brad's externalization practice)
- Dual-Horizon Memory Model (Atkinson-Shiffrin applied to AI memory)
- Bjork's desirable difficulties (generation effect, retrieval practice, spacing)

### Session Memory (PCM Layer)
- Hindsight episodic memory system (Retain/Recall/Reflect)
- SECI spiral in practice (Socialization -> Externalization -> Combination -> Internalization)
- Nonaka & Takeuchi knowledge conversion (organizational to personal level)

### Web Research (Validation)
- Hybrid BM25 + vector retrieval (industry standard)
- HyDE query expansion (15-20% improvement on abstract queries)
- HippoRAG (graph-augmented RAG, 20% improvement on multi-hop QA)
- A-MEM NeurIPS 2025 (adaptive memory, self-supervised organization)
- MAGMA (multi-agent graph memory architecture)
- REMem ICLR 2026 (retrieval-augmented episodic memory)
