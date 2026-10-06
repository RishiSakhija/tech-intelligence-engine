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

## Decision

Implement **hybrid retrieval with Reciprocal Rank Fusion (RRF)** combining three signals:

1. **Lexical Search (BM25)** via Elasticsearch — for exact term matching, IDs, rare technical terms
2. **Semantic Search (Dense Vector)** via Qdrant — for conceptual similarity, synonyms, paraphrases
3. **Metadata Filtering** via Elasticsearch — hard filters (date, source type, license) + soft boosts (authority, recency)

**Fusion**: RRF with k=60 on top-100 from each retriever → merged top-100 → cross-encoder rerank → final top-20.

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **BM25 only** | Simple, fast, explainable | Misses semantic matches; poor on "MoE routing" vs "Mixture of Experts" |
| **Vector only** | Great semantic match | Poor on exact IDs; no metadata filtering; latency higher |
| **Single dense+sparse index (e.g., Elasticsearch dense_vector)** | Single system | ES dense_vector less mature; no cross-encoder rerank integration; vendor lock-in |
| **Late fusion (weighted sum)** | Simple | Requires weight tuning; less robust than RRF |
| **Early fusion (concatenated vectors)** | Single index | Loses interpretability; harder to tune per-signal |

## Rationale

- **RRF** is parameter-free, robust, and outperforms weighted fusion in practice (NIST TREC studies)
- **Elasticsearch + Qdrant** are both open-source, self-hostable, and production-proven
- **Cross-encoder reranker** adds ~150ms but improves nDCG by 10-15% (validated in literature)
- **Separation of concerns**: Lexical for precision, semantic for recall, metadata for control
- **Explainability**: Each signal contributes visibly to "Why this result?"

## Consequences

### Positive
- Handles all query types effectively
- Each component independently scalable and tunable
- Open-source stack; no vendor lock-in
- Clear debugging: can inspect each retriever's output

### Negative
- Three systems to operate (ES, Qdrant, PostgreSQL)
- RRF fusion adds complexity vs. single index
- Cross-encoder requires GPU for low latency

### Risks
- **Mitigation**: Fallback to BM25-only if vector DB unavailable
- **Mitigation**: Reranker timeout → return base rank (degraded but functional)
- **Mitigation**: Pre-compute embeddings at index time; not query time

## Implementation Notes

- ES: Custom analyzer for technical terms (preserve "GPT-4o", "FlashAttention-2")
- Qdrant: HNSW index; payload for filtering (entity_type, source, date)
- Reranker: `BAAI/bge-reranker-large` batched inference (batch=16)
- Caching: Redis cache for frequent queries (TTL 1 hour)

## Related ADRs

- ADR-0002: Vector Database Selection
- ADR-0003: Embedding Model Selection
- ADR-0007: Personalization Approach (ranking boost)