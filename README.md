# Tech Intelligence Engine

> A personalized intelligence engine for discovering, understanding, and tracking the rapidly changing technology ecosystem.

## Product Thesis

The technology ecosystem—especially AI/ML—evolves faster than any human can track. New models, papers, frameworks, releases, and benchmarks appear daily. Existing solutions are either too broad (Google), too narrow (arXiv RSS), too social (Twitter/X), or too shallow (news aggregators).

**Tech Intelligence Engine** is a vertical search and intelligence platform focused exclusively on the technology ecosystem. It combines:

- **Focused search** over curated, high-authority technical sources
- **Discovery** of relevant developments you didn't know to search for
- **Personalization** that learns your technical interests and ranks accordingly
- **Freshness** prioritizing recent, impactful changes
- **Source quality** weighting authoritative sources over noise
- **Entity relationships** connecting papers, models, code, researchers, companies
- **Change detection** tracking what actually changed in releases, benchmarks, models
- **Technical intelligence** extracting structured facts: benchmarks, architectures, capabilities
- **Deep/agentic research** for complex multi-step technical investigations

## Why This Exists

| Problem | Current Solutions Fail Because |
|---------|-------------------------------|
| "What new transformer papers matter this week?" | arXiv is firehose; no ranking by impact/relevance |
| "Show me all LLM benchmark results for coding" | Benchmarks scattered across papers, blogs, leaderboards |
| "Track changes to PyTorch 2.5 release" | Changelogs buried; no diff/impact analysis |
| "Find researchers working on MoE routing" | No entity-centric search across papers+code+talks |
| "Alert me when my dependencies release breaking changes" | GitHub notifications are noisy; no semantic understanding |

## What Makes It Different

| Dimension | Generic Search | News Aggregators | This Project |
|-----------|----------------|------------------|--------------|
| **Scope** | Entire web | Curated feeds | Tech ecosystem only |
| **Ranking** | PageRank/engagement | Recency/popularity | Technical relevance + authority + personalization |
| **Entities** | Keywords | Topics/tags | Papers, models, researchers, companies, releases, benchmarks |
| **Freshness** | Crawl-dependent | RSS/ping | Multi-source change detection |
| **Output** | Links | Summaries | Structured intelligence: facts, relationships, diffs |
| **Personalization** | History-based | Topic follows | Technical interest modeling + explicit controls |

## Core Capabilities (Planned)

### Search
- Hybrid lexical + semantic retrieval over curated technical corpus
- Metadata filtering: source type, date, author, company, model, benchmark
- Query understanding for technical intent (paper vs. code vs. release vs. benchmark)

### Discovery & Tracking
- Personalized daily/weekly briefing
- Follow: researchers, companies, models, repositories, topics, benchmarks
- "What changed" diff view for releases, benchmarks, model cards

### Deep Research
- Agentic multi-step research for complex questions
- Source verification and citation tracking
- Structured output: comparison tables, benchmark summaries, architecture diagrams

### Intelligence Layer
- Entity extraction: models, datasets, benchmarks, researchers, companies
- Relationship mapping: paper → model → code → benchmark → researcher
- Technical fact extraction: benchmark scores, model specs, architecture details

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        FRONTEND (React/Next.js)                 │
│  Search UI • Discovery Feed • Research Workspace • Settings    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        API GATEWAY (FastAPI/Go)                 │
│  Auth • Rate Limit • Request Routing • Response Cache           │
└─────────────────────────────────────────────────────────────────┘
          │                    │                    │
          ▼                    ▼                    ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   SEARCH SVC    │ │ DISCOVERY SVC   │ │  RESEARCH SVC   │
│  (hybrid retr.) │ │ (personalized)  │ │ (agentic)       │
└────────┬────────┘ └────────┬────────┘ └────────┬────────┘
         │                   │                    │
         └───────────────────┼────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      INDEX / STORAGE LAYER                      │
│  PostgreSQL (metadata, users, relationships)                   │
│  Elasticsearch/OpenSearch (lexical + hybrid search)            │
│  Vector DB (Qdrant/Weaviate) (semantic search)                 │
│  Redis (cache, sessions, realtime)                             │
└─────────────────────────────────────────────────────────────────┘
                             ▲
                             │
┌─────────────────────────────────────────────────────────────────┐
│                      INGESTION PIPELINE                         │
│  Sources → Fetch → Normalize → Dedupe → Enrich → Index         │
│  • ArXiv API • GitHub API • Hugging Face API • RSS/Atom        │
│  • Docs sites • Company blogs • Benchmark leaderboards         │
└─────────────────────────────────────────────────────────────────┘
```

## Current Status

**Phase 0: Foundation & Research** (Current)
- [x] Repository initialized
- [ ] Project constitution & product vision
- [ ] Competitor analysis
- [ ] Data source strategy
- [ ] Search architecture design
- [ ] System architecture & data model
- [ ] Evaluation framework
- [ ] Roadmap & MVP definition

**Phase 1: Data Foundation** (Next)
- [ ] Ingestion framework
- [ ] Core source connectors (ArXiv, GitHub, HF, RSS)
- [ ] Normalization & deduplication
- [ ] Basic indexing pipeline

**Phase 2: Search MVP**
- [ ] Hybrid search (lexical + semantic)
- [ ] Basic ranking (recency + authority + relevance)
- [ ] Search API + minimal UI

**Phase 3: Search Quality**
- [ ] Reranking (cross-encoder)
- [ ] Query understanding/intent classification
- [ ] Evaluation benchmarks

**Phase 4: Personalized Discovery**
- [ ] User interest modeling
- [ ] Follow system (entities, topics)
- [ ] Personalized ranking

**Phase 5: Tracking & Change Detection**
- [ ] Release/changelog diffing
- [ ] Benchmark tracking
- [ ] Alert system

**Phase 6: Agentic Research**
- [ ] Research planner agent
- [ ] Source research agents
- [ ] Synthesis & verification

## Repository Structure

```
tech-intelligence-engine/
├── README.md                    # This file
├── .gitignore
├── .env.example
├── docs/
│   ├── PROJECT_CONSTITUTION.md  # Highest-level source of truth
│   ├── PRODUCT_VISION.md        # Product vision & UX
│   ├── REQUIREMENTS.md          # Functional & non-functional reqs
│   ├── COMPETITOR_ANALYSIS.md   # Competitive landscape
│   ├── DATA_SOURCE_STRATEGY.md  # Source ecosystem & access
│   ├── SEARCH_STRATEGY.md       # Search system design
│   ├── PERSONALIZATION.md       # Personalization approach
│   ├── AGENT_ARCHITECTURE.md    # Agent roles & boundaries
│   ├── SYSTEM_ARCHITECTURE.md   # High-level system design
│   ├── DATA_MODEL.md            # Domain entities & relationships
│   ├── SECURITY.md              # Security requirements
│   ├── EVALUATION.md            # Evaluation framework
│   ├── ROADMAP.md               # Phased roadmap
│   └── decisions/
│       ├── README.md            # ADR format guide
│       └── 0001-*.md            # Architecture Decision Records
├── research/
│   ├── competitors/
│   ├── sources/
│   ├── search-quality/
│   ├── architecture/
│   ├── experiments/
│   └── findings/
└── (implementation directories added in later phases)
```

## Development Principles

1. **Evidence over intuition** — Every architectural claim backed by research or experiment
2. **Vertical depth over horizontal breadth** — Own the tech ecosystem completely before expanding
3. **Simplicity first** — Start with deterministic pipelines; add agents only where they add clear value
4. **Measurable quality** — Search quality evaluated against benchmarks, not vibes
5. **Source transparency** — Every result traceable to source; citations mandatory
6. **User control** — Personalization explainable and adjustable
7. **Freshness as feature** — Ingestion latency measured and optimized
8. **No vendor lock-in** — Open-source components preferred; avoid proprietary APIs where alternatives exist

## Research & Documentation Links

| Document | Purpose |
|----------|---------|
| [Project Constitution](docs/PROJECT_CONSTITUTION.md) | Product thesis, principles, non-goals |
| [Product Vision](docs/PRODUCT_VISION.md) | UX, workflows, long-term direction |
| [Requirements](docs/REQUIREMENTS.md) | Prioritized functional/non-functional requirements |
| [Competitor Analysis](docs/COMPETITOR_ANALYSIS.md) | Market landscape & gap identification |
| [Data Source Strategy](docs/DATA_SOURCE_STRATEGY.md) | Source catalog, access methods, licensing |
| [Search Strategy](docs/SEARCH_STRATEGY.md) | Retrieval, ranking, reranking design |
| [Personalization](docs/PERSONALIZATION.md) | Interest modeling, follow system, privacy |
| [Agent Architecture](docs/AGENT_ARCHITECTURE.md) | Agent roles, boundaries, orchestration |
| [System Architecture](docs/SYSTEM_ARCHITECTURE.md) | Component diagram, data flows |
| [Data Model](docs/DATA_MODEL.md) | Entities, relationships, schema concepts |
| [Security](docs/SECURITY.md) | Threat model, mitigations, agent risks |
| [Evaluation](docs/EVALUATION.md) | Benchmarks, metrics, quality gates |
| [Roadmap](docs/ROADMAP.md) | Phased delivery plan |

## Future Demo Section

*Placeholder for live demo, screenshots, or video walkthrough once MVP is functional.*

## License

MIT License — see [LICENSE](LICENSE) for details (to be added).

---

**Note**: This repository contains project foundation and documentation only. Application implementation has NOT started. See [Roadmap](docs/ROADMAP.md) for phased development plan.