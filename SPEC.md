# STORM Content Creator -- Quick Reference

**This is a methodology reference and Hermes skill specification, not runnable code.**

An adaptation of Stanford's STORM method (NAACL 2024) for content creation.

## Pipeline

Phase 1: Perspective Discovery (AI maps 6-8 expert viewpoints, human approves)
Phase 2: Human Expert Interview (AI researches via three-layer search, human stress-tests)
Phase 3: Curate and Outline (living, evolving structure)
Phase 4: Grounded Writing (every claim sourced, "needs more research" over padding)
Phase 5: Moderator Pass (audit for source bias and false connections -- highest-leverage role)

## Model Map

Fast model (DeepSeek 4 Flash): interviews, perspective discovery, polish
Strong model (Owl Alpha / Sonnet 4.6): outline, writing, moderator

## Search Stack

web_search: internet facts and current data
RAG: personal vault knowledge and cross-domain connections
session_search: prior thinking (run first to avoid redundancy)

## When to Use

Full STORM (all 5 phases): cornerstone content, major research pieces
Lightweight (Phases 1, 3, 5): regular posts, time-sensitive pieces

## License

MIT

## Credits

Based on STORM by Shao et al., Stanford Oval Lab, NAACL 2024.
Co-STORM collaborative extension, EMNLP 2024.
Reference implementation: github.com/stanford-oval/storm
