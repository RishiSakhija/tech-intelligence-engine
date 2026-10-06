# Search Strategy

> Conceptual design of the search system: query understanding, retrieval, ranking, reranking, and quality considerations.

---

## 1. Search Philosophy

**Search is not retrieval. Search is understanding + retrieval + ranking + explanation.**

| Traditional Search | Our Approach |
|-------------------|--------------|
| Query → Keywords → Inverted Index → BM25 → 10 Blue Links | Query → Intent + Entities → Hybrid Retrieval → Multi-Signal Ranking → Entity Cards + Explanation |
| Document-centric | Entity-centric |
| Static ranking | Personalized + Freshness-aware |
| No citations | Mandatory source attribution |
| Single modality | Hybrid lexical + semantic + metadata |

---

## 2. Query Understanding

### 2.1 Intent Classification

| Intent | Signals | Handling |
|--------|---------|----------|
| **Entity Lookup** | Proper nouns, specific identifiers ("Llama-3.1-70B", "arXiv:2401.12345") | Direct entity retrieval; show entity card |
| **Comparative** | "vs", "compare", "versus", "better", "difference" | Trigger comparison schema; multi-entity retrieval |
| **Exploratory** | Broad terms ("new attention mechanisms", "MoE routing") | Cluster results by subtopic; diversity prioritized |
| **How-To / Tutorial** | "how to", "tutorial", "guide", "implement" | Prioritize code, docs, tutorials; deprioritize papers |
| **Troubleshooting** | "error", "bug", "issue", "failed", "not working" | Prioritize GitHub issues, discussions, StackOverflow |
| **Tracking** | "releases", "changelog", "updates", "what's new" | Prioritize release entities; chronological sort |
| **Benchmark Query** | Benchmark names ("MMLU", "HumanEval", "SWE-bench") + model/task | Structured benchmark retrieval |

### 2.2 Entity Extraction from Queries

**Gazetteer + NER Pipeline**:

```python
# Entity types to extract
ENTITY_TYPES = [
    "MODEL",           # Llama-3.1-70B, GPT-4o, Qwen2.5-72B
    "BENCHMARK",       # MMLU, HumanEval, SWE-bench, MT-Bench
    "RESEARCHER",      # "Yann LeCun", "Karpathy"
    "COMPANY",         # "Google", "Meta", "Anthropic", "Hugging Face"
    "FRAMEWORK",       # "PyTorch", "Transformers", "vLLM", "LangChain"
    "PAPER_ID",        # "arXiv:2401.12345", "2401.12345"
    "TASK",            # "coding", "reasoning", "translation", "summarization"
    "METRIC",          # "accuracy", "F1", "BLEU", "pass@1"
    "VERSION",         # "v1.2", "3.1", "2.5.1"
    "LICENSE",         # "Apache-2.0", "MIT", "CC-BY-4.0"
]
```

**Implementation**:
- Gazetteer: Curated lists from index (model names, benchmark names, researcher names)
- NER: Fine-tuned transformer (e.g., `dslim/bert-base-NER` fine-tuned on technical queries)
- Fallback: LLM-based extraction for complex queries (cached)

### 2.3 Query Expansion & Reformulation

| Technique | Use Case |
|-----------|----------|
| **Synonym expansion** | "LLM" → "large language model", "foundation model" |
| **Acronym expansion** | "RLHF" → "reinforcement learning from human feedback" |
| **Model family expansion** | "Llama 3" → "Llama-3-8B", "Llama-3-70B", "Llama-3.1-*" |
| **Benchmark alias resolution** | "HumanEval" → "HumanEval pass@1", "HumanEval+" |
| **Temporal expansion** | "recent" → "last 30 days"; "this year" → "2024" |

---

## 3. Retrieval Strategy

### 3.1 Hybrid Retrieval Architecture

```
                    ┌─────────────────┐
                    │   QUERY         │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
       ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
       │  LEXICAL    │ │  SEMANTIC   │ │  METADATA   │
       │  (BM25)     │ │  (Vector)   │ │  FILTERS    │
       │             │ │             │ │             │
       │ Elasticsearch│ │ Qdrant/     │ │ PostgreSQL  │
       │             │ │ Weaviate    │ │ (facets)    │
       └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
              │              │              │
              └──────────────┼──────────────┘
                             ▼
                    ┌─────────────────┐
                    │  RECIPROCAL     │
                    │  RANK FUSION    │
                    │  (RRF)          │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  TOP-K (100)    │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  RERANKER       │
                    │  (Cross-Encoder)│
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  FINAL TOP-K    │
                    │  (10-20)        │
                    └─────────────────┘
```

### 3.2 Lexical Search (BM25)

**Fields & Weights**:
| Field | Weight | Notes |
|-------|--------|-------|
| `title` | 3.0 | Highest signal |
| `abstract` / `description` | 2.0 | |
| `authors` | 1.5 | |
| `tags` / `keywords` | 1.5 | |
| `full_text` | 1.0 | If available |
| `repository_name` | 1.0 | For code |
| `model_card_text` | 1.0 | For models |

**Analyzers**:
- Standard + English stemmer
- Technical term preservation (don't split "Transformer-XL", "GPT-4o")
- N-gram for partial matching on model names

### 3.3 Semantic Search (Dense Vector)

**Embedding Model**: `BAAI/bge-large-en-v1.5` (1024 dim, MIT license, strong on technical)
- Alternative: `BAAI/bge-m3` (multi-lingual, multi-granularity)
- Alternative: `nomic-ai/nomic-embed-text-v1.5` (long context, Apache-2.0)

**Chunking Strategy**:
| Document Type | Chunking |
|---------------|----------|
| Papers | Title + Abstract (single vector); Full text: 512-token chunks with 50 overlap |
| Models | Model card sections (architecture, training, eval) as separate vectors |
| Code/Repos | README + key docs; not full code |
| Releases | Changelog sections |
| Benchmarks | Task + dataset + metric description |

**Index**: Qdrant (local, open-source, filtering support) or Weaviate (if GraphQL needed)

### 3.4 Metadata Filtering (Pre/Post-Filter)

**Pre-filter** (applied before fusion): Hard filters (date range, source type, license)
**Post-filter** (applied after fusion): Soft filters (author, company, model family) with boosting

**Facet Index**: Elasticsearch keyword fields for all filterable attributes

---

## 4. Ranking

### 4.1 Ranking Signals (Base Ranker)

| Signal | Weight | Computation |
|--------|--------|-------------|
| **BM25 Score** | 0.25 | Normalized 0-1 |
| **Vector Similarity** | 0.25 | Cosine similarity normalized |
| **Recency** | 0.20 | Exponential decay: `exp(-days/half_life)`; half_life=90 days |
| **Authority** | 0.15 | Source quality + citation count + stars + author h-index |
| **Technical Relevance** | 0.10 | Entity match boost (query entities in doc) |
| **Diversity Penalty** | -0.05 | MMR penalty for same entity |

**Authority Score Components**:
```
authority = 0.4 * source_quality_score +
            0.3 * log(citations + 1) / log(max_citations) +
            0.2 * log(github_stars + 1) / log(max_stars) +
            0.1 * author_h_index_normalized
```

### 4.2 Personalization Boost (Post-MVP)

```
final_score = base_score * (1 + personalization_boost)

personalization_boost = 
    0.3 * explicit_follow_boost +      # Followed entities: +30%
    0.2 * implicit_interest_boost +    # Interest vector similarity: +20%
    0.1 * recent_activity_boost        # Recent searches/clicks: +10%
```

### 4.3 Recency Strategy

| Document Type | Recency Signal | Half-Life |
|---------------|----------------|-----------|
| Papers (ArXiv) | Submission date | 180 days |
| Papers (Conference) | Publication date | 365 days |
| Models (HF) | Last updated / card update | 90 days |
| Releases (GitHub) | Release date | 60 days |
| Blog Posts | Publication date | 30 days |
| Benchmarks | Last submission date | 180 days |

**Breaking Change Boost**: +50% for releases tagged "breaking" in last 30 days

---

## 5. Reranking

### 5.1 Cross-Encoder Reranker

**Model**: `BAAI/bge-reranker-large` (or `BAAI/bge-reranker-v2-m3`)
- Input: Query + Document (title + abstract + key metadata)
- Output: Relevance score (0-1)
- Batch size: 16-32 for latency

**Why Cross-Encoder**: Bi-encoder (vector search) loses query-document interaction. Cross-encoder captures fine-grained relevance.

**Fallback**: If reranker unavailable, use BM25 + vector fusion only (degraded quality)

### 5.2 Reranking Pipeline

```
Top-100 from RRF
       │
       ▼
┌──────────────────┐
│  Group by Entity │  (deduplicate same paper/model across sources)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Cross-Encoder   │  Score each unique entity
│  (batch=16)      │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Final Sort      │  Rerank score * 0.7 + Base rank * 0.3
└────────┬─────────┘
         │
         ▼
    Top-20 Results
```

---

## 6. Result Presentation

### 6.1 Entity Cards (Not Document Snippets)

Each result is an **entity card** with structured fields:

```json
{
  "entity_type": "PAPER",
  "entity_id": "arxiv:2401.12345",
  "title": "Edge0: Serving 35B MoEs from SSD...",
  "authors": ["Bupalinyu", "..."],
  "venue": "arXiv",
  "date": "2024-01-15",
  "tldr": "Streaming MoE inference engine with prerouter...",
  "benchmarks": [{"name": "MMLU", "value": "0.78", "model": "35B MoE"}],
  "code_link": "https://github.com/Edge0-AI/edge0",
  "pdf_link": "https://arxiv.org/pdf/2401.12345",
  "citations": 42,
  "source": "ArXiv",
  "source_quality": 95,
  "why_ranked": {
    "bm25": 0.82,
    "vector": 0.91,
    "recency": 0.95,
    "authority": 0.78,
    "entity_match": 1.0,
    "personalization": 0.0
  }
}
```

### 6.2 "Why This Result?" Transparency

Every result includes expandable explanation:
- **Lexical match**: Which terms matched
- **Semantic similarity**: Vector score
- **Recency**: Days since publication
- **Authority**: Source quality + citations
- **Entity match**: Which query entities found in result
- **Personalization**: Which follows/interests boosted it

---

## 7. Specialized Search Modes

### 7.1 Benchmark Search

**Query**: "MMLU scores for 7B models"
**Response**: Structured table (not list)

| Model | Size | MMLU | License | Context | VRAM (4-bit) | Source |
|-------|------|------|---------|---------|--------------|--------|
| Llama-3.1-8B | 8B | 73.2 | Llama 3 | 128k | 5.5 GB | HF Card |
| Qwen2.5-7B | 7B | 76.1 | Apache-2.0 | 32k | 4.8 GB | HF Card |
| Gemma-2-9B | 9B | 74.8 | Gemma | 8k | 6.2 GB | Paper |

**Implementation**: Pre-computed benchmark entity index; query → filtered table

### 7.2 Model Comparison

**Query**: "Compare Llama-3.1-70B vs Qwen2.5-72B"
**Response**: Side-by-side entity cards + comparison table

### 7.3 Release/Changelog Search

**Query**: "Transformers 4.40 breaking changes"
**Response**: Semantic diff view (see `PRODUCT_VISION.md`)

---

## 8. Failure Handling & Edge Cases

| Scenario | Handling |
|----------|----------|
| **No results** | Relax filters; suggest broader terms; show "trending in related topics" |
| **Vector DB down** | Fallback to BM25 only (degraded but functional) |
| **Reranker timeout** | Skip reranking; return base rank (latency SLA) |
| **Query too vague** | Ask clarifying questions ("Which benchmark?" "Which model size?") |
| **Ambiguous entity** | "Did you mean: [Llama-3.1-8B, Llama-3.1-70B, Llama-3.2]?" |
| **Empty index** | Graceful empty state with onboarding |

---

## 9. Evaluation Methodology (See `EVALUATION.md`)

**Key Metrics**:
- nDCG@10, nDCG@20 (relevance)
- MRR (first relevant result)
- Precision@K (entity-level)
- Freshness@K (median age of top-K)
- Source diversity@K
- Personalization lift (A/B)

**Test Sets**:
- 200 labeled queries across intent types
- 50 comparative queries
- 50 benchmark queries
- 50 tracking queries

---

## 10. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-10-06 | Initial search strategy |

---

*This document informs `SYSTEM_ARCHITECTURE.md`, `REQUIREMENTS.md` (FR-SRCH-*), and `EVALUATION.md`.*