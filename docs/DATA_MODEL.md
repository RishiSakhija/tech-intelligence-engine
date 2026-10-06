# Data Model

> Conceptual entities, relationships, and schema design for the Tech Intelligence Engine domain.

---

## 1. Entity-Relationship Overview

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│  RESEARCHER │───────│   PAPER     │───────│   MODEL     │
│             │ writes│             │describes│             │
└─────────────┘       └──────┬──────┘       └──────┬──────┘
                             │                    │
                    ┌────────┴────────┐   ┌───────┴───────┐
                    ▼                 ▼   ▼               ▼
             ┌─────────────┐  ┌─────────────┐ ┌─────────────┐
             │  BENCHMARK  │  │  CODE/REPO  │ │  DATASET    │
             │             │  │             │ │             │
             └──────┬──────┘  └──────┬──────┘ └──────┬──────┘
                    │                │               │
                    │         ┌──────┴──────┐       │
                    │         ▼             ▼       │
                    │  ┌──────────┐ ┌──────────┐    │
                    └──│  RELEASE │ │  COMPANY │────┘
                       │          │ │          │
                       └──────────┘ └──────────┘
                              │
                       ┌──────┴──────┐
                       ▼             ▼
                ┌──────────┐  ┌──────────┐
                │  TOPIC   │  │  USER    │
                │          │  │          │
                └──────────┘  └──────────┘
```

---

## 2. Core Entities

### 2.1 Researcher / Author

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `external_ids` | JSONB | `{"semantic_scholar": "123", "orcid": "0000-...", "github": "karpathy", "hf": "karpathy"}` |
| `name` | TEXT | Canonical name |
| `aliases` | TEXT[] | Known name variations |
| `affiliations` | JSONB[] | `[{"org": "Meta", "role": "Chief AI Scientist", "start_date": "2013-01", "end_date": null}]` |
| `profile` | JSONB | Bio, website, social links, photo URL |
| `metrics` | JSONB | `{"h_index": 180, "citation_count": 250000, "paper_count": 800}` |
| `expertise_topics` | TEXT[] | Inferred from papers: `["Computer Vision", "Deep Learning", "Autonomous Vehicles"]` |
| `created_at` | TIMESTAMPTZ | |
| `updated_at` | TIMESTAMPTZ | |

**Relationships**:
- `WRITES` → Paper (many-to-many via authorship)
- `AFFILIATED_WITH` → Company (many-to-many, temporal)
- `CONTRIBUTES_TO` → Repository (via commits)
- `CREATES` → Model / Dataset (via HF Hub)

---

### 2.2 Paper / Publication

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `entity_id` | UUID | FK to entities table (canonical) |
| `title` | TEXT | |
| `abstract` | TEXT | |
| `full_text_ref` | TEXT | S3/GCS path or URL to PDF |
| `venue` | TEXT | "arXiv", "NeurIPS 2024", "ICML 2023", "Nature" |
| `venue_type` | ENUM | `PREPRINT`, `CONFERENCE`, `JOURNAL`, `WORKSHOP` |
| `publication_date` | DATE | |
| `submission_date` | DATE | For arXiv |
| `version` | INTEGER | arXiv version (v1, v2...) |
| `doi` | TEXT | If available |
| `arxiv_id` | TEXT | e.g., "2401.12345" |
| `authors` | UUID[] | FK to Researchers (ordered) |
| `topics` | TEXT[] | Controlled vocabulary |
| `tasks` | TEXT[] | `["coding", "reasoning", "translation"]` |
| `benchmarks` | JSONB[] | `[{"name": "MMLU", "score": 0.82, "split": "test"}]` |
| `models_introduced` | UUID[] | FK to Models |
| `datasets_introduced` | UUID[] | FK to Datasets |
| `code_repositories` | UUID[] | FK to Repositories |
| `citations` | UUID[] | FK to Papers (references) |
| `cited_by` | UUID[] | FK to Papers (citations) |
| `citation_count` | INTEGER | Denormalized |
| `influential_citation_count` | INTEGER | Semantic Scholar metric |
| `tldr` | TEXT | AI-generated summary |
| `source_quality` | FLOAT | 0-100 |
| `source` | ENUM | `ARXIV`, `SEMANTIC_SCHOLAR`, `CROSSREF`, `PAPERS_WITH_CODE` |
| `source_id` | TEXT | Original source identifier |
| `license` | TEXT | e.g., "arXiv license", "CC-BY-4.0" |
| `created_at` | TIMESTAMPTZ | |
| `updated_at` | TIMESTAMPTZ | |

---

### 2.3 Model

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `entity_id` | UUID | FK to entities |
| `name` | TEXT | "Llama-3.1-70B-Instruct" |
| `model_family` | TEXT | "Llama", "Qwen", "Gemma", "Mistral" |
| `base_model` | UUID | FK to base model (if fine-tune) |
| `parameter_count` | BIGINT | 70_000_000_000 |
| `parameter_count_label` | TEXT | "70B" |
| `architecture` | JSONB | `{"type": "Transformer", "attention": "GQA", "layers": 80, "hidden_dim": 8192}` |
| `context_length` | INTEGER | 131072 |
| `license` | TEXT | "Llama 3.1 Community License", "Apache-2.0" |
| `license_type` | ENUM | `OPEN`, `RESTRICTED`, `PROPRIETARY`, `CUSTOM` |
| `release_date` | DATE | |
| `developer` | UUID | FK to Company/Researcher |
| `hf_model_id` | TEXT | "meta-llama/Llama-3.1-70B-Instruct" |
| `hf_tags` | TEXT[] | `["text-generation", "chat", "instruction-tuned"]` |
| `training_data` | TEXT | Description |
| `training_compute` | TEXT | e.g., "3.8e25 FLOPs" |
| `quantizations` | JSONB[] | `[{"method": "AWQ", "bits": 4, "group_size": 128, "provider": "TheBloke"}]` |
| `benchmark_scores` | JSONB[] | `[{"benchmark_id": "...", "metric": "acc", "value": 0.82, "split": "test", "source": "HF Card"}]` |
| `eval_results` | JSONB | Full eval results from model card |
| `model_card_markdown` | TEXT | Raw model card |
| `source` | ENUM | `HF_HUB`, `GITHUB`, `PAPER`, `BLOG` |
| `source_id` | TEXT | |
| `source_quality` | FLOAT | 0-100 |
| `created_at` | TIMESTAMPTZ | |
| `updated_at` | TIMESTAMPTZ | |

**Relationships**:
- `BASED_ON` → Model (base model)
- `FINE_TUNED_FROM` → Model
- `EVALUATED_ON` → Benchmark (many-to-many with scores)
- `HOSTED_ON` → Repository / HF Space
- `INTRODUCED_IN` → Paper

---

### 2.4 Benchmark

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `entity_id` | UUID | FK to entities |
| `name` | TEXT | "MMLU", "HumanEval", "SWE-bench" |
| `full_name` | TEXT | "Massive Multitask Language Understanding" |
| `task` | TEXT | "language_understanding", "code_generation" |
| `subtask` | TEXT | "STEM", "Humanities", "Python" |
| `metric` | TEXT | "accuracy", "pass@1", "F1", "BLEU" |
| `metric_type` | ENUM | `HIGHER_BETTER`, `LOWER_BETTER` |
| `dataset` | UUID | FK to Dataset |
| `dataset_split` | TEXT | "test", "validation", "dev" |
| `description` | TEXT | |
| `paper` | UUID | FK to Paper (original benchmark paper) |
| `leaderboard_url` | TEXT | |
| `submissions` | JSONB[] | Versioned: `[{"model_id": "...", "score": 0.82, "date": "2024-01-15", "source": "HF Leaderboard", "hardware": "A100x8", "notes": "4-bit GPTQ"}]` |
| `current_sota` | JSONB | `{"model_id": "...", "score": 0.86, "date": "2024-03-01"}` |
| `source` | ENUM | `PAPERS_WITH_CODE`, `HELM`, `LMSYS`, `HF_LEADERBOARD` |
| `source_quality` | FLOAT | 0-100 |
| `created_at` | TIMESTAMPTZ | |
| `updated_at` | TIMESTAMPTZ | |

**Key Design**: Benchmark submissions are **versioned** — each submission is a record with date, source, hardware, quantization. This enables tracking progress over time.

---

### 2.5 Repository / Code

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `entity_id` | UUID | FK to entities |
| `github_id` | BIGINT | GitHub numeric ID |
| `owner` | TEXT | "vllm-project" |
| `name` | TEXT | "vllm" |
| `full_name` | TEXT | "vllm-project/vllm" |
| `description` | TEXT | |
| `url` | TEXT | "https://github.com/vllm-project/vllm" |
| `stars` | INTEGER | |
| `forks` | INTEGER | |
| `watchers` | INTEGER | |
| `language` | TEXT | Primary: "Python" |
| `languages` | JSONB | `{"Python": 85, "C++": 10, "CUDA": 5}` |
| `topics` | TEXT[] | GitHub topics |
| `license` | TEXT | "Apache-2.0" |
| `license_type` | ENUM | `OPEN`, `RESTRICTED`, `PROPRIETARY` |
| `is_fork` | BOOLEAN | |
| `is_archived` | BOOLEAN | |
| `default_branch` | TEXT | "main" |
| `releases` | JSONB[] | Versioned releases from GitHub |
| `latest_release` | JSONB | `{"tag": "v0.5.0", "date": "2024-01-15", "breaking": true, "changelog_url": "..."}` |
| `dependencies` | JSONB[] | Parsed from requirements.txt / pyproject.toml |
| `related_papers` | UUID[] | FK to Papers |
| `related_models` | UUID[] | FK to Models |
| `related_benchmarks` | UUID[] | FK to Benchmarks |
| `source_quality` | FLOAT | 0-100 |
| `last_fetched_at` | TIMESTAMPTZ | |
| `created_at` | TIMESTAMPTZ | |
| `updated_at` | TIMESTAMPTZ | |

---

### 2.6 Dataset

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `entity_id` | UUID | FK to entities |
| `name` | TEXT | "The Pile", "Common Crawl", "CodeParrot" |
| `hf_dataset_id` | TEXT | "EleutherAI/the-pile" |
| `description` | TEXT | |
| `size_gb` | FLOAT | |
| `num_examples` | BIGINT | |
| `languages` | TEXT[] | |
| `tasks` | TEXT[] | |
| `license` | TEXT | |
| `license_type` | ENUM | `OPEN`, `RESTRICTED`, `PROPRIETARY` |
| `paper` | UUID | FK to Paper (if associated) |
| `hf_tags` | TEXT[] | |
| `splits` | JSONB | `{"train": 1000000, "test": 10000}` |
| `source` | ENUM | `HF_HUB`, `PAPERS_WITH_CODE`, `KAGGLE` |
| `source_quality` | FLOAT | 0-100 |
| `created_at` | TIMESTAMPTZ | |
| `updated_at` | TIMESTAMPTZ | |

---

### 2.7 Company / Organization

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `entity_id` | UUID | FK to entities |
| `name` | TEXT | "Meta", "Google", "Hugging Face" |
| `aliases` | TEXT[] | ["Facebook AI Research", "FAIR"] |
| `website` | TEXT | |
| `github_orgs` | TEXT[] | ["facebookresearch", "pytorch"] |
| `hf_orgs` | TEXT[] | ["meta-llama", "facebook"] |
| `description` | TEXT | |
| `type` | ENUM | `BIG_TECH`, `AI_LAB`, `STARTUP`, `UNIVERSITY`, `NON_PROFIT`, `GOVERNMENT` |
| `headquarters` | TEXT | |
| `founded_year` | INTEGER | |
| `employee_count` | INTEGER | |
| `key_researchers` | UUID[] | FK to Researchers |
| `models` | UUID[] | FK to Models |
| `papers` | UUID[] | FK to Papers |
| `repositories` | UUID[] | FK to Repositories |
| `source_quality` | FLOAT | 0-100 |
| `created_at` | TIMESTAMPTZ | |
| `updated_at` | TIMESTAMPTZ | |

---

### 2.8 Release / Version

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `entity_id` | UUID | FK to entities (Model, Repository, Framework) |
| `entity_type` | ENUM | `MODEL`, `REPOSITORY`, `FRAMEWORK`, `DATASET` |
| `version` | TEXT | "v0.5.0", "1.2.3", "3.1.1" |
| `previous_version` | TEXT | "v0.4.2" |
| `release_date` | DATE | |
| `release_type` | ENUM | `MAJOR`, `MINOR`, `PATCH`, `HOTFIX`, `PRE_RELEASE` |
| `changelog_url` | TEXT | |
| `changelog_text` | TEXT | Parsed changelog |
| `breaking_changes` | JSONB[] | Structured: `[{"component": "API", "description": "Removed deprecated `old_fn`", "migration": "Use `new_fn` instead"}]` |
| `new_features` | JSONB[] | |
| `bug_fixes` | JSONB[] | |
| `security_fixes` | JSONB[] | |
| `deprecations` | JSONB[] | |
| `dependencies_updated` | JSONB[] | `[{"name": "torch", "old": "2.2", "new": "2.3", "breaking": false}]` |
| `benchmarks_affected` | UUID[] | FK to Benchmarks (if perf changes) |
| `source` | ENUM | `GITHUB_RELEASE`, `HF_MODEL_CARD`, `BLOG`, `CHANGELOG_MD` |
| `source_id` | TEXT | |
| `created_at` | TIMESTAMPTZ | |

---

### 2.9 Topic / Tag (Controlled Vocabulary)

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `name` | TEXT | "Mixture of Experts", "RAG", "Quantization" |
| `slug` | TEXT | "moe", "rag", "quantization" |
| `parent_id` | UUID | Hierarchical: "Attention" → "Flash Attention" |
| `description` | TEXT | |
| `synonyms` | TEXT[] | ["MoE", "Mixture-of-Experts"] |
| `related_topics` | UUID[] | |
| `document_count` | INTEGER | Denormalized |
| `trending_score` | FLOAT | Computed daily |
| `created_at` | TIMESTAMPTZ | |

---

### 2.10 User & Personalization

| Entity | Key Fields |
|--------|------------|
| `User` | id, email, name, avatar_url, auth_provider, created_at, settings (JSONB) |
| `UserFollow` | user_id, entity_id, entity_type, notification_prefs (JSONB), created_at |
| `UserInterestVector` | user_id, vector (1024-dim binary/float), updated_at |
| `UserSearchHistory` | user_id, query, intent, results_count, clicked_entity_ids, dwell_ms, created_at |
| `UserResearchHistory` | user_id, task_id, question, status, created_at |
| `ApiKey` | user_id, key_hash, name, scopes, rate_limit, last_used, expires_at |

---

## 3. Relationship Types

| Relationship | Source → Target | Attributes |
|--------------|-----------------|------------|
| `WRITES` | Researcher → Paper | position (author order), corresponding |
| `AFFILIATED_WITH` | Researcher → Company | role, start_date, end_date, current |
| `CITES` | Paper → Paper | context (background, method, comparison) |
| `INTRODUCES` | Paper → Model / Dataset / Benchmark | |
| `EVALUATES_ON` | Model → Benchmark | score, metric, split, date, hardware, quantization, source |
| `BASED_ON` | Model → Model (base) | fine_tune_type (LoRA, full, RLHF) |
| `HOSTED_ON` | Model → Repository / Space | |
| `IMPLEMENTS` | Repository → Paper / Model | |
| `DEPENDS_ON` | Repository → Repository | version_constraint |
| `RELEASES` | Company / Repository → Release | |
| `HAS_TOPIC` | Any Entity → Topic | confidence (0-1), source (manual, inferred) |
| `FOLLOWS` | User → Entity | notification_prefs |
| `SIMILAR_TO` | Entity → Entity | similarity_score, method |

---

## 4. Canonical Entity Resolution

**Problem**: Same real-world entity appears across sources with different IDs.

**Solution**: `entities` table = canonical registry. `entity_aliases` maps source IDs.

```
entities
├── id: UUID (canonical)
├── type: ENUM (RESEARCHER, PAPER, MODEL, BENCHMARK, REPOSITORY, DATASET, COMPANY, TOPIC, RELEASE)
├── canonical_name: TEXT
├── canonical_data: JSONB (merged best fields from all sources)
├── quality_score: FLOAT (0-100)
├── primary_source: ENUM (source we trust most for this entity)
└── timestamps

entity_aliases
├── entity_id: UUID (FK to entities)
├── source: ENUM (ARXIV, SEMANTIC_SCHOLAR, HF_HUB, GITHUB, CROSSREF, PAPERS_WITH_CODE, ...)
├── source_id: TEXT (original ID in that source)
├── confidence: FLOAT (0-1)
├── matched_by: ENUM (EXACT_DOI, EXACT_ARXIV, EXACT_HF_ID, EXACT_GITHUB, FUZZY_TITLE_AUTHOR, LLM_VERIFIED)
└── timestamps
```

**Resolution Priority**:
1. **DOI** → Crossref + Semantic Scholar + ArXiv (if DOI present)
2. **ArXiv ID** → ArXiv + Semantic Scholar + HF Daily Papers
3. **HF Model ID** → HF Hub + Papers with Code + GitHub (model repos)
4. **GitHub repo (owner/name)** → GitHub + HF (linked repos) + Papers with Code
5. **Fuzzy**: Title + Authors + Year → LLM verification → Human review queue

---

## 5. Change Tracking Model

For any entity with versions (Model, Repository, Paper, Benchmark):

```sql
entity_changes
├── id: UUID
├── entity_id: UUID
├── entity_type: ENUM
├── change_type: ENUM (CREATED, UPDATED, VERSION_BUMP, BREAKING_CHANGE, BENCHMARK_UPDATE, DEPRECATED, RETRACTED)
├── version_from: TEXT
├── version_to: TEXT
├── change_summary: TEXT (AI-generated)
├── structured_diff: JSONB (see Release entity)
├── significance_score: FLOAT (0-1) (breaking=1.0, major=0.7, minor=0.3, patch=0.1)
├── source_doc_id: UUID (FK to Document that triggered this)
├── detected_at: TIMESTAMPTZ
└── notified_users: UUID[] (who was alerted)
```

---

## 6. Indexing Strategy

| Query Pattern | Index |
|---------------|-------|
| Search by title/abstract | Elasticsearch: `title^3`, `abstract^2`, `full_text` |
| Filter by entity type | ES: `entity_type` keyword |
| Filter by date range | ES: `published_at` date |
| Filter by topics/tasks/benchmarks/models/companies | ES: keyword arrays |
| Vector similarity search | Qdrant: HNSW on embeddings |
| Author → Papers | PostgreSQL: GIN index on `authors` array |
| Model → Benchmarks | PostgreSQL: GIN on `benchmark_scores` JSONB |
| Paper → Citations | PostgreSQL: GIN on `citations` / `cited_by` arrays |
| User follows | PostgreSQL: composite index `(user_id, entity_type)` |
| Interest vector search | Qdrant: user interest vector collection |

---

## 7. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-10-06 | Initial data model |

---

*This document informs `SYSTEM_ARCHITECTURE.md` (database schema), `REQUIREMENTS.md` (FR-ING-009), and `DATA_SOURCE_STRATEGY.md` (normalization targets).*