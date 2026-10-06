# Data Source Strategy

> Comprehensive catalog of data sources, access methods, licensing considerations, and expected role in the system.

---

## 1. Source Categorization Framework

| Tier | Criteria | Examples |
|------|----------|----------|
| **Tier 1: Primary Authoritative** | Original publication venue; highest authority; structured access | ArXiv, GitHub Releases, HF Hub, Official Docs |
| **Tier 2: Curated Aggregators** | High-quality curation; structured data; reliable access | Papers with Code, Semantic Scholar, Crossref |
| **Tier 3: Community / Secondary** | Valuable but less structured; may require scraping | Blogs, Reddit, HN, Newsletters, YouTube |
| **Tier 4: Fallback / General** | Broad coverage; lower precision; web search fallback | Bing/Google Custom Search, Common Crawl |

---

## 2. Primary Sources (Tier 1)

### 2.1 ArXiv

| Attribute | Detail |
|-----------|--------|
| **Source Name** | ArXiv (arxiv.org) |
| **Type** | Preprint repository |
| **Relevance** | Critical — primary venue for ML/AI papers |
| **Freshness** | Daily updates; new submissions ~1k/day in CS |
| **Authority** | High — community standard for ML preprints |
| **Access Method** | OAI-PMH API (`http://export.arxiv.org/api/query`) + RSS feeds |
| **Rate Limits** | Polite use: 1 request/3 sec; no hard limit but be respectful |
| **Licensing** | ArXiv license (non-exclusive distribution); metadata CC0 |
| **Key Categories** | cs.AI, cs.CL, cs.LG, cs.CV, cs.NE, cs.RO, cs.SE, stat.ML |
| **Expected Role** | Primary paper source; titles, abstracts, authors, categories, PDF links |
| **Parsing Notes** | Use `arxiv` Python package; handle version updates (v1, v2, v3); extract PDF for full-text if needed |
| **Risks** | API changes rare but possible; no webhook (polling only) |

### 2.2 GitHub

| Attribute | Detail |
|-----------|--------|
| **Source Name** | GitHub (github.com) |
| **Type** | Code hosting + releases + issues + discussions |
| **Relevance** | Critical — primary venue for ML code, frameworks, model releases |
| **Freshness** | Real-time via webhooks; REST/GraphQL API for backfill |
| **Authority** | High — official repos for major frameworks (PyTorch, HF, vLLM, etc.) |
| **Access Method** | GitHub API v4 (GraphQL) + REST v3; Webhooks for real-time |
| **Rate Limits** | 5000 req/hr (authenticated); 100 req/hr (unauthenticated); conditional requests (ETag) don't count |
| **Licensing** | Repository licenses vary; metadata API terms allow indexing |
| **Target Repos** | Top 10k starred ML/AI repos + known framework orgs (pytorch, huggingface, vllm-project, etc.) |
| **Expected Role** | Releases (tags), release notes, commits, issues, repo metadata, README |
| **Parsing Notes** | Use `github` GraphQL for releases + release notes; parse changelogs for breaking changes; track dependency updates via dependabot PRs |
| **Risks** | Rate limits require careful scheduling; GraphQL complexity; repo churn (archived, deleted) |

### 2.3 Hugging Face Hub

| Attribute | Detail |
|-----------|--------|
| **Source Name** | Hugging Face Hub (huggingface.co) |
| **Type** | Model/Dataset/Space/Paper registry |
| **Relevance** | Critical — central registry for models, datasets, demos, papers |
| **Freshness** | Real-time via webhooks; API for backfill |
| **Authority** | High — de facto standard for model cards |
| **Access Method** | REST API (`huggingface_hub` Python library); Webhooks |
| **Rate Limits** | Generous for authenticated; ~1000 req/min |
| **Licensing** | Model cards: CC-BY-4.0 (typically); model weights: per-model license |
| **Entities** | Models (2M+), Datasets (500k+), Spaces (1M+), Papers (Daily Papers), Collections, Organizations |
| **Expected Role** | Model metadata (architecture, license, benchmarks, tags), Dataset metadata, Paper submissions (Daily Papers), Space demos |
| **Parsing Notes** | `huggingface_hub.HfApi` for listing; model card parsing for structured fields (license, tags, metrics, eval_results); Daily Papers RSS for trending |
| **Risks** | API v1/v2 differences; large model cards; license diversity |

### 2.4 Official Documentation Sites

| Source | URL | Access | Relevance |
|--------|-----|--------|-----------|
| **PyTorch** | pytorch.org/docs | RSS + Sitemap + Scrape | Framework releases, tutorials, migration guides |
| **Transformers (HF)** | huggingface.co/docs/transformers | RSS + Scrape | Model usage, pipelines, migration |
| **vLLM** | docs.vllm.ai | RSS + Scrape | Inference optimization, benchmarks |
| **LangChain** | python.langchain.com | RSS + Scrape | LLM app patterns, integrations |
| **LlamaIndex** | docs.llamaindex.ai | RSS + Scrape | RAG patterns, agents |
| **JAX/Flax** | jax.readthedocs.io | Scrape | Alternative framework |
| **TensorFlow/Keras** | tensorflow.org/api_docs | Scrape | Legacy but still used |
| **Ray/Anyscale** | docs.ray.io | Scrape | Distributed ML |
| **Weights & Biases** | docs.wandb.ai | Scrape | Experiment tracking |
| **MLflow** | mlflow.org/docs | Scrape | ML lifecycle |

**Common Attributes**:
- **Access**: RSS for changelogs/blogs; sitemap + structured scrape for docs
- **Freshness**: RSS = near real-time; docs scrape = weekly
- **Licensing**: Typically MIT/Apache for code; docs often CC-BY
- **Parsing**: Extract versioned API references, migration guides, breaking changes

---

## 3. Curated Aggregators (Tier 2)

### 3.1 Papers with Code

| Attribute | Detail |
|-----------|--------|
| **Source Name** | Papers with Code (paperswithcode.com) |
| **Type** | Paper + Code + Benchmark leaderboard aggregator |
| **Relevance** | High — only source with structured benchmark leaderboards |
| **Freshness** | Community updates; daily new papers |
| **Authority** | Medium-High — community curated, moderated |
| **Access Method** | Official API (paperswithcode Python package); RSS for new papers |
| **Rate Limits** | Not documented; be respectful |
| **Licensing** | CC-BY-SA for community content; paper metadata from ArXiv |
| **Key Data** | Tasks, Datasets, Methods, Benchmarks (leaderboards with scores), Paper-Code links |
| **Expected Role** | Benchmark entity population; paper ↔ code ↔ benchmark relationships |
| **Parsing Notes** | API returns structured JSON; benchmark scores have: model, paper, metric, value, hardware, date |
| **Risks** | Community data quality varies; incomplete coverage; API stability |

### 3.2 Semantic Scholar

| Attribute | Detail |
|-----------|--------|
| **Source Name** | Semantic Scholar (semanticscholar.org) |
| **Type** | AI-powered academic search engine |
| **Relevance** | High — citations, references, author profiles, TL;DR |
| **Freshness** | Near real-time for ArXiv; lag for publisher content |
| **Authority** | High — AI-enhanced metadata |
| **Access Method** | Graph API (requires API key); `semantic_scholar` Python package |
| **Rate Limits** | 100 req/5 min (free tier); higher with academic verification |
| **Licensing** | API terms: non-commercial research; metadata from publishers varies |
| **Key Data** | Paper metadata, citations, references, authors, venues, fields of study, TL;DR, influential citations |
| **Expected Role** | Citation counts, reference graphs, author profiles, paper embeddings (SPECTER) |
| **Parsing Notes** | Batch API for paper details; author search for profiles; citation graph for influence |
| **Risks** | Rate limits restrictive for full backfill; commercial use requires agreement |

### 3.3 Crossref

| Attribute | Detail |
|-----------|--------|
| **Source Name** | Crossref (crossref.org) |
| **Type** | DOI registration agency metadata |
| **Relevance** | Medium — DOI metadata for published papers |
| **Freshness** | Real-time for new DOIs |
| **Authority** | High — official DOI metadata |
| **Access Method** | REST API (`https://api.crossref.org`); no key required (polite pool) |
| **Rate Limits** | 50 req/sec (polite pool with user-agent + email) |
| **Licensing** | CC0 for metadata |
| **Key Data** | DOI, title, authors, venue, publication date, references, license, funding |
| **Expected Role** | DOI resolution; venue metadata; reference linking; license info |
| **Parsing Notes** | Use `habanero` or direct HTTP; polite pool requires `mailto:` in User-Agent |
| **Risks** | Publisher metadata quality varies; not all ArXiv papers have DOIs |

---

## 4. Community / Secondary Sources (Tier 3)

### 4.1 Technical Blogs & Company Announcements

| Source | URL | Access | Relevance | Notes |
|--------|-----|--------|-----------|-------|
| **Hugging Face Blog** | huggingface.co/blog | RSS | High | Model releases, tutorials, research |
| **Google AI Blog** | ai.googleblog.com | RSS | High | Major model releases (Gemini, etc.) |
| **Meta AI Blog** | ai.meta.com/blog | RSS | High | Llama, PyTorch, research |
| **Microsoft Research** | microsoft.com/en-us/research | RSS | Medium | Research publications |
| **NVIDIA Blog** | blogs.nvidia.com | RSS | High | Hardware, CUDA, models |
| **Anthropic Blog** | anthropic.com/news | RSS | High | Claude releases, research |
| **OpenAI Blog** | openai.com/blog | RSS | High | GPT releases, research |
| **DeepMind Blog** | deepmind.com/blog | RSS | High | AlphaFold, Gemini, research |
| **Databricks Blog** | databricks.com/blog | RSS | Medium | ML platforms, MosaicML |
| **Weights & Biases** | wandb.ai/site/articles | RSS | Medium | Experiments, reports |
| **Modal Labs** | modal.com/blog | RSS | Medium | Infrastructure, serverless |
| **Together AI** | together.ai/blog | RSS | Medium | Inference, models |

**Common**: RSS/Atom feeds; parse for model releases, benchmarks, announcements.

### 4.2 Community Aggregators

| Source | URL | Access | Relevance | Notes |
|--------|-----|--------|-----------|-------|
| **Hacker News** | hn.algolia.com | API | Medium | Trending discussions; filter ML/AI |
| **Reddit (r/MachineLearning, r/LocalLLaMA, r/MLPapers)** | reddit.com/r/... | Pushshift/API | Medium | Community discussion; sentiment |
| **Twitter/X (curated lists)** | x.com | API v2 | Low-Medium | Real-time announcements; noisy |
| **AI Weekly / TLDR AI / Import AI** | Various | RSS | Medium | Curated; derivative |

**Note**: Community sources are *signals* not *primary sources*. Use for velocity/trending, not canonical facts.

### 4.3 Benchmark Leaderboards (Direct)

| Source | URL | Access | Relevance |
|--------|-----|--------|-----------|
| **LMSYS Chatbot Arena** | chat.lmsys.org | Scrape/API | High — LLM pairwise eval |
| **Open LLM Leaderboard (HF)** | huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard | API | High — Standard benchmarks |
| **HELM (Stanford)** | crfm.stanford.edu/helm | Scrape | High — Comprehensive eval |
| **SWE-bench** | swebench.com | Scrape | High — Coding agents |
| **MMLU / MMLU-Pro** | github.com/hendrycks/test | GitHub | High — Academic benchmarks |

---

## 5. Fallback / General Sources (Tier 4)

| Source | Purpose | Access |
|--------|---------|--------|
| **Bing Custom Search / Google CSE** | Long-tail queries; coverage gaps | API (paid) |
| **Common Crawl** | Historical web corpus | S3 (free) |
| **Wikipedia / Wikidata** | Entity disambiguation; general knowledge | API / Dumps |

---

## 6. Source Coverage Matrix

| Entity Type | Primary Sources | Secondary Sources | Coverage Target |
|-------------|-----------------|-------------------|-----------------|
| **Papers** | ArXiv, Semantic Scholar, Crossref | Papers with Code, HF Daily Papers | >95% of ML papers |
| **Models** | HF Hub, GitHub (model repos) | Papers with Code, Blogs | >90% of open models |
| **Datasets** | HF Hub, Papers with Code | ArXiv (dataset papers) | >80% of ML datasets |
| **Benchmarks** | Papers with Code, HELM, LMSYS | HF Leaderboards, Blogs | >90% of major benchmarks |
| **Releases** | GitHub Releases, HF Model Cards | Blogs, Changelogs | >80% of framework releases |
| **Researchers** | Semantic Scholar, ArXiv, ORCID | HF Profiles, Google Scholar | Top 10k ML researchers |
| **Companies/Orgs** | GitHub Orgs, HF Orgs, Crossref | Blogs, News | Major AI labs + top 100 tech |

---

## 7. Licensing & Legal Considerations

### 7.1 Source Licensing Summary

| Source | Content License | Metadata License | Indexing Allowed? |
|--------|-----------------|------------------|-------------------|
| ArXiv | ArXiv license (non-exclusive) | CC0 | ✅ Yes (API provided) |
| GitHub | Per-repo license | API Terms | ✅ Yes (API provided) |
| HF Hub | Model cards: CC-BY-4.0 | API Terms | ✅ Yes (API provided) |
| Papers with Code | CC-BY-SA | CC-BY-SA | ✅ Yes (API provided) |
| Semantic Scholar | Varies by publisher | API Terms | ⚠️ Non-commercial only |
| Crossref | CC0 | CC0 | ✅ Yes |
| Blogs/RSS | Per-site (often CC-BY) | Per-site | ⚠️ Check robots.txt / Terms |

### 7.2 Key Legal Principles

1. **Metadata vs. Full Text**: We index *metadata* (titles, abstracts, authors, structured fields) — generally permissible. Full-text PDF indexing requires separate analysis.
2. **Robots.txt Compliance**: Respect `robots.txt` for all crawls.
3. **Rate Limiting**: Stay well below limits; use conditional requests (ETag, If-Modified-Since).
4. **Attribution**: Provide source links on all results; comply with CC-BY attribution requirements.
5. **No Redistribution**: We do not redistribute full papers/models/datasets — only link to canonical sources.
6. **Commercial Use**: Semantic Scholar API restricts commercial use. Verify before production launch.

### 7.3 Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Semantic Scholar commercial restriction | Use only for citation counts/author profiles in non-commercial phase; replace with OpenAlex in production |
| HF model card license diversity | Store license per model; filter by license in search |
| GitHub API rate limits | Conditional requests; webhook-first; backfill during off-peak |
| Blog content scraping | RSS-only where available; respect robots.txt; cache aggressively |
| PDF full-text extraction | Defer to Phase 3+; start with abstracts + metadata only |

---

## 8. Ingestion Architecture Implications

### 8.1 Source Connector Pattern

```python
# Abstract base for all connectors
class SourceConnector:
    async def fetch_incremental(self, since: datetime) -> List[RawDocument]
    async def fetch_full(self) -> AsyncGenerator[RawDocument, None]
    def normalize(self, raw: RawDocument) -> NormalizedDocument
    def get_source_metadata(self) -> SourceMetadata
```

### 8.2 Scheduling Strategy

| Source | Frequency | Method |
|--------|-----------|--------|
| ArXiv | Every 30 min | API polling (new + updated) |
| GitHub | Real-time | Webhooks (releases, tags) + daily backfill |
| HF Hub | Real-time | Webhooks (models, papers) + daily backfill |
| RSS Feeds | Every 15 min | Feed polling (If-Modified-Since) |
| Papers with Code | Daily | API full sync |
| Semantic Scholar | Daily | API batch (rate limited) |
| Docs Sites | Weekly | Sitemap crawl + RSS |

### 8.3 Deduplication Strategy

1. **DOI-based**: Primary key for papers (Crossref + ArXiv + Semantic Scholar)
2. **ArXiv ID**: For preprints without DOI
3. **Model ID**: HF model ID (org/name) as canonical
4. **GitHub Repo**: owner/repo as canonical
5. **Fuzzy Matching**: Title + authors + year for remaining (LLM-assisted)
6. **Human-in-loop**: Review queue for low-confidence matches

---

## 9. Source Quality Scoring

Each source gets a **Source Quality Score (0-100)** used in ranking:

| Factor | Weight | Notes |
|--------|--------|-------|
| **Authority** | 30% | Peer-reviewed > preprint > blog > forum |
| **Freshness** | 20% | Recency of publication/update |
| **Structuredness** | 20% | Schema availability (API vs. scrape) |
| **Completeness** | 15% | Coverage of entity attributes |
| **Reliability** | 15% | Uptime, consistency, error rate |

**Tier 1 sources**: 85-100
**Tier 2 sources**: 70-85
**Tier 3 sources**: 40-70
**Tier 4 sources**: 20-40

---

## 10. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-10-06 | Initial data source strategy |

---

*This document informs `INGESTION_PIPELINE` design, `SEARCH_STRATEGY.md` (source weighting), and `ROADMAP.md` (phased source onboarding).*