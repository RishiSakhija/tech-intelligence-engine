# Requirements

> Prioritized functional and non-functional requirements for Tech Intelligence Engine.

---

## Priority System

| Priority | Label | Description |
|----------|-------|-------------|
| **P0** | Essential | Must have for MVP; blocks launch if missing |
| **P1** | Important | Core value proposition; ship soon after MVP |
| **P2** | Later | Significant value; can defer to post-MVP phases |
| **P3** | Deferred | Nice to have; explicitly not in current roadmap |

---

## 1. Functional Requirements

### 1.1 Ingestion & Data Foundation

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| FR-ING-001 | Ingest ArXiv CS papers (cs.AI, cs.CL, cs.LG, cs.CV, cs.LG, cs.SE, etc.) via API | P0 | ~1k/day; incremental polling |
| FR-ING-002 | Ingest GitHub releases, tags, commits for top 10k starred repos | P0 | Webhook + API fallback; focus on ML/AI repos |
| FR-ING-003 | Ingest Hugging Face Hub: models, datasets, papers, spaces metadata | P0 | API pagination; daily full sync + incremental |
| FR-ING-004 | Ingest RSS/Atom feeds from major tech blogs (HF blog, Google AI, Meta AI, etc.) | P0 | 50+ sources; configurable |
| FR-ING-005 | Ingest Papers with Code leaderboards (benchmarks, tasks, datasets) | P1 | Structured benchmark data |
| FR-ING-006 | Ingest Semantic Scholar paper metadata (citations, authors, references) | P1 | API key required; rate limited |
| FR-ING-007 | Ingest Crossref DOI metadata for publications | P2 | Supplemental |
| FR-ING-008 | Ingest conference proceedings (NeurIPS, ICML, ICLR, ACL, CVPR) | P2 | HTML scraping + PDF parsing |
| FR-ING-009 | Normalize all sources to unified document schema | P0 | See `DATA_MODEL.md` |
| FR-ING-010 | Deduplicate documents across sources (same paper on ArXiv + HF + Semantic Scholar) | P0 | Fuzzy title/author matching + DOI |
| FR-ING-011 | Extract structured metadata: authors, affiliations, model names, benchmark scores | P1 | NER + LLM extraction + verification |
| FR-ING-012 | Compute document quality score (authority, citations, reproducibility signals) | P1 | Heuristic + ML |
| FR-ING-013 | Incremental updates: detect changed documents, re-index only deltas | P1 | Change hash + last-modified |
| FR-ING-014 | Dead letter queue for failed ingestions with retry/alerting | P1 | Operational requirement |

### 1.2 Search & Retrieval

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| FR-SRCH-001 | Hybrid search: BM25 (lexical) + dense vector (semantic) + metadata filters | P0 | Elasticsearch + Vector DB |
| FR-SRCH-002 | Query understanding: intent classification (lookup, comparison, exploratory, how-to) | P1 | Rule-based + lightweight classifier |
| FR-SRCH-003 | Entity-aware query parsing: extract model names, benchmarks, researchers, companies | P1 | Gazetteer + NER |
| FR-SRCH-004 | Metadata filtering facets: source type, date, author, org, model, task, benchmark, license | P0 | Post-filter on ES |
| FR-SRCH-005 | Recency boosting: configurable decay function (exponential, half-life) | P0 | Freshness principle |
| FR-SRCH-006 | Authority boosting: citation count, repo stars, author h-index, source tier | P1 | Pre-computed scores |
| FR-SRCH-007 | Cross-encoder reranking for top-K (K=50) results | P1 | bge-reranker-large or similar |
| FR-SRCH-008 | Result diversification: MMR or cluster-based to avoid duplicate entities | P1 | Same paper from multiple sources |
| FR-SRCH-009 | "Why this result?" explanation: show contributing signals per result | P1 | Transparency principle |
| FR-SRCH-010 | Search autocomplete/suggestions for entities (models, researchers, benchmarks) | P2 | Prefix index on entity names |
| FR-SRCH-011 | Saved searches with optional alerting | P2 | Background job |
| FR-SRCH-012 | Search analytics logging (query, results shown, clicks, dwell) | P1 | For implicit personalization |

### 1.3 Personalization & Discovery

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| FR-PERS-001 | Follow system: users can follow entities (researchers, companies, models, repos, topics, benchmarks) | P0 | Explicit personalization |
| FR-PERS-002 | Personalized briefing feed: changes to followed entities since last visit | P0 | Core discovery loop |
| FR-PERS-003 | Implicit interest model: learn from clicks, dwell, saves, research activity | P1 | Vector + graph profile |
| FR-PERS-004 | Personalized ranking: boost results matching user interest profile | P1 | Linear combination with base rank |
| FR-PERS-005 | "Why recommended?" explanation for discovery items | P1 | Transparency |
| FR-PERS-006 | Interest profile export/import (JSON) | P2 | Portability principle |
| FR-PERS-007 | One-click "Reset my personalization" | P1 | User control |
| FR-PERS-008 | Trending in my topics: velocity-ranked items in followed areas | P1 | Discovery |
| FR-PERS-009 | Serendipity slot: 1-2 high-quality items outside follows but semantically close | P2 | Exploration |

### 1.4 Change Detection & Tracking

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| FR-TRCK-001 | Detect new versions/releases for followed models, repos, packages | P1 | Semantic version parsing |
| FR-TRCK-002 | Changelog diffing: extract breaking changes, new features, bug fixes | P2 | LLM + structured parsing |
| FR-TRCK-003 | Benchmark score tracking: detect new SOTA, new submissions on followed benchmarks | P2 | Structured benchmark entity |
| FR-TRCK-004 | Paper version tracking: detect v2, v3, errata, retractions | P2 | ArXiv version history |
| FR-TRCK-005 | Alert system: push/email for high-priority changes (configurable per entity) | P2 | Post-MVP |

### 1.5 Agentic Deep Research

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| FR-RES-001 | Research planner: decompose complex question into sub-questions + source plan | P2 | Phase 6 |
| FR-RES-002 | Source research agents: parallel search across sources for sub-questions | P2 | Bounded tools |
| FR-RES-003 | Entity resolution across sources: normalize same model/paper/researcher | P2 | Critical for synthesis |
| FR-RES-004 | Synthesis agent: build structured output (tables, comparisons, narratives) | P2 | Citation-linked |
| FR-RES-005 | Verification agent: check citations resolve, flag unverified claims | P2 | Quality gate |
| FR-RES-006 | Research report output: executive summary + evidence pack + gaps | P2 | Exportable |
| FR-RES-007 | Cost/step limits per research task (configurable) | P2 | Cost control |

### 1.6 User Management & Settings

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| FR-USER-001 | User registration/authentication (email/password + OAuth: GitHub, Google) | P1 | Post-MVP |
| FR-USER-002 | User settings: profile, follows, notification preferences, privacy | P1 | |
| FR-USER-003 | API key management for programmatic access | P2 | Phase 3+ |
| FR-USER-004 | Data export (GDPR): all user data in machine-readable format | P1 | Privacy principle |
| FR-USER-005 | Account deletion with data purge | P1 | Privacy principle |

---

## 2. Non-Functional Requirements

### 2.1 Performance

| ID | Requirement | Priority | Target |
|----|-------------|----------|--------|
| NFR-PERF-001 | Search API p95 latency | P0 | < 500ms |
| NFR-PERF-002 | Search API p99 latency | P0 | < 1000ms |
| NFR-PERF-003 | Briefing feed load time | P0 | < 1000ms |
| NFR-PERF-004 | Ingestion throughput | P1 | > 10k docs/hour |
| NFR-PERF-005 | Ingestion latency (primary sources) | P1 | < 1 hour median |
| NFR-PERF-006 | Research report generation (5-step) | P2 | < 60s |
| NFR-PERF-007 | Concurrent users (Phase 1) | P1 | 100 |
| NFR-PERF-008 | Concurrent users (Phase 3) | P2 | 1000 |

### 2.2 Reliability & Availability

| ID | Requirement | Priority | Target |
|----|-------------|----------|--------|
| NFR-REL-001 | Search API uptime | P0 | 99.9% |
| NFR-REL-002 | Ingestion pipeline: no data loss | P0 | At-least-once delivery |
| NFR-REL-003 | Graceful degradation: search works if vector DB down (lexical only) | P1 | Circuit breakers |
| NFR-REL-004 | Backup/restore for PostgreSQL + ES + Vector DB | P1 | Daily automated |
| NFR-REL-005 | Health checks for all services | P1 | Kubernetes probes |

### 2.3 Scalability

| ID | Requirement | Priority | Target |
|----|-------------|----------|--------|
| NFR-SCALE-001 | Document index size (Phase 1) | P0 | 500k docs |
| NFR-SCALE-002 | Document index size (Phase 3) | P1 | 5M docs |
| NFR-SCALE-003 | Horizontal scaling: stateless API, sharded ES, clustered vector DB | P2 | Phase 3+ |
| NFR-SCALE-004 | Ingestion horizontal scaling (partition by source) | P1 | |

### 2.4 Security

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| NFR-SEC-001 | All secrets in env vars / secret manager (never in code) | P0 | `.env.example` only |
| NFR-SEC-002 | API rate limiting per user/IP | P1 | Token bucket |
| NFR-SEC-003 | Input validation & sanitization on all endpoints | P0 | Pydantic/FastAPI |
| NFR-SEC-004 | Prompt injection protection for agent prompts | P2 | See `SECURITY.md` |
| NFR-SEC-005 | Content safety: filter malicious content from indexed sources | P1 | See `SECURITY.md` |
| NFR-SEC-006 | Audit logging for admin actions | P2 | |
| NFR-SEC-007 | Dependency vulnerability scanning (CI) | P1 | Dependabot/Trivy |

### 2.5 Observability

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| NFR-OBS-001 | Structured logging (JSON) for all services | P0 | |
| NFR-OBS-002 | Distributed tracing (OpenTelemetry) | P1 | Search + ingestion |
| NFR-OBS-003 | Metrics: latency, error rate, throughput, queue depth | P0 | Prometheus |
| NFR-OBS-004 | Alerting on: ingestion lag, search latency spike, error rate > 1% | P1 | PagerDuty/email |
| NFR-OBS-005 | Search quality dashboards (nDCG, coverage, freshness) | P1 | Custom |

### 2.6 Maintainability

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| NFR-MAINT-001 | Test coverage: >80% unit, >60% integration | P1 | |
| NFR-MAINT-002 | CI/CD: automated test, lint, type-check, build, deploy | P1 | GitHub Actions |
| NFR-MAINT-003 | Documentation: API docs (OpenAPI), architecture docs, runbooks | P1 | |
| NFR-MAINT-004 | Database migrations: versioned, reversible, tested | P1 | Alembic |
| NFR-MAINT-005 | Configuration via env vars only (no config files in repo) | P0 | `.env.example` |

---

## 3. MVP Requirements (Phase 2-3)

**Minimum Viable Product = P0 requirements only**

| Area | P0 Requirements |
|------|-----------------|
| **Ingestion** | FR-ING-001, FR-ING-002, FR-ING-003, FR-ING-004, FR-ING-009, FR-ING-010, FR-ING-013 |
| **Search** | FR-SRCH-001, FR-SRCH-004, FR-SRCH-005, FR-SRCH-006 |
| **Personalization** | FR-PERS-001, FR-PERS-002 |
| **UI** | Search page, Briefing feed, Entity detail, Settings (follows) |
| **Infrastructure** | PostgreSQL, Elasticsearch, Vector DB, Redis, API, Frontend |
| **Quality** | NFR-PERF-001/002/003, NFR-REL-001, NFR-SEC-001/003, NFR-OBS-001/003 |

---

## 4. Post-MVP Requirements (Phase 4+)

| Phase | Key Additions |
|-------|---------------|
| **Phase 4: Search Quality** | FR-SRCH-002, FR-SRCH-003, FR-SRCH-007, FR-SRCH-008, FR-SRCH-009, FR-ING-011, FR-ING-012 |
| **Phase 5: Personalized Discovery** | FR-PERS-003, FR-PERS-004, FR-PERS-005, FR-PERS-007, FR-PERS-008 |
| **Phase 6: Tracking & Change Detection** | FR-TRCK-001, FR-TRCK-002, FR-TRCK-003, FR-TRCK-005 |
| **Phase 7: Agentic Research** | FR-RES-001 through FR-RES-007 |
| **Phase 8: Production Hardening** | FR-USER-001 through FR-USER-005, NFR-SCALE-003, NFR-REL-004 |

---

## 5. Future Ideas (P3 - Explicitly Deferred)

| Idea | Why Deferred |
|------|--------------|
| Mobile app (iOS/Android) | Web-first; responsive sufficient |
| Browser extension | Integration, not core product |
| Slack/Discord bots | Distribution channel |
| Newsletter generation | Feature, not platform |
| Code search within repos (not just metadata) | Different product (Sourcegraph) |
| Model playground / inference | Infrastructure; use partners |
| Fine-tuning / training platform | Out of scope |
| Enterprise SSO/SAML/RBAC | Premature; add with paying customers |
| Multi-language UI | English-first; tech is English-dominant |
| Video/content understanding (YouTube, talks) | High effort, lower precision |
| Patent search | Different domain |

---

## 6. Traceability Matrix

| Requirement | Constitution Principle | Product Vision Workflow | Architecture Component |
|-------------|------------------------|-------------------------|------------------------|
| FR-ING-001..004 | Freshness, Authority | Briefing feed | Ingestion Pipeline |
| FR-ING-009..010 | Entities not documents | Entity cards | Normalization |
| FR-SRCH-001..006 | Hybrid retrieval, Authority | Search experience | Search Service |
| FR-SRCH-007 | Quality over speed | Relevance | Reranker Service |
| FR-PERS-001..002 | Explicit > implicit | Follow system | Personalization Service |
| FR-PERS-003..004 | User control | Interest model | Personalization Service |
| FR-TRCK-001..003 | Freshness, Change detection | What changed | Tracking Service |
| FR-RES-001..006 | Agents propose, humans dispose | Deep research | Research Service |
| NFR-SEC-001..005 | Trust, Privacy | - | All services |

---

## 7. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-10-06 | Initial requirements |

---

*Requirements are living documents. Changes require ADR and update to this file.*