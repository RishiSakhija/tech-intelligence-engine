# Project Constitution

> **Highest-level source of truth** for Tech Intelligence Engine.
> All product, architecture, and priority decisions must align with this document.

---

## 1. Project Identity

| Attribute | Value |
|-----------|-------|
| **Project Name** | Tech Intelligence Engine |
| **Short Name** | tech-intel |
| **Tagline** | A personalized intelligence engine for discovering, understanding, and tracking the rapidly changing technology ecosystem. |
| **Repository** | github.com/RishiSakhija/tech-intelligence-engine |
| **Primary Language** | Python (backend), TypeScript (frontend) |
| **License** | MIT (planned) |

---

## 2. Product Thesis

**The technology ecosystem—especially AI/ML—evolves faster than any human can track.**

New models, papers, frameworks, releases, and benchmarks appear daily. Existing solutions fail developers and researchers because:

- **Generic search** (Google, Bing) optimizes for web popularity, not technical authority or recency
- **Academic search** (Google Scholar, Semantic Scholar) covers papers but misses code, releases, benchmarks, industry developments
- **Code search** (GitHub, Sourcegraph) finds code but lacks paper/model context, benchmark data, semantic understanding
- **News aggregators** (Hacker News, TLDR, newsletters) are social/curatorial, not systematic or personalized
- **AI chatbots** (Perplexity, Phind, ChatGPT) answer questions but don't *track*, *monitor*, or *connect* entities over time

**We build a vertical intelligence engine** that treats the tech ecosystem as a structured, queryable knowledge graph—papers, models, code, researchers, companies, benchmarks, releases—as interconnected entities with typed relationships, freshness signals, and quality scores.

---

## 3. Problem Statement

### 3.1 Core Problems

1. **Discovery Gap**: Developers miss important developments because they don't know the right query terms or sources
2. **Freshness Gap**: Critical changes (breaking releases, new SOTA benchmarks, security patches) are buried in noise
3. **Context Gap**: A paper, its code, its benchmark results, its author's other work, and related models exist in disconnected silos
4. **Personalization Gap**: Existing tools offer topic follows, not technical interest modeling that actually improves ranking
5. **Verification Gap**: LLM-generated summaries hallucinate; no systematic citation/evidence linking to primary sources

### 3.2 Target Users (Priority Order)

| Tier | User | Primary Job-to-be-Done |
|------|------|------------------------|
| **P0** | **ML Engineers / AI Researchers** | "Track SOTA papers, models, benchmarks in my subfield; know when something beats my baseline" |
| **P1** | **Senior Engineers / Tech Leads** | "Monitor dependency releases, breaking changes, framework updates; assess adoption risk" |
| **P2** | **Engineering Managers / CTOs** | "Understand technology landscape for strategic decisions; track competitor releases" |
| **P3** | **Technical Writers / DevRel** | "Find authoritative sources, examples, benchmarks for documentation" |

**Primary Persona**: *ML Engineer* working on LLM fine-tuning / RAG / agent systems, needs to track: new model releases, benchmark results (MMLU, HumanEval, etc.), framework updates (PyTorch, Transformers, vLLM), relevant papers (arXiv cs.CL, cs.LG), and implementation tricks from GitHub.

---

## 4. Jobs-to-be-Done

| JTBD | Current Workaround | Our Solution |
|------|-------------------|--------------|
| "What new papers this week are relevant to MoE routing?" | arXiv RSS + manual filtering | Personalized discovery feed with semantic relevance + authority scoring |
| "Show me all coding benchmark results for 7B models" | Manual search across Papers with Code, HF leaderboards, blogs | Structured benchmark entity with versioned scores, linked to papers/models |
| "Alert me when Transformers releases breaking changes" | GitHub notifications (too noisy) | Semantic changelog diffing + impact classification |
| "Find researchers working on long-context attention" | Google Scholar + manual profiling | Entity-centric search: researcher → papers → code → collaborations |
| "Compare Llama-3.1-70B vs Qwen2.5-72B on reasoning benchmarks" | Manual table building | Structured model comparison with cited benchmark sources |

---

## 5. Product Principles

| Principle | Description | Implication |
|-----------|-------------|-------------|
| **Authority over popularity** | Rank by technical credibility (citations, reproducibility, maintainer reputation) not engagement | No social signals in core ranking |
| **Freshness as a first-class signal** | Ingestion latency < 1 hour for primary sources; change detection on key entities | Streaming ingestion pipeline; not batch |
| **Entities, not documents** | Search returns structured entities (Model, Paper, Benchmark, Release) with relationships | Knowledge graph backend; not inverted index only |
| **Citations mandatory** | Every claim traceable to primary source; no unattributed LLM synthesis | Evidence-linked output; verification layer |
| **User control over algorithmic opacity** | Personalization explainable; follow/unfollow entities; reset profile | Explicit preference UI; no dark patterns |
| **Vertical depth > horizontal breadth** | Own the tech ecosystem completely before expanding | No general web crawl; curated source list |
| **Deterministic by default, agents where justified** | Pipelines are reproducible; agents only for open-ended research | No agent in critical path without fallback |

---

## 6. Differentiation (Defensible Moats)

| Moat | Why Defensible | Status |
|------|----------------|--------|
| **Curated source graph** | 2+ years to build high-coverage, high-quality source list with access methods, rate limits, parsing logic | Research phase |
| **Entity resolution across sources** | Linking arXiv paper ↔ HF model ↔ GitHub repo ↔ benchmark result requires fuzzy matching + human-in-loop validation | Design phase |
| **Technical fact extraction** | Benchmark scores, model specs, architecture details extracted via specialized pipelines + verification | Not started |
| **Personalized technical ranking** | Interest model trained on implicit (click/dwell) + explicit (follow) signals on technical entities | Not started |
| **Change detection semantics** | Not just "page changed" but "benchmark score improved," "breaking API change," "new model variant released" | Not started |

---

## 7. Non-Goals (Explicitly NOT Building)

| Non-Goal | Reason |
|----------|--------|
| General web search | Commoditized; Google/Bing win |
| Social network / community features | Not core to intelligence; adds moderation burden |
| Newsletter product | Distribution channel, not intelligence |
| Code completion / IDE plugin | Different product; partner instead |
| Model hosting / inference | Infrastructure play; use HF, Replicate, etc. |
| Training / fine-tuning platform | Out of scope |
| Mobile app (Phase 0-3) | Web-first; responsive design sufficient |
| Enterprise SSO / RBAC (Phase 0-2) | Premature; add when paying customers exist |
| Multi-language UI (Phase 0-3) | English-first; tech ecosystem is English-dominant |

---

## 8. Trust Principles

1. **Source transparency**: Every result shows source, retrieval date, confidence
2. **No hallucination tolerance**: LLM output always grounded in retrieved evidence; unverified claims flagged
3. **Reproducibility**: Same query + same index = same results (modulo personalization)
4. **Bias acknowledgment**: Document source coverage gaps (e.g., Chinese-language sources underrepresented)
5. **Correction mechanism**: Users can flag errors; corrections propagate to index

---

## 9. Freshness Principles

| Source Type | Target Latency | Method |
|-------------|----------------|--------|
| ArXiv new submissions | < 30 min | API polling + webhook |
| GitHub releases/tags | < 15 min | Webhook + API fallback |
| Hugging Face model cards | < 1 hour | API polling |
| Company blogs / changelogs | < 2 hours | RSS + scheduled crawl |
| Benchmark leaderboards | < 4 hours | Scheduled scrape + API |
| Conference proceedings | < 24 hours | Scheduled crawl |

**Stale threshold**: Documents not updated in 90 days flagged for re-verification.

---

## 10. Personalization Principles

1. **Explicit > implicit**: Follow actions weigh 10x implicit signals
2. **Entity-level**: Follow researchers, companies, models, repos, topics—not just keywords
3. **Decay**: Interest signals decay with half-life of 30 days unless reinforced
4. **Transparency**: "Why this result?" shows contributing signals
5. **Portability**: Export/import interest profile (JSON)
6. **Reset**: One-click "forget my history"

---

## 11. AI/Agent Principles

| Principle | Rationale |
|-----------|-----------|
| Agents propose, humans dispose | Research agents produce *draft* answers; user verifies |
| Bounded tools only | Agents cannot: write to production DB, make external API calls beyond whitelisted sources, execute arbitrary code |
| Deterministic fallback | Every agent task has a non-agent fallback (e.g., keyword search) |
| Cost control | Per-query token budget; hard limits on agent steps |
| Observability | Every agent step logged: prompt, tool calls, output, latency, cost |

---

## 12. Privacy Principles

1. **Minimal collection**: Only store what improves search/discovery
2. **No third-party sharing**: Interest profiles never sold or shared
3. **Local-first option**: Future: on-device embedding + local index for sensitive workflows
4. **GDPR/CCPA ready**: Delete endpoint from Day 1
5. **No tracking pixels**: No third-party analytics on search results

---

## 13. Technical Constraints

| Constraint | Value | Rationale |
|------------|-------|-----------|
| **Team size** | 1-3 engineers (current) | Architecture must be maintainable by small team |
| **Budget** | <$500/mo infrastructure (Phase 0-2) | Self-hosted where possible; avoid managed services |
| **Latency budget** | Search p95 < 500ms | User-facing; impacts perceived quality |
| **Ingestion throughput** | 10k docs/hour (Phase 1) | Must handle ArXiv daily volume (~1k) + GitHub releases |
| **Storage** | < 10TB (Phase 1-2) | PostgreSQL + ES + Vector DB on single node initially |
| **Language** | Python 3.11+ / TypeScript 5+ | Team expertise; ecosystem maturity |

---

## 14. MVP Boundaries

### IN SCOPE (MVP - Phase 2-3)
- [ ] Hybrid search (lexical + semantic) over 5+ primary sources
- [ ] Basic ranking: recency + authority + BM25 + vector similarity
- [ ] Search API + minimal React UI
- [ ] Source attribution on every result
- [ ] Basic follow: topics, companies, researchers
- [ ] Evaluation benchmark with 100+ labeled queries

### OUT OF SCOPE (Post-MVP)
- Agentic deep research
- Change detection / diffing
- Benchmark entity with versioned scores
- Cross-source entity resolution
- Personalized ranking (beyond topic boost)
- Alerting / notifications
- Multi-modal search (images, video)

---

## 15. Success Criteria (Phase 0-3)

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Index coverage** | >500k technical documents | ArXiv (CS), GitHub (top 10k repos), HF models, major docs |
| **Search latency (p95)** | <500ms | API load test |
| **Relevance (nDCG@10)** | >0.65 vs. human judgments | Labeled eval set |
| **Freshness (median)** | <2 hours for primary sources | Ingestion pipeline monitoring |
| **Source attribution** | 100% of results | Automated check |
| **Zero hallucinated citations** | 0 in eval set | Manual audit |

---

## 16. Decision-Making Rules

| Decision Type | Process |
|---------------|---------|
| **Architecture choices** | ADR in `docs/decisions/` with alternatives, rationale, consequences |
| **Source addition** | Document in `DATA_SOURCE_STRATEGY.md` with access method, licensing, quality assessment |
| **Feature prioritization** | P0/P1/P2/P3 in `REQUIREMENTS.md`; P0 only in MVP |
| **Technology adoption** | Must have: 1) clear need, 2) evaluated 2+ alternatives, 3) migration path documented |
| **Scope creep** | Any new feature requires: user story, success metric, maintenance owner |

---

## 17. Assumptions vs. Validated Findings

| Area | Assumption | Validation Status |
|------|------------|-------------------|
| **ArXiv API reliability** | Stable, no auth needed, ~1k CS papers/day | ✅ Validated (research) |
| **GitHub API rate limits** | 5000/hr sufficient with conditional requests | ✅ Validated |
| **HF Hub API coverage** | Models, datasets, papers, spaces all accessible | ✅ Validated |
| **Semantic Scholar API** | Free tier sufficient for paper metadata | ⚠️ Needs verification (rate limits) |
| **Vector search quality** | bge-large-en-v1.5 sufficient for technical docs | ⚠️ Needs benchmarking |
| **Reranker necessity** | Cross-encoder adds >10% nDCG | ⚠️ Needs experiment |
| **Entity resolution feasibility** | Fuzzy matching + LLM verification achieves >90% precision | ❌ Unvalidated (high risk) |
| **Personalization lift** | Entity follows improve nDCG@10 by >15% | ❌ Unvalidated |
| **Agentic research value** | Users pay for / regularly use deep research | ❌ Unvalidated |

---

## 18. Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-10-06 | Rishi Sakhija | Initial constitution |

---

*This document is the constitutional authority. Changes require ADR and update to this file.*