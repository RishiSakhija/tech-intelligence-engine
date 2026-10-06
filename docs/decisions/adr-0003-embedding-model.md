# ADR-0003: Embedding Model: BAAI/bge-large-en-v1.5

**Status**: Accepted
**Date**: 2026-10-06
**Deciders**: Rishi Sakhija
**Technical Story**: Embedding model selection for semantic search

## Context

We need a dense embedding model for semantic search over technical documents (papers, model cards, code READMEs, changelogs, blogs). Requirements:

- Strong performance on technical/scientific text
- 1024 dimensions (balance of quality vs. storage/index size)
- Open license (MIT/Apache) for commercial use
- Fast inference (CPU acceptable, GPU preferred)
- Multilingual a plus (future)
- Long context (512+ tokens) for abstracts + titles

## Decision

**Choose BAAI/bge-large-en-v1.5** as the primary embedding model.

**Fallback/Alternatives for specific uses**:
- `BAAI/bge-m3` — if multilingual needed (1024 dim, multi-granularity)
- `nomic-ai/nomic-embed-text-v1.5` — if longer context needed (8192 tokens, Apache-2.0)
- `BAAI/bge-small-en-v1.5` — for CPU-only / edge (384 dim, faster)

## Alternatives Considered

| Model | Dim | License | MTEB (Retrieval) | Context | Notes |
|-------|-----|---------|------------------|---------|-------|
| **bge-large-en-v1.5** | 1024 | MIT | 62.3 (BEIR) | 512 | Strong technical; fast |
| **bge-m3** | 1024 | MIT | 63.1 (BEIR) | 8192 | Multi-lingual; multi-granularity |
| **nomic-embed-text-v1.5** | 768 | Apache-2.0 | 60.2 (BEIR) | 8192 | Long context; fully open |
| **e5-large-v2** | 1024 | MIT | 61.8 (BEIR) | 512 | Microsoft; strong |
| **OpenAI text-embedding-3-large** | 3072 | Proprietary | 64.6 (BEIR) | 8192 | API only; cost; vendor lock-in |
| **Cohere embed-v3** | 1024 | Proprietary | 63.5 (BEIR) | 512 | API only; cost |
| **SFR-Embedding-Mistral** | 4096 | Apache-2.0 | 64.2 (BEIR) | 32768 | Large; slower; newer |

## Rationale

- **bge-large-en-v1.5** hits the sweet spot: strong BEIR scores, MIT license, 512 context sufficient for title+abstract, fast inference
- **BGE family** is widely adopted in open-source RAG; good community support
- **1024 dim** balances quality with Qdrant index size (~4MB per 1M vectors vs 16MB for 3072-dim)
- **No API dependency** — can run locally on CPU/GPU; no rate limits, no cost per request
- **Fine-tuning path**: Can fine-tune on our domain (technical queries + relevance judgments) if needed

## Consequences

### Positive
- Zero marginal cost per embedding (self-hosted)
- Full control over model version, updates, fine-tuning
- Strong baseline performance without tuning
- Compatible with reranker (bge-reranker-large same family)

### Negative
- English-only (v1.5); multilingual requires bge-m3 or separate model
- 512 token limit — long documents need chunking
- Self-hosted GPU needed for batch embedding throughput

### Risks
- **Model obsolescence**: Newer models (bge-m3, SFR) may surpass — monitor MTEB
- **Mitigation**: Abstract embedding interface; swap model with re-index

## Implementation Notes

- Inference: `sentence-transformers` or `transformers` + mean pooling
- Batch size: 32 (GPU), 8 (CPU)
- Normalize embeddings (L2) for cosine similarity
- Cache embeddings for re-indexing
- Monitor: embedding latency, GPU memory, throughput

## Related ADRs

- ADR-0001: Hybrid Search Architecture
- ADR-0002: Vector Database Selection