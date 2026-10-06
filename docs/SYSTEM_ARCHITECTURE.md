# System Architecture

> High-level architecture, component diagram, data flows, and technology choices.

---

## 1. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              EXTERNAL CLIENTS                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │  Web App     │  │  Mobile Web  │  │  API Clients │  │  Webhooks    │        │
│  │  (React)     │  │  (PWA)       │  │  (SDK/CLI)   │  │  (GitHub/HF) │        │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘        │
└─────────┼─────────────────┼─────────────────┼─────────────────┼────────────────┘
          │                 │                 │                 │
          ▼                 ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              API GATEWAY (Kong / Traefik / FastAPI)              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │  Auth        │  │  Rate Limit  │  │  Request     │  │  Response    │        │
│  │  (JWT/OAuth) │  │  (Token Bucket)│ │  Routing     │  │  Cache       │        │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘        │
└─────────────────────────────────────────────────────────────────────────────────┘
          │                 │                 │                 │
    ┌─────┴─────┐     ┌─────┴─────┐     ┌─────┴─────┐     ┌─────┴─────┐
    ▼           ▼     ▼           ▼     ▼           ▼     ▼           ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ SEARCH │ │DISCOVERY│ │ RESEARCH│ │  USER   │ │ ADMIN   │ │ WEBHOOK  │ │ HEALTH │
│  SVC   │ │  SVC   │ │  SVC    │ │  SVC    │ │  SVC    │ │  HANDLER │ │ CHECK  │
└────┬───┘ └────┬───┘ └────┬───┘ └────┬───┘ └────┬───┘ └────┬───┘ └────┬───┘
     │          │          │          │          │          │          │
     └──────────┼──────────┼──────────┼──────────┼──────────┼──────────┘
                ▼          ▼          ▼          ▼          ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              SHARED INFRASTRUCTURE                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │ PostgreSQL   │  │ Elasticsearch│  │ Vector DB    │  │ Redis        │        │
│  │ (Primary DB) │  │ (Lexical +   │  │ (Qdrant/     │  │ (Cache,      │        │
│  │  Users,      │  │  Hybrid)     │  │  Weaviate)   │  │  Sessions,   │        │
│  │  Entities,   │  │              │  │              │  │  Queues,     │        │
│  │  Relations   │  │              │  │              │  │  Rate Limit) │        │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘        │
└─────────────────────────────────────────────────────────────────────────────────┘
                │                │                │                │
                ▼                ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              INGESTION PIPELINE (Async Workers)                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │ FETCH    │ │ NORMALIZE│ │ DEDUPE   │ │ ENRICH   │ │ INDEX    │ │ MONITOR  │  │
│  │ WORKERS  │ │ WORKERS  │ │ WORKERS  │ │ WORKERS  │ │ WORKERS  │ │ WORKERS  │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘  │
│         │            │            │            │            │            │       │
│         └────────────┴────────────┴────────────┴────────────┴────────────┘       │
│                                  │                                                │
│                                  ▼                                                │
│                    ┌────────────────────────┐                                    │
│                    │   MESSAGE QUEUE        │                                    │
│                    │   (Redis Streams /     │                                    │
│                    │    RabbitMQ / Kafka)   │                                    │
│                    └────────────────────────┘                                    │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Service Specifications

### 2.1 API Gateway
- **Technology**: Kong (production) / Traefik / FastAPI middleware (dev)
- **Responsibilities**: TLS termination, auth validation, rate limiting, request routing, response caching, request/response logging
- **Scaling**: Stateless, horizontal

### 2.2 Search Service
- **Technology**: FastAPI (Python 3.11+)
- **Responsibilities**:
  - Query understanding (intent, entity extraction)
  - Hybrid retrieval (ES + Vector DB)
  - Ranking (base + personalization)
  - Reranking (cross-encoder)
  - Result formatting (entity cards)
- **Dependencies**: Elasticsearch, Vector DB, Redis (cache), Personalization Service
- **Latency Budget**: p95 < 500ms
- **Scaling**: Stateless, horizontal; read replicas for ES

### 2.3 Discovery Service
- **Technology**: FastAPI
- **Responsibilities**:
  - Briefing feed generation
  - Follow system management
  - Trending computation
  - Serendipity recommendations
- **Dependencies**: PostgreSQL (follows), Redis (cached feeds), Search Service
- **Latency Budget**: p95 < 1000ms
- **Scheduling**: Feed pre-computation every 15 min + on-demand

### 2.4 Research Service
- **Technology**: FastAPI + Agent Orchestrator (custom / Temporal)
- **Responsibilities**:
  - Research planner invocation
  - Source research agent coordination
  - Synthesis + verification
  - Report generation + export
- **Dependencies**: Search Service, LLM Provider, Vector DB (for context)
- **Latency Budget**: p95 < 60s (5-step research)
- **Cost Control**: Per-request budget enforcement

### 2.5 User Service
- **Technology**: FastAPI
- **Responsibilities**:
  - Authentication (JWT + OAuth: GitHub, Google)
  - User profile, settings, follows
  - API key management
  - Data export / deletion (GDPR)
- **Dependencies**: PostgreSQL, Redis (sessions), Email service

### 2.6 Ingestion Pipeline (Async Workers)
- **Technology**: Python workers (Celery / Redis Streams / Temporal)
- **Components**:

| Worker | Responsibility | Scaling |
|--------|----------------|---------|
| **Fetch** | Poll APIs, consume webhooks, fetch RSS | Per-source (partitioned) |
| **Normalize** | Map raw source data → unified schema | Stateless, horizontal |
| **Dedupe** | Exact (DOI/ID) + fuzzy (title+author) matching | Single-writer for entity resolution |
| **Enrich** | Classification, entity extraction, quality scoring | GPU workers for LLM tasks |
| **Index** | Write to ES, Vector DB, PostgreSQL | Batched writes |
| **Monitor** | Lag alerts, quality metrics, dead letter queue | Singleton |

---

## 3. Data Flow Details

### 3.1 Search Request Flow

```
1. Client → API Gateway (auth, rate limit)
2. Gateway → Search Service
3. Search Service:
   a. Query Understanding (intent, entities) → ~50ms
   b. Lexical Search (ES) → ~100ms
   c. Vector Search (Qdrant) → ~100ms
   d. Metadata Filtering (ES) → ~20ms
   e. Reciprocal Rank Fusion (RRF) → ~10ms
   f. Personalization Boost (Redis cache) → ~10ms
   g. Cross-Encoder Rerank (top-50) → ~150ms
   h. Entity Card Formatting → ~20ms
4. Response → Gateway (cache) → Client
Total: ~450ms p95
```

### 3.2 Ingestion Flow (Per Document)

```
1. Fetch Worker → Raw Document (JSON) → Queue
2. Normalize Worker → Normalized Document → Queue
3. Dedupe Worker:
   a. Exact key lookup (DOI, ArXiv ID, HF Model ID, GitHub repo)
   b. If match: merge metadata, update timestamp
   c. If no match: fuzzy match queue (LLM-assisted)
   d. Output: Canonical Entity ID → Queue
4. Enrich Worker (parallel):
   a. Classification Agent → type, topics, tasks
   b. Entity Extraction Agent → benchmarks, models, researchers
   c. Quality Scorer → authority, freshness, completeness
   d. Entity Resolution Agent → link to KG entities
5. Index Worker (batched):
   a. PostgreSQL: Entity + Document + Relationships
   b. Elasticsearch: Full document + facets
   c. Vector DB: Chunk embeddings
   d. Redis: Invalidate relevant caches
6. Monitor Worker: Log metrics, check lag, alert on anomalies
```

### 3.3 Briefing Feed Generation

```
Scheduled (every 15 min) + On-Demand:
1. For each active user:
   a. Get followed entities (PostgreSQL)
   b. Query change log for each entity since last_visit
   c. Rank changes: breaking > benchmark > version > minor
   d. Get implicit recommendations (vector search on interest vector)
   e. Get serendipity items (k-NN on interest vector, filtered)
   f. Merge + rank → Feed items
   g. Cache in Redis (key: user_id, TTL: 1 hour)
2. On user visit:
   a. Serve cached feed
   b. Update last_visit timestamp
   c. Async: recompute feed for next visit
```

---

## 4. Technology Choices & Rationale

### 4.1 Core Stack

| Layer | Choice | Alternatives Considered | Rationale |
|-------|--------|------------------------|-----------|
| **API Framework** | FastAPI | Flask, Django, Go (Gin), Node (Express) | Python ecosystem for ML; async; OpenAPI; type safety |
| **Primary DB** | PostgreSQL 16+ | MySQL, MongoDB, CockroachDB | Relational for entities/relations; JSONB for flexibility; mature |
| **Lexical Search** | Elasticsearch 8+ | OpenSearch, Typesense, Meilisearch | Hybrid search; mature; filtering; scaling |
| **Vector DB** | Qdrant | Weaviate, Pinecone, Milvus, Chroma | Open-source; filtering; Rust performance; local dev |
| **Cache/Queue** | Redis 7+ | RabbitMQ, Kafka, Redis Streams | Simple; streams for queues; pub/sub for invalidation |
| **Embedding Model** | BAAI/bge-large-en-v1.5 | bge-m3, nomic-embed, e5-large, OpenAI | MIT license; strong benchmarks; local inference |
| **Reranker** | BAAI/bge-reranker-large | bge-reranker-v2, Cohere, Jina | MIT license; strong; local |
| **LLM Provider** | Multi-provider (OpenAI, Anthropic, Local) | Single provider | Cost optimization; redundancy; local for sensitive |
| **Frontend** | Next.js 14+ (React 18, TypeScript) | Remix, SvelteKit, vanilla | App Router; RSC; Vercel deploy; ecosystem |
| **Orchestration** | Temporal (or Celery + Redis Streams) | Airflow, Dagster, custom | Durable execution; retries; visibility |
| **Observability** | OpenTelemetry + Prometheus + Grafana | Datadog, Honeycomb | Open standards; self-hosted; cost |

### 4.2 Infrastructure (Phase 1-2: Single Node / Small Cluster)

```
┌─────────────────────────────────────────┐
│           DOCKER COMPOSE / K3S          │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │  API    │ │  API    │ │  API    │   │  (3 replicas)
│  │ Gateway │ │Gateway  │ │Gateway  │   │
│  └────┬────┘ └────┬────┘ └────┬────┘   │
│       │           │           │        │
│  ┌────┴───────────┴───────────┴────┐  │
│  │      LOAD BALANCER (Traefik)     │  │
│  └────┬───────────┬───────────┬────┘  │
│       │           │           │        │
│  ┌────┴────┐ ┌────┴────┐ ┌────┴────┐  │
│  │PostgreSQL│ │Elastic- │ │ Qdrant  │  │  (Single node, persistent volumes)
│  │          │ │ search  │ │         │  │
│  └─────────┘ └─────────┘ └─────────┘  │
│  ┌─────────┐ ┌─────────┐              │
│  │  Redis  │ │ Workers │              │
│  │         │ │(Celery) │              │
│  └─────────┘ └─────────┘              │
└─────────────────────────────────────────┘
```

**Resource Estimates (Phase 1)**:
| Component | CPU | RAM | Disk | GPU |
|-----------|-----|-----|------|-----|
| API (3x) | 2 cores | 4 GB | 10 GB | No |
| PostgreSQL | 4 cores | 16 GB | 200 GB | No |
| Elasticsearch | 4 cores | 16 GB | 500 GB | No |
| Qdrant | 4 cores | 16 GB | 200 GB | No |
| Redis | 2 cores | 8 GB | 20 GB | No |
| Workers (4x) | 4 cores | 16 GB | 50 GB | **Yes (1x A10G/T4)** |
| **Total** | ~24 cores | ~92 GB | ~1 TB | 1 GPU |

---

## 5. Data Models (Summary - See `DATA_MODEL.md`)

### 5.1 Core Tables (PostgreSQL)

```sql
-- Users & Auth
users, api_keys, oauth_accounts, sessions

-- Entities (Canonical)
entities (id, type, name, canonical_data, quality_score, created_at, updated_at)
entity_aliases (entity_id, source, source_id, confidence)

-- Documents
documents (id, entity_id, source, source_id, url, title, abstract, full_text_ref, metadata, hash, quality_score, indexed_at)

-- Relationships
entity_relationships (source_entity_id, target_entity_id, relationship_type, confidence, source_doc_id)

-- User Personalization
user_follows (user_id, entity_id, entity_type, notification_prefs, created_at)
user_interest_vector (user_id, vector, updated_at)
user_search_history (user_id, query, results_shown, clicks, dwell_ms, created_at)

-- Ingestion
ingestion_log (source, status, docs_processed, docs_failed, latency_ms, started_at, completed_at)
dead_letter_queue (source, raw_payload, error, retry_count, created_at)
```

### 5.2 Elasticsearch Index

```json
{
  "mappings": {
    "properties": {
      "entity_id": {"type": "keyword"},
      "entity_type": {"type": "keyword"},
      "title": {"type": "text", "analyzer": "technical"},
      "abstract": {"type": "text", "analyzer": "technical"},
      "full_text": {"type": "text", "analyzer": "technical"},
      "authors": {"type": "keyword"},
      "author_ids": {"type": "keyword"},
      "source": {"type": "keyword"},
      "source_quality": {"type": "float"},
      "published_at": {"type": "date"},
      "indexed_at": {"type": "date"},
      "topics": {"type": "keyword"},
      "tasks": {"type": "keyword"},
      "benchmarks": {"type": "keyword"},
      "models": {"type": "keyword"},
      "companies": {"type": "keyword"},
      "license": {"type": "keyword"},
      "quality_score": {"type": "float"},
      "citation_count": {"type": "integer"},
      "github_stars": {"type": "integer"}
    }
  }
}
```

### 5.3 Vector DB Collections (Qdrant)

| Collection | Vector Size | Payload |
|------------|-------------|---------|
| `documents_title_abstract` | 1024 | entity_id, entity_type, title, source, published_at |
| `documents_full_text_chunks` | 1024 | entity_id, chunk_index, text, chunk_type |
| `entities` | 1024 | entity_id, entity_type, name, description |

---

## 6. API Design (REST + WebSocket)

### 6.1 Search API

```
POST   /api/v1/search              # Hybrid search
GET    /api/v1/search/suggest      # Autocomplete
GET    /api/v1/search/explain      # Why this result?
```

### 6.2 Discovery API

```
GET    /api/v1/briefing            # Personalized feed
GET    /api/v1/trending            # Trending in topics
POST   /api/v1/follows             # Follow entity
DELETE /api/v1/follows/{entity_id} # Unfollow
GET    /api/v1/follows             # List follows
```

### 6.3 Research API

```
POST   /api/v1/research            # Start deep research
GET    /api/v1/research/{task_id}  # Poll status / get result
GET    /api/v1/research/{task_id}/export  # Export report
```

### 6.4 Entity API

```
GET    /api/v1/entities/{entity_id}      # Entity detail
GET    /api/v1/entities/{entity_id}/changes  # Change history
GET    /api/v1/entities/{entity_id}/related  # Related entities
```

### 6.5 WebSocket (Real-time)

```
WS /api/v1/ws/briefing     # Live briefing updates
WS /api/v1/ws/research     # Research progress updates
```

---

## 7. Deployment Strategy

### 7.1 Environments

| Environment | Purpose | Infrastructure |
|-------------|---------|----------------|
| **Local** | Development | Docker Compose |
| **Staging** | Integration testing | K3s (single node) |
| **Production** | Live traffic | Kubernetes (managed: GKE/EKS) or VMs |

### 7.2 CI/CD Pipeline

```
Git Push → GitHub Actions
  ├─ Lint (ruff, mypy, eslint)
  ├─ Type Check
  ├─ Unit Tests (pytest, vitest)
  ├─ Integration Tests (testcontainers)
  ├─ Build Docker Images
  ├─ Security Scan (Trivy, Bandit)
  ├─ Deploy to Staging (auto)
  ├─ E2E Tests (Playwright)
  └─ Deploy to Production (manual approval)
```

### 7.3 Database Migrations

- **Tool**: Alembic
- **Policy**: Backward-compatible only; no destructive changes without version bump
- **Process**: Auto-generated → Review → Test on staging → Deploy with migration step

---

## 8. Scaling Strategy

### 8.1 Phase 1 (0-100 users): Single Node
- All services on one machine (Docker Compose)
- Vertical scaling: increase RAM/CPU
- SQLite → PostgreSQL migration when needed

### 8.2 Phase 2 (100-1000 users): Small Cluster
- Separate DB nodes (PostgreSQL primary + read replica)
- Elasticsearch cluster (3 nodes)
- Qdrant cluster (3 nodes)
- API horizontal scaling (3-5 replicas)
- Workers: dedicated nodes

### 8.3 Phase 3 (1000+ users): Distributed
- Kubernetes (GKE/EKS)
- PostgreSQL: Cloud SQL / RDS with read replicas
- Elasticsearch: Managed (Elastic Cloud) or 5+ node cluster
- Qdrant: Cluster mode
- Redis: Cluster mode
- CDN for static assets
- Multi-region for latency

---

## 9. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-10-06 | Initial system architecture |

---

*This document informs `REQUIREMENTS.md` (NFR-*), `ROADMAP.md`, and implementation planning.*