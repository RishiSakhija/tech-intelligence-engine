# Roadmap

> Phased delivery plan for Tech Intelligence Engine. Adjusts based on learning; dates are directional.

---

## $0 Development Constraint

Development and MVP validation must be achievable with a **$0 infrastructure/API budget** using open-source software, local execution, and genuinely free public APIs/services where available.

- Paid APIs are **NOT required** for core development
- Paid services may be considered only in a future production phase
- External free services are **replaceable dependencies**
- Architecture must remain **local-first**
- No vendor should become a **hard dependency** for the MVP
- Do NOT claim free tiers are permanent or unlimited

---

## Incremental Scale Targets

The project intentionally starts small. Scale increases only after correctness, search quality, ingestion reliability, resource usage, and evaluation results justify it.

| Goal | Target | When |
|------|--------|------|
| **Goal 01** | 1,000 useful documents | Goal 1 |
| **Goal 02** | 10,000 useful documents | Goal 2 |
| **Goal 03** | 50,000 useful documents | Goal 3 |
| **Goal 04** | Expand sources | Goal 8+ |
| **Goal 05** | Evaluate whether larger scale is necessary | After Goal 8 |

**Do NOT promise 500k documents as an early requirement.**

---

## 1. Phase Overview

| Phase | Name | Focus | Duration | Key Deliverable |
|-------|------|-------|----------|-----------------|
| **0** | Foundation & Research | Product definition, architecture, source strategy, evaluation design | 4-6 weeks | This documentation set |
| **1** | Data Foundation | Ingestion pipeline, core sources, normalization, deduplication, basic indexing | 6-8 weeks | 1,000+ docs indexed; searchable |
| **2** | Search MVP | Hybrid search, basic ranking, search API, minimal UI | 4-6 weeks | Working search engine |
| **3** | Search Quality | Query understanding, reranking, evaluation, relevance tuning | 6-8 weeks | nDCG@10 > 0.65 |
| **4** | Personalized Discovery | Follow system, briefing feed, implicit personalization, trending | 6-8 weeks | Daily active usage |
| **5** | Tracking & Change Detection | Release diffing, benchmark tracking, alerts | 6-8 weeks | "What changed" working |
| **6** | Agentic Research | Research planner, source agents, synthesis, verification | 8-12 weeks | Deep research reports |
| **7** | Production Hardening | Auth, scaling, observability, security, API, enterprise features | Ongoing | Production-ready |

---

## 2. Phase 0: Foundation & Research (Current)

**Goal**: Evidence-driven foundation before any implementation.

| Task | Status | Owner | Dependencies |
|------|--------|-------|--------------|
| Project Constitution | ✅ Done | - | - |
| Product Vision | ✅ Done | - | Constitution |
| Requirements (P0-P3) | ✅ Done | - | Vision |
| Competitor Analysis | ✅ Done | - | Research |
| Data Source Strategy | ✅ Done | - | Competitor Analysis |
| Search Strategy | ✅ Done | - | Requirements |
| Personalization Design | ✅ Done | - | Requirements |
| Agent Architecture | ✅ Done | - | Requirements |
| System Architecture | ✅ Done | - | All above |
| Data Model | ✅ Done | - | System Architecture |
| Security Requirements | ✅ Done | - | System Architecture |
| Evaluation Framework | ✅ Done | - | Requirements |
| Roadmap | ✅ Done | - | All above |
| **Cross-check & Consistency Audit** | 🔄 In Progress | - | All docs |
| **Initial ADRs** | 🔄 Pending | - | Architecture decisions |
| **Repository Setup** | ✅ Done | - | - |

**Exit Criteria**: All docs reviewed, consistent, ADRs for major decisions, team alignment.

---

## 3. Phase 1: Data Foundation

**Goal**: Reliable, fresh, deduplicated index of core technical sources.

### 3.1 Ingestion Pipeline (Core)

| Task | Priority | Effort | Notes |
|------|----------|--------|-------|
| Project scaffolding (FastAPI, PostgreSQL, Redis, Celery) | P0 | 1 week | Cookiecutter / template |
| ArXiv connector (API + RSS) | P0 | 1 week | Incremental polling |
| GitHub connector (Releases + Webhooks) | P0 | 1.5 weeks | GraphQL + REST; webhook receiver |
| HF Hub connector (Models + Papers + Webhooks) | P0 | 1 week | `huggingface_hub` lib |
| RSS/Atom connector (Blogs) | P0 | 0.5 weeks | Generic feed parser |
| Normalization to unified schema | P0 | 1 week | See `DATA_MODEL.md` |
| Exact deduplication (DOI, ArXiv ID, HF ID, GitHub) | P0 | 1 week | High confidence |
| Fuzzy deduplication queue (LLM-assisted) | P1 | 1 week | Human review UI |
| Quality scoring (authority, freshness, completeness) | P1 | 0.5 weeks | Heuristic v1 |
| PostgreSQL schema + migrations | P0 | 0.5 weeks | Alembic |
| Elasticsearch index + mappings | P0 | 0.5 weeks | See `DATA_MODEL.md` |
| Vector DB (Qdrant) collections + embedding pipeline | P0 | 1 week | bge-large-en-v1.5 |
| Ingestion monitoring (lag, errors, dead letter) | P1 | 0.5 weeks | Grafana dashboards |
| Backfill historical data (ArXiv 2020+, top GitHub repos) | P1 | 1 week | Rate limit management |

### 3.2 Source Onboarding Order

| Week | Sources | Target Volume |
|------|---------|---------------|
| 1-2 | ArXiv (CS categories, recent only) | ~1,000 papers |
| 2-3 | GitHub (top 100 ML repos + webhooks) | ~500 releases |
| 3-4 | HF Hub (Models + Daily Papers, recent) | ~2,000 models + papers |
| 4-5 | Blogs (5 major RSS feeds) | ~200 posts |
| 5-6 | Papers with Code (API, recent) | ~500 benchmarks |

**Scale increases only after evaluation results justify it.**

### 3.3 Phase 1 Exit Criteria

- [ ] 1,000+ documents indexed across 1-2 sources
- [ ] Ingestion lag < 1 hour for Tier 1 sources
- [ ] Deduplication precision > 95% (manual audit)
- [ ] Basic search works via API (no UI yet)
- [ ] Monitoring dashboards live (lag, volume, errors)
- [ ] Dead letter queue < 1% of throughput

---

## 4. Phase 2: Search MVP

**Goal**: Working hybrid search with basic ranking and minimal UI.

### 4.1 Search Service

| Task | Priority | Effort | Notes |
|------|----------|--------|-------|
| Lexical search (Elasticsearch BM25) | P0 | 0.5 weeks | Technical analyzer |
| Vector search (Qdrant) | P0 | 0.5 weeks | Embedding at index time |
| Reciprocal Rank Fusion (RRF) | P0 | 0.5 weeks | k=60 |
| Metadata filtering (facets) | P0 | 0.5 weeks | ES post-filter |
| Recency boosting (exponential decay) | P0 | 0.5 weeks | Configurable half-life |
| Authority boosting (source quality + citations) | P0 | 0.5 weeks | Pre-computed scores |
| Search API (FastAPI) | P0 | 1 week | OpenAPI spec |
| Query logging (for eval + implicit signals) | P0 | 0.5 weeks | Async to Redis → PostgreSQL |

### 4.2 Minimal Frontend (Next.js)

| Task | Priority | Effort | Notes |
|------|----------|--------|-------|
| Search page (query input, results list) | P0 | 1 week | Server components |
| Entity cards (Paper, Model, Benchmark, Release) | P0 | 1 week | Structured display |
| Faceted sidebar (source, date, entity type) | P0 | 0.5 weeks | URL-synced state |
| "Why this result?" expandable | P1 | 0.5 weeks | Ranking breakdown |
| Entity detail pages | P1 | 1 week | Paper, Model, Benchmark, Repo |
| Responsive design (mobile) | P1 | 0.5 weeks | Tailwind CSS |
| Dark mode | P2 | 0.5 weeks | |

### 4.3 Phase 2 Exit Criteria

- [ ] Hybrid search returns results in < 500ms p95
- [ ] Faceted filtering works
- [ ] Entity cards display structured info (not just snippets)
- [ ] Source attribution on every result
- [ ] Basic UI usable for daily searches
- [ ] Evaluation pipeline runs nightly on gold set

---

## 5. Phase 3: Search Quality

**Goal**: Measurably better relevance through query understanding, reranking, and tuning.

### 5.1 Query Understanding

| Task | Priority | Effort |
|------|----------|--------|
| Intent classifier (rule-based + lightweight ML) | P0 | 1 week |
| Entity extraction from queries (gazetteer + NER) | P0 | 1 week |
| Query expansion (synonyms, acronyms, model families) | P1 | 0.5 weeks |
| Ambiguity detection ("Did you mean?") | P1 | 0.5 weeks |

### 5.2 Reranking

| Task | Priority | Effort |
|------|----------|--------|
| Cross-encoder integration (bge-reranker-large) | P0 | 1 week |
| Batch inference optimization (GPU) | P0 | 0.5 weeks |
| Rerank top-50 → final top-20 | P0 | 0.5 weeks |
| Fallback path (reranker timeout → base rank) | P0 | 0.5 weeks |

### 5.3 Result Quality

| Task | Priority | Effort |
|------|----------|--------|
| Result diversification (MMR) | P1 | 0.5 weeks |
| Deduplication at result level (same entity) | P1 | 0.5 weeks |
| "Why this result?" per-result breakdown | P1 | 0.5 weeks |
| Benchmark search mode (structured tables) | P1 | 1 week |
| Model comparison mode | P2 | 1 week |

### 5.4 Evaluation & Tuning

| Task | Priority | Effort |
|------|----------|--------|
| Gold set expansion (200 queries) | P0 | Ongoing |
| LTR feature engineering (if needed) | P2 | 2 weeks |
| Hyperparameter tuning (weights, half-lives) | P1 | 1 week |
| A/B framework for ranking experiments | P1 | 1 week |

### 5.5 Phase 3 Exit Criteria

- [ ] nDCG@10 > 0.65 on gold set
- [ ] Reranker improves nDCG by >10% over base
- [ ] Query understanding handles 80% of query types
- [ ] Benchmark comparison tables render correctly
- [ ] No regressions on latency/freshness

---

## 6. Phase 4: Personalized Discovery

**Goal**: Daily briefing feed that users actually open.

### 6.1 Follow System

| Task | Priority | Effort |
|------|----------|--------|
| Follow/unfollow API + UI | P0 | 1 week |
| Followable entity types (6 types) | P0 | 0.5 weeks |
| Notification preferences per entity | P1 | 0.5 weeks |
| Follow management page | P1 | 0.5 weeks |

### 6.2 Briefing Feed

| Task | Priority | Effort |
|------|----------|--------|
| Change detection for followed entities | P0 | 1 week |
| Feed generation (explicit + implicit + serendipity) | P0 | 1 week |
| Feed caching (Redis, 15-min refresh) | P0 | 0.5 weeks |
| Feed UI (homepage) | P0 | 1 week |
| "Since your last visit" timestamp tracking | P0 | 0.5 weeks |

### 6.3 Implicit Personalization

| Task | Priority | Effort |
|------|----------|--------|
| Interest vector computation (online + batch) | P0 | 1 week |
| Personalized ranking boost | P0 | 0.5 weeks |
| "Trending in your topics" | P1 | 0.5 weeks |
| Serendipity slot | P1 | 0.5 weeks |
| Interest profile UI (transparency) | P1 | 0.5 weeks |
| Export/Import / Reset controls | P1 | 0.5 weeks |

### 6.4 Phase 4 Exit Criteria

- [ ] Briefing feed loads < 1s
- [ ] Follow system covers all entity types
- [ ] Personalization lift > 15% in A/B test
- [ ] User retention (7-day) > 30%
- [ ] Privacy controls (reset, export, delete) work

---

## 7. Phase 5: Tracking & Change Detection

**Goal**: Semantic "what changed" for releases, benchmarks, models.

### 7.1 Change Detection

| Task | Priority | Effort |
|------|----------|--------|
| Release changelog parser (structured diff) | P0 | 1.5 weeks |
| Model card diff (HF) | P0 | 1 week |
| Paper version detection (ArXiv v1→v2) | P1 | 0.5 weeks |
| Benchmark score tracking (new SOTA) | P1 | 1 week |
| Change significance classifier (breaking/major/minor) | P1 | 1 week |

### 7.2 Diff UI

| Task | Priority | Effort |
|------|----------|--------|
| Semantic diff view (breaking, new, deprecated) | P0 | 1 week |
| Version selector for entities | P1 | 0.5 weeks |
| Change history timeline | P1 | 0.5 weeks |

### 7.3 Alerts

| Task | Priority | Effort |
|------|----------|--------|
| Alert preferences per entity | P1 | 0.5 weeks |
| Email/push notification delivery | P1 | 1 week |
| Digest emails (daily/weekly) | P2 | 0.5 weeks |

### 7.4 Phase 5 Exit Criteria

- [ ] Release diffs show breaking changes accurately (>90% precision)
- [ ] Benchmark SOTA changes detected within 24h
- [ ] Alerts deliver within 1 hour of detection
- [ ] Users can configure granular notifications

---

## 8. Phase 6: Agentic Research

**Goal**: Deep research reports with citations for complex questions.

### 8.1 Agent Framework

| Task | Priority | Effort |
|------|----------|--------|
| Agent orchestrator (Temporal or custom) | P0 | 2 weeks |
| Research Planner Agent | P0 | 1.5 weeks |
| Source Research Agent (parallel) | P0 | 1.5 weeks |
| Entity Resolution Agent | P0 | 1 week |
| Synthesis Agent (tables, narratives) | P0 | 1.5 weeks |
| Verification Agent (citations, claims) | P0 | 1 week |
| Cost/step limiting + observability | P0 | 0.5 weeks |

### 8.2 Research UX

| Task | Priority | Effort |
|------|----------|--------|
| Research workspace UI | P0 | 1.5 weeks |
| Progress streaming (WebSocket) | P0 | 0.5 weeks |
| Report rendering (tables, citations, gaps) | P0 | 1 week |
| Export (Markdown, PDF, Notion) | P1 | 1 week |
| Follow-up questions | P1 | 0.5 weeks |

### 8.3 Phase 6 Exit Criteria

- [ ] 5-step research completes in < 60s
- [ ] Citation accuracy > 95%
- [ ] Human evaluation > 4.0/5.0 on accuracy/completeness
- [ ] Cost per research < $0.50
- [ ] Hallucination rate < 5%

---

## 9. Phase 7: Production Hardening

**Goal**: Production-ready system with auth, scaling, observability, security.

| Area | Tasks |
|------|-------|
| **Auth** | JWT + OAuth (GitHub, Google); API keys; MFA |
| **Scaling** | K8s deployment; read replicas; cluster ES/Qdrant; Redis cluster |
| **Observability** | OpenTelemetry; distributed tracing; SLO dashboards; alerting |
| **Security** | Penetration test; secret rotation; dependency scanning; WAF |
| **API** | Public API v1; rate limits; SDK (Python/JS); documentation |
| **Compliance** | GDPR delete/export; privacy policy; terms of service |
| **Multi-tenancy** | Organizations; teams; RBAC (if enterprise demand) |

---

## 10. Resource & Timeline Estimate

| Phase | Engineering Weeks | Infrastructure Cost (Monthly) |
|-------|-------------------|-------------------------------|
| 0 | 4-6 (research) | $0 (dev) |
| 1 | 6-8 | $0 (local dev, open-source only) |
| 2 | 4-6 | $0 (local dev) |
| 3 | 6-8 | $0 (local dev, GPU if available) |
| 4 | 6-8 | $0 |
| 5 | 6-8 | $0 |
| 6 | 8-12 | $0 (local models) |
| 7 | Ongoing | Production budget TBD |

**Total to Phase 3 (Search MVP + Quality)**: ~20-28 weeks (5-7 months) at $0 infrastructure cost
**Total to Phase 6 (Full Vision)**: ~40-56 weeks (10-14 months) at $0 infrastructure cost

> All development targets $0 infrastructure cost. GPU usage assumes local hardware or free tier where available.

---

## 11. Risk Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Entity resolution fails at scale | High | High | Start simple (exact keys); defer fuzzy to Phase 3; human-in-loop |
| Reranker latency too high | Medium | High | Batch inference; model distillation; CPU fallback |
| Personalization lift < 5% | Medium | Medium | Stronger explicit signals; better interest vectors; more training data |
| Agent hallucination rate high | High | High | Verification agent; citation grounding; human review for v1 |
| Ingestion lag > 4 hours | Medium | High | Priority queues; horizontal workers; monitoring alerts |
| Semantic Scholar API commercial block | Medium | Medium | OpenAlex fallback; rely on ArXiv + HF + Crossref |
| Team bandwidth (1-2 engineers) | High | High | Ruthless prioritization; P0 only; defer P2/P3 |

---

## 12. Decision Points (Go/No-Go)

| Decision Point | Criteria | If No-Go |
|----------------|----------|----------|
| **End of Phase 1** | 1,000+ docs indexed; lag < 1h; dedupe > 95% | Simplify sources; reduce scope |
| **End of Phase 2** | Search works; latency < 500ms; UI usable | Optimize ES/Vector config; defer UI polish |
| **End of Phase 3** | nDCG@10 > 0.65; reranker helps | Revisit ranking signals; more training data |
| **End of Phase 4** | Personalization lift > 10% in A/B | Strengthen explicit follows; defer implicit |
| **End of Phase 6** | Research quality > 4.0/5.0; cost < $0.50 | Limit scope (no synthesis); use simpler agents |

---

## 13. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-10-06 | Initial roadmap |

---

*This roadmap is a living document. Update after each phase retrospective.*