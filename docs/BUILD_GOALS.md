# Build Goals

> Incremental engineering strategy for Tech Intelligence Engine.
> Each goal leaves the architecture in a better validated state than before.

---

## Architectural Rule

> **Every small goal must leave the architecture in a better validated state than before.**

```text
Build
→ Test
→ Measure
→ Review
→ Document
→ Lock
→ Next goal
```

An architectural decision should only be marked **Accepted** when supported by evidence or a clear project constraint.

---

## Goal 0 — Foundation Locked

**Status**: Complete

Repository, product definition, architecture documents, ADRs.

**Output**: Complete documentation set (13 core docs + 10 ADRs), initialized repo with `.gitignore`, `.env.example`, professional README.

---

## Goal 1 — First Real Data Slice

**Target**:
- One source (ArXiv)
- Small corpus (~1,000 papers)
- Normalized schema
- PostgreSQL storage
- Reproducible ingestion

**Why now**: Validates the ingestion pipeline end-to-end with real data before adding complexity.

**Prerequisites**: Goal 0 complete.

**Output**: Working ingestion script that fetches, normalizes, and stores ArXiv papers in PostgreSQL.

**Success criteria**:
- A fresh developer can run ingestion locally and inspect stored records
- Ingestion completes without errors
- Schema matches `DATA_MODEL.md`
- Ingestion is idempotent (re-running produces same results)

**What NOT to build yet**:
- Multiple sources
- Vector embeddings
- Semantic search
- Deduplication across sources
- Enrichment agents

---

## Goal 2 — First Search

**Target**:
- Lexical search (BM25) over Goal 1 corpus
- Basic API (FastAPI)
- Meaningful results for technical queries

**Why now**: Validates the search path with real data before adding semantic retrieval.

**Prerequisites**: Goal 1 complete.

**Output**: Search API that returns relevant results for queries like "transformer attention" or "Llama 3.1".

**Success criteria**:
- Technical queries return sensible results
- API responds in < 500ms p95
- Results show source attribution
- Basic metadata filtering works (date, source)

**What NOT to build yet**:
- Semantic/vector search
- Hybrid fusion
- Reranking
- Personalization
- UI

---

## Goal 3 — Semantic Search

**Target**:
- Embeddings using local model (BAAI/bge-large-en-v1.5 or alternative)
- Vector search (Qdrant or OpenSearch)
- Local model execution

**Why now**: Validates semantic retrieval with local models before hybrid fusion.

**Prerequisites**: Goal 2 complete.

**Output**: Vector search returning conceptually relevant documents for queries like "papers about mixture of experts routing".

**Success criteria**:
- Semantic queries retrieve conceptually relevant documents
- Local embedding model runs without paid APIs
- Vector search latency < 200ms p95
- Embeddings stored and retrieved correctly

**What NOT to build yet**:
- Hybrid fusion (RRF)
- Cross-encoder reranking
- Query understanding
- Personalization

---

## Goal 4 — Hybrid Search

**Target**:
- Lexical + semantic retrieval
- Reciprocal Rank Fusion (RRF) with k=60

**Why now**: Combines validated lexical and semantic retrievers; RRF is parameter-free and robust.

**Prerequisites**: Goals 2 and 3 complete.

**Output**: Hybrid search API combining BM25 + vector results via RRF.

**Success criteria**:
- Hybrid results outperform either retriever alone on evaluation set
- RRF fusion adds < 50ms latency
- Results are explainable ("Why this result?")

**What NOT to build yet**:
- Cross-encoder reranking
- Query understanding/intent classification
- Personalization
- Metadata filtering beyond basics

---

## Goal 5 — Ranking

**Target**:
- Relevance signals (BM25, vector similarity)
- Freshness signals (exponential decay, configurable half-life)
- Authority signals (source quality, citations, stars)
- Deterministic scoring function

**Why now**: Ranking combines validated signals into a single explainable score.

**Prerequisites**: Goal 4 complete.

**Output**: Ranking function with documented weights and explainable output.

**Success criteria**:
- Ranking improves measured relevance (nDCG@10) over raw retrieval
- All signals documented with weights
- "Why this result?" shows per-signal contribution
- Deterministic: same query + same index = same results

**What NOT to build yet**:
- Cross-encoder reranking
- Personalization boost
- Query understanding/intent classification
- Learning-to-rank

---

## Goal 6 — Minimal UI

**Target**:
- Search box (Next.js)
- Results display with entity cards
- Source information
- Basic filters (source, date, entity type)

**Why now**: Enables end-to-end usage and manual evaluation.

**Prerequisites**: Goal 5 complete.

**Output**: Working web UI for searching the corpus.

**Success criteria**:
- The engine can be used end-to-end by a human
- Results show structured entity cards (not snippets)
- Source attribution visible on every result
- Basic filters work
- Mobile responsive

**What NOT to build yet**:
- Briefing feed / discovery
- Follow system
- Research workspace
- Settings / personalization UI
- Dark mode

---

## Goal 7 — Evaluation

**Target**:
- Labeled query set (gold set)
- nDCG@10, MRR@10, Precision@5
- Latency measurements (p50, p95, p99)
- Freshness metrics

**Why now**: Makes search quality measurable; enables regression detection.

**Prerequisites**: Goal 6 complete.

**Output**: Evaluation pipeline running on gold set with published metrics.

**Success criteria**:
- 50+ labeled queries across intent types (lookup, comparative, exploratory)
- nDCG@10 baseline established
- Nightly evaluation runs automatically
- Regression alerts on >5% metric drop
- Latency percentiles tracked

**What NOT to build yet**:
- A/B testing framework
- Personalization evaluation
- Research quality evaluation
- Large gold set (200+ queries) — start with 50

---

## Goal 8 — Additional Sources

**Target**:
- Add GitHub (releases, webhooks)
- Add Hugging Face Hub (models, papers)
- Add RSS feeds (tech blogs)
- Incremental, one at a time

**Why now**: Source diversity validated after core search quality is established.

**Prerequisites**: Goal 7 complete.

**Output**: Multi-source corpus with consistent schema.

**Success criteria**:
- Each new source ingests without pipeline changes
- Deduplication works across sources (exact keys first)
- Source quality scores computed
- Ingestion lag < 1 hour for Tier 1 sources

**What NOT to build yet**:
- Fuzzy deduplication across sources
- Semantic Scholar / Crossref (rate-limited, commercial terms)
- All 20+ blog feeds at once
- Full historical backfill

---

## Goal 9 — Entity Resolution

**Target**:
- Exact identifier matching (DOI, ArXiv ID, HF Model ID, GitHub owner/repo)
- Canonical entity registry with aliases

**Why now**: Entity resolution starts with high-confidence exact keys; fuzzy matching deferred.

**Prerequisites**: Goal 8 complete.

**Output**: Entity registry linking same real-world objects across sources.

**Success criteria**:
- Exact key matches resolve > 95% of cross-source duplicates
- Entity aliases table populated with confidence scores
- Human review queue for ambiguous cases (not built yet)

**What NOT to build yet**:
- Fuzzy matching (title + authors + year)
- LLM-assisted verification
- Human review UI
- Cross-source entity resolution at scale

---

## Goal 10 — Personalization

**Target**:
- Explicit follows (entity-level: researchers, companies, models, repos, benchmarks, topics)
- Follow API + UI
- Basic explicit boost in ranking

**Why now**: Explicit follows are high-signal, user-controlled, and validate the personalization path before implicit modeling.

**Prerequisites**: Goal 8 complete.

**Output**: Follow system with ranking influence.

**Success criteria**:
- Users can follow/unfollow 6 entity types
- Explicit boost improves relevance for followed entities
- "Why this result?" shows personalization contribution
- Export/import follows (JSON)

**What NOT to build yet**:
- Implicit interest vectors
- Serendipity recommendations
- Trending in your topics
- Cold-start onboarding flow

---

## Goal 11 — Change Detection

**Target**:
- Version tracking for models, repositories, papers
- Semantic diff for releases (breaking/new/deprecated/benchmarks)
- Change log per entity

**Why now**: Only after reliable version/history data exists from multiple ingestion cycles.

**Prerequisites**: Goal 8 complete (multiple ingestion runs), Goal 9 complete (entity resolution).

**Output**: "What changed" view for followed entities.

**Success criteria**:
- New model versions detected and diffed
- Breaking changes identified in changelogs
- Change history queryable per entity
- Briefing feed shows changes since last visit

**What NOT to build yet**:
- Benchmark SOTA tracking
- Alerting/notifications
- Paper version detection (ArXiv v1→v2)
- Company-level change aggregation

---

## Goal 12 — Agentic Research

**Target**:
- Research planner (decompose question)
- Source research agents (parallel search)
- Synthesis agent (tables, narratives)
- Verification agent (citations, confidence)

**Why now**: Only after the underlying retrieval and evidence system is strong. Agents depend on good search + evidence.

**Prerequisites**: Goals 4, 7, 9, 11 complete (search quality measurable, entities resolved, evidence available).

**Output**: Deep research workflow producing cited reports.

**Success criteria**:
- 5-step research completes in < 60s
- Citation accuracy > 95%
- Hallucination rate < 5%
- Cost per research < $0.50 (local models)
- Human evaluation > 4.0/5.0

**What NOT to build yet**:
- Complex multi-agent orchestration frameworks
- Evaluation agent
- Automated fact-checking beyond citations
- Research workspace UI (minimal only)

---

## Cross-Goal Principles

| Principle | Application |
|-----------|-------------|
| **Evidence over intuition** | Each goal requires measurable success criteria |
| **Local-first** | No paid APIs; local models; open-source only |
| **Replaceable dependencies** | External services abstracted behind interfaces |
| **Deterministic by default** | Agents only where non-determinism adds value |
| **Explainability** | Every result shows "why" |
| **Source transparency** | Every result traceable to primary source |
| **Small increments** | Each goal shippable and testable independently |