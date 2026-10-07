# ADR-0001: Hybrid Search Architecture (Lexical + Semantic + Metadata)

**Status**: Accepted
**Date**: 2026-10-06
**Deciders**: Rishi Sakhija
**Technical Story**: Search architecture design

## Context

We need a search system that handles diverse technical queries: exact lookups (model IDs, paper IDs), semantic similarity ("papers about MoE routing"), and filtered browsing ("MMLU benchmarks for 7B models from 2024"). No single retrieval method handles all cases well.

**Requirements**:
- Sub-500ms p95 latency
- Support for metadata filtering (source, date, entity type, topics, benchmarks)
- Strong on both exact match and semantic similarity
- Personalization-ready ranking signals
- Explainable results ("Why this result?")
- **$0 infrastructure budget — open-source, self-hosted only**

## Decision

Implement **hybrid retrieval with Reciprocal Rank Fusion (RRF)** combining three signals:

1. **Lexical Search (BM25)** via **OpenSearch (preferred)** or Elasticsearch — for exact term matching, IDs, rare technical terms
2. **Semantic Search (Dense Vector)** via Qdrant (preferred) or OpenSearch — for conceptual similarity, synonyms, paraphrases
3. **Metadata Filtering** via OpenSearch/PostgreSQL — hard filters (date, source type, license) + soft boosts (authority, recency)

**Fusion**: RRF with k=60 on top-100 from each retriever → merged top-100 → cross-encoder rerank (optional) → final top-20.

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **BM25 only** | Simple, fast, explainable | Misses semantic matches; poor on "MoE routing" vs "Mixture of Experts" |
| **Vector only** | Great semantic match | Poor on exact IDs; no metadata filtering; latency higher |
| **Single dense+sparse index (OpenSearch)** | Single system | OpenSearch dense_vector less mature for cross-encoder integration |
| **Late fusion (weighted sum)** | Simple | Requires weight tuning; less robust than RRF |
| **Early fusion (concatenated vectors)** | Single index | Loses interpretability; harder to tune per-signal |

## Decision Status by Component

| Component | Status | Notes |
|-----------|--------|-------|
| **Hybrid retrieval (BM25 + Vector + Metadata)** | **Accepted** | Core architecture |
| **RRF fusion (k=60)** | **Accepted** | Parameter-free, robust |
| **OpenSearch for lexical + metadata** | **Preferred candidate** | Open-source, self-hosted, hybrid capable; validation required |
| **Qdrant for vector search** | **Preferred candidate** | Rust performance, payload filtering; validation required |
| **Cross-encoder reranker** | **Candidate** | Optional; validation required for latency/quality tradeoff |

## Rationale

- **RRF** is parameter-free, robust, and outperforms weighted fusion in practice (NIST TREC studies)
- **OpenSearch + Qdrant** are both open-source, self-hosted, and production-proven — aligns with $0 budget constraint
- **Cross-encoder reranker** adds ~150ms but improves nDCG by 10-15% (validated in literature); marked as candidate pending validation
- **Separation of concerns**: Lexical for precision, semantic for recall, metadata for control
- **Explainability**: Each signal contributes visibly to "Why this result?"

## Consequences

### Positive
- Handles all query types effectively
- Each component independently scalable and tunable
- Open-source stack; no vendor lock-in; $0 infrastructure cost
- Clear debugging: can inspect each retriever's output

### Negative
- Multiple systems to operate (OpenSearch, Qdrant, PostgreSQL)
- RRF fusion adds complexity vs. single index
- Cross-encoder requires GPU for low latency (optional)

### Risks
- **Mitigation**: Fallback to BM25-only if vector DB unavailable
- **Mitigation**: Reranker timeout → return base rank (degraded but functional)
- **Mitigation**: Pre-compute embeddings at index time; not query time
- **Validation required**: OpenSearch dense_vector maturity for hybrid use case

## Implementation Notes

- OpenSearch: Custom analyzer for technical terms (preserve "GPT-4o", "FlashAttention-2")
- Qdrant: HNSW index; payload for filtering (entity_type, source, date)
- Reranker: `BAAI/bge-reranker-large` batched inference (batch=16) — **candidate, validation required**
- Caching: Redis cache for frequent queries (TTL 1 hour)

## Related ADRs

- ADR-0002: Vector Database Selection
- ADR-0003: Embedding Model Selection
- ADR-0007: Personalization Approach (ranking boost)