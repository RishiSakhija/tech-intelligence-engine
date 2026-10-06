# Evaluation Framework

> How we measure whether the search engine is actually good. Metrics, benchmarks, datasets, and quality gates.

---

## 1. Evaluation Philosophy

**The project must be measured, not judged by "looks good."**

| Anti-Pattern | Our Approach |
|--------------|--------------|
| "Vibe checks" on 5 queries | Systematic benchmark with 200+ labeled queries |
| Optimizing for demo queries | Diverse query set covering all intent types |
| Ignoring freshness | Freshness as explicit metric |
| No citation verification | Citation accuracy = core metric |
| Single metric (nDCG) | Multi-dimensional: relevance, freshness, diversity, authority, personalization |
| No regression detection | Nightly evaluation + alerting on regression |

---

## 2. Evaluation Dimensions

### 2.1 Primary Metrics

| Metric | Definition | Target (MVP) | Target (Phase 3+) |
|--------|------------|--------------|-------------------|
| **nDCG@10** | Normalized Discounted Cumulative Gain at 10 | > 0.65 | > 0.75 |
| **nDCG@20** | nDCG at 20 | > 0.60 | > 0.70 |
| **MRR@10** | Mean Reciprocal Rank of first relevant | > 0.70 | > 0.80 |
| **Precision@5** | % of top-5 that are relevant | > 0.70 | > 0.80 |
| **Recall@100** | % of all relevant found in top-100 | > 0.50 | > 0.65 |

### 2.2 Quality Dimensions (Beyond Relevance)

| Dimension | Metric | Target |
|-----------|--------|--------|
| **Freshness** | Median age of top-10 results (days) | < 30 (Tier 1 sources) |
| **Freshness@10** | % of top-10 published in last 90 days | > 60% |
| **Source Diversity** | Unique sources in top-10 | > 4 |
| **Entity Diversity** | Unique entities in top-10 | > 7 |
| **Authority** | Avg source quality score of top-10 | > 80 |
| **Citation Accuracy** | % of citations that resolve + match claim | > 95% |
| **Personalization Lift** | nDCG@10 personalized vs. non-personalized | > +15% |
| **Latency (p95)** | Search API response time | < 500ms |

### 2.3 Research Quality Metrics (Phase 6+)

| Metric | Definition | Target |
|--------|------------|--------|
| **Answer Accuracy** | Human-rated correctness (1-5) | > 4.0 |
| **Completeness** | Covers all sub-questions | > 90% |
| **Citation Coverage** | % claims with valid citation | > 95% |
| **Hallucination Rate** | Unsupported claims per report | < 0.05 |
| **User Satisfaction** | Post-research survey (1-5) | > 4.0 |

---

## 3. Benchmark Dataset

### 3.1 Query Set Composition (200 Queries Minimum)

| Category | Count | Intent Types | Examples |
|----------|-------|--------------|----------|
| **Entity Lookup** | 30 | Lookup | "Llama-3.1-70B release date", "arXiv:2401.12345" |
| **Comparative** | 30 | Comparison | "Llama-3.1 vs Qwen2.5 benchmarks", "MoE vs dense transformer" |
| **Exploratory** | 30 | Exploratory | "New attention mechanisms 2024", "RAG architectures" |
| **How-To / Tutorial** | 20 | Tutorial | "How to quantize Llama-3.1 to 4-bit", "Fine-tune Qwen2.5" |
| **Troubleshooting** | 20 | Troubleshooting | "Flash attention compile error H100", "vLLM OOM" |
| **Benchmark Query** | 20 | Lookup/Comparison | "MMLU scores for 7B models", "SWE-bench leaderboard" |
| **Tracking** | 20 | Tracking | "Transformers releases last month", "PyTorch 2.5 breaking changes" |
| **Research** | 30 | Complex | "Compare all open MoE models on MMLU and inference latency" |

### 3.2 Relevance Judgments

**Judgment Scale** (per query-result pair):
| Score | Label | Meaning |
|-------|-------|---------|
| 3 | **Essential** | Perfect answer; authoritative source; exactly matches intent |
| 2 | **Relevant** | Good answer; useful; minor gaps |
| 1 | **Marginal** | Tangentially related; low authority; outdated |
| 0 | **Irrelevant** | Wrong entity; wrong intent; spam |

**Judgment Process**:
1. **Expert Annotators** (2-3 ML engineers) label independently
2. **Adjudication** → Consensus on disagreements
3. **Gold Set** → Frozen for regression testing
4. **Continuous Expansion** → Add 20 queries/month from real usage

### 3.3 Freshness Ground Truth

For tracking queries, each query has:
- `expected_freshness_days`: Maximum acceptable age (e.g., 7 for "last week releases")
- `must_include_entities`: Specific releases/papers that must appear
- `must_exclude_entities`: Old versions that should not outrank new

### 3.4 Citation Ground Truth

For research queries, each expected claim has:
- `claim`: "Llama-3.1-70B scores 86.1 on MMLU"
- `expected_source`: "HF model card meta-llama/Llama-3.1-70B"
- `expected_value`: 86.1
- `tolerance`: ±0.5

---

## 4. Evaluation Pipeline

### 4.1 Offline Evaluation (Nightly)

```python
# Pseudocode
def run_evaluation():
    for query in GOLD_QUERIES:
        results = search_service.search(query.text, user_id=EVAL_USER)
        
        # Relevance
        ndcg = compute_ndcg(results, query.judgments, k=10)
        mrr = compute_mrr(results, query.judgments, k=10)
        
        # Freshness
        median_age = median_days_old(results[:10])
        fresh_pct = pct_within_days(results[:10], 90)
        
        # Diversity
        source_div = unique_sources(results[:10])
        entity_div = unique_entities(results[:10])
        
        # Authority
        avg_authority = mean(r.source_quality for r in results[:10])
        
        # Citations (for research queries)
        if query.type == "research":
            citation_acc = verify_citations(results[0].report)
        
        log_metrics(query.id, {ndcg, mrr, median_age, fresh_pct, source_div, entity_div, avg_authority, citation_acc})
    
    # Aggregate
    report = aggregate_by_category()
    alert_if_regression(report, threshold=0.05)
    publish_dashboard(report)
```

### 4.2 Online Evaluation (A/B Testing)

| Experiment | Control | Treatment | Metric | Minimum Detectable Effect |
|------------|---------|-----------|--------|---------------------------|
| **Personalization** | Base rank | +Personalization | Session success rate | +5% |
| **Reranker** | BM25+Vector | +Cross-encoder | nDCG@10 | +0.05 |
| **Recency Boost** | Half-life 180d | Half-life 90d | Freshness@10 | +10% |
| **Authority Weight** | 0.15 | 0.25 | Authority@10 | +5% |

**Assignment**: Consistent hashing on user_id (or session_id for anonymous)
**Duration**: Minimum 2 weeks; 1000+ sessions per variant
**Guardrails**: No degradation on latency, error rate, or non-target metrics

### 4.3 Regression Detection

| Signal | Threshold | Action |
|--------|-----------|--------|
| nDCG@10 drop | >5% vs. 7-day avg | Block deploy; investigate |
| Latency p95 increase | >20% vs. baseline | Block deploy |
| Error rate | >1% | Page on-call |
| Citation accuracy drop | <90% | Block research deploy |
| Freshness median | >2x target | Investigate ingestion |

---

## 5. Evaluation Infrastructure

### 5.1 Components

| Component | Technology |
|-----------|------------|
| **Benchmark Storage** | PostgreSQL (queries, judgments, metadata) |
| **Evaluation Runner** | Python (pytest + custom) |
| **Metrics DB** | Prometheus + Grafana |
| **Dashboard** | Grafana (relevance, freshness, latency, personalization) |
| **Alerting** | Prometheus Alertmanager → PagerDuty/Slack |
| **A/B Framework** | Custom (consistent hashing) or GrowthBook |

### 5.2 Data Flow

```
Nightly Cron (02:00 UTC)
       │
       ▼
┌──────────────────┐
│  Fetch Gold Set  │  (PostgreSQL)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Run Searches    │  (Search API; EVAL_USER = no personalization)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Compute Metrics │  (nDCG, MRR, Freshness, Diversity, Authority)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Store Results   │  (Prometheus + PostgreSQL history)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Check Regression│  (vs. 7-day rolling avg)
└────────┬─────────┘
         │
         ▼
    Alert / Dashboard
```

---

## 6. Quality Gates (Deployment Blockers)

| Gate | Check | Threshold | Phase |
|------|-------|-----------|-------|
| **Search Relevance** | nDCG@10 on gold set | > 0.60 | MVP |
| **Search Latency** | p95 < 500ms | < 500ms | MVP |
| **Freshness** | Median age top-10 | < 60 days | MVP |
| **Citation Accuracy** | Research citations resolve | > 90% | Phase 6 |
| **Personalization Lift** | A/B test significance | p < 0.05, lift > 5% | Phase 5 |
| **No Regressions** | Nightly eval vs. 7-day avg | No metric >5% worse | All |

---

## 7. Continuous Improvement Loop

```
USER INTERACTIONS
       │
       ▼
┌──────────────────┐
│  Implicit Signals│  (clicks, dwell, follows, research)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Retraining Data │  (LTR features, interest vectors)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Model Updates   │  (Reranker fine-tune, embedding refresh)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Offline Eval    │  (Gold set + new queries from logs)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  A/B Test        │  (Gradual rollout)
└────────┬─────────┘
         │
         ▼
    PRODUCTION
```

---

## 8. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-10-06 | Initial evaluation framework |

---

*This document informs `REQUIREMENTS.md` (NFR-*), `ROADMAP.md` (evaluation milestones), and `SEARCH_STRATEGY.md` (ranking signals).*