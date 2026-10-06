# ADR-0002: Vector Database Selection: Qdrant

**Status**: Accepted
**Date**: 2026-10-06
**Deciders**: Rishi Sakhija
**Technical Story**: Vector DB selection for semantic search

## Context

We need a vector database for dense embedding storage and similarity search. Requirements:

- 1024-dimensional vectors (bge-large-en-v1.5)
- ~1M vectors initially, up to 10M (Phase 3)
- Filtering by payload (entity_type, source, date, topics)
- Sub-100ms p95 for top-100 ANN search
- Self-hosted, open-source (MIT/Apache preferred)
- Horizontal scaling path
- Python client with async support

## Decision

**Choose Qdrant** as the vector database.

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **Weaviate** | GraphQL; hybrid search built-in; modules (generative, reranker) | Heavier (Java/Go); more complex; GraphQL not needed |
| **Pinecone** | Managed; serverless; excellent performance | Proprietary; cost at scale; vendor lock-in |
| **Milvus** | Highly scalable; cloud-native; GPU indexing | Complex deployment; heavier resource usage |
| **Chroma** | Simple; Python-native; good for dev | Not production-ready for 1M+; limited filtering |
| **pgvector (PostgreSQL)** | Single DB; ACID; simple | ANN performance degrades >100k; no HNSW tuning |
| **Elasticsearch dense_vector** | Single stack; already using ES | ANN less mature; less tuning; memory heavy |
| **Vespa** | Full search platform; ranking expressions | Steep learning curve; overkill for vector-only |

## Rationale

- **Qdrant** is Rust-based, fast, memory-efficient
- **Filtering**: Native payload filtering (pre-filter / post-filter) with HNSW
- **Open-source** (Apache-2.0); active development; growing adoption
- **Horizontal scaling**: Cluster mode with sharding/replication
- **Client**: Official Python async client; good documentation
- **Local dev**: Single binary / Docker; no JVM
- **Cost**: Free self-hosted; predictable resource usage

## Consequences

### Positive
- Best-in-class ANN performance for our scale
- Payload filtering avoids post-filter round-trips
- Simple operational model (single binary / Docker)
- Good Python ecosystem integration

### Negative
- Separate system from PostgreSQL/ES (operational overhead)
- Cluster mode setup more complex than single node
- Less built-in hybrid search than Weaviate (we do hybrid at application layer)

### Risks
- **Cluster stability**: Test thoroughly before Phase 3 scale
- **Mitigation**: Start single-node; migrate to cluster when needed

## Implementation Notes

- Collections: `documents_title_abstract`, `documents_chunks`, `entities`
- HNSW params: `m=16`, `ef_construct=100`, `ef_search=64`
- Quantization: Scalar quantization (int8) for memory savings at scale
- Payload indexes on: `entity_type`, `source`, `published_at`, `topics`

## Related ADRs

- ADR-0001: Hybrid Search Architecture
- ADR-0003: Embedding Model Selection