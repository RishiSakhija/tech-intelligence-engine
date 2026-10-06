# Agent Architecture

> Defines where agents belong, where they do NOT belong, and the boundaries of agentic capabilities in the system.

---

## 1. Agent Philosophy

> **Agents can propose and execute bounded tasks, but product truth and architectural authority remain governed by the project specification and human review.**

### Core Principles

| Principle | Rationale |
|-----------|-----------|
| **Deterministic by default** | Pipelines (ingestion, indexing, ranking) are reproducible; agents only where non-determinism adds value |
| **Bounded tools only** | Agents cannot: write to production DB, make external API calls beyond whitelisted sources, execute arbitrary code |
| **Human-in-the-loop for truth** | Agent output = *draft*; user verifies; corrections feed back |
| **Cost control** | Per-task token budget; hard step limits; observability on every call |
| **Fallback always exists** | Every agent task has a non-agent alternative (e.g., keyword search) |
| **Observability first** | Every agent step logged: prompt, tools, output, latency, cost, confidence |

---

## 2. Where Agents DO Belong

### 2.1 Agent Roles (Validated)

| Agent | Purpose | Inputs | Outputs | When It Runs | Deterministic? |
|-------|---------|--------|---------|--------------|----------------|
| **Research Planner** | Decompose complex question into sub-questions + source plan | User question + context | Structured plan (sub-questions, sources, tools, estimated steps) | On "Deep Research" request | LLM-based |
| **Source Research Agent** | Execute search + extraction for a sub-question across assigned sources | Sub-question + source list | Raw evidence snippets + citations | Parallel per sub-question | LLM-based (search) + deterministic (extraction) |
| **Entity Resolution Agent** | Normalize same entity across sources (model on HF + ArXiv + GitHub) | Candidate entities from multiple sources | Unified entity + confidence + merge decisions | During research synthesis; batch during ingestion | LLM-based (fuzzy match) |
| **Classification Agent** | Classify document: type, topics, tasks, benchmarks, entities | Document (title, abstract, metadata) | Structured labels + confidence | During ingestion (every document) | LLM-based (can be distilled to classifier) |
| **Entity Extraction Agent** | Extract structured facts: benchmark scores, model specs, architecture details | Document text + schema | Structured facts (JSON) + citations | During ingestion (high-value docs) | LLM-based |
| **Change Detection Agent** | Semantic diff: "What changed?" between versions | Old version + new version (changelog, model card, paper) | Structured diff (breaking, new, deprecated, benchmarks) | On new release/model version/paper version | LLM-based |
| **Verification Agent** | Check citations resolve; flag unverified claims; score confidence | Synthesized answer + evidence pack | Verified answer + confidence scores + flagged claims | Before research output to user | Deterministic (citation check) + LLM (claim verification) |
| **Deep Research Agent** | Orchestrate multi-step research; produce final report | User question | Structured report (summary, tables, evidence, gaps) | On "Deep Research" request | LLM-based (orchestration) |
| **Evaluation Agent** | Judge search quality / research quality against gold labels | Query + results + gold labels | Quality scores + error analysis | Offline evaluation runs | LLM-based (as judge) |

---

## 3. Where Agents Do NOT Belong

| Task | Why Not Agent | Correct Approach |
|------|---------------|------------------|
| **Ingestion scheduling** | Deterministic, periodic, well-defined | Cron / Airflow / Temporal |
| **Document normalization** | Schema mapping is deterministic | Pure functions + unit tests |
| **Deduplication (DOI/ArXiv ID)** | Exact match logic | Deterministic key lookup |
| **BM25 / Vector search** | Algorithmic; latency-critical | Optimized search engines |
| **Reranking (cross-encoder)** | Fixed model inference | Batched model serving |
| **Ranking score computation** | Formula application | Pure function |
| **Database migrations** | Schema changes require review | Alembic + CI |
| **API request routing** | Deterministic; latency-critical | FastAPI / Gateway |
| **Authentication/Authorization** | Security-critical; standard patterns | Auth library (Authlib, etc.) |
| **Rate limiting** | Algorithmic | Token bucket / Redis |
| **Health checks** | Deterministic probes | Standard endpoints |

**Rule**: If you can write a pure function with unit tests for it, it's not an agent task.

---

## 4. Agent Boundaries & Guardrails

### 4.1 Tool Whitelisting

Each agent gets a **strictly defined toolset**:

```python
# Example: Source Research Agent Tools
ALLOWED_TOOLS = {
    "search_arxiv": {"max_results": 50, "date_range": "last_year"},
    "search_github": {"max_results": 30, "repos": "whitelisted"},
    "search_hf_hub": {"max_results": 30, "types": ["model", "paper"]},
    "search_papers_with_code": {"max_results": 20},
    "fetch_document": {"max_length": 50000, "allowed_domains": WHITELIST},
    "extract_structured": {"schema": FACT_SCHEMA},  # LLM tool
}
```

**Prohibited for ALL agents**:
- `write_database`, `delete_database`, `execute_sql`
- `http_request` to non-whitelisted domains
- `execute_code`, `shell_command`
- `send_email`, `send_notification` (except via internal queue)
- `modify_index`, `reindex`

### 4.2 Execution Constraints

| Constraint | Value | Enforcement |
|------------|-------|-------------|
| **Max steps per task** | 10 (planner), 5 (research), 3 (extraction) | Step counter in orchestrator |
| **Token budget per task** | 50k (planner), 100k (deep research), 10k (extraction) | Token counter |
| **Max parallel agents** | 5 (research), 3 (ingestion enrichment) | Semaphore |
| **Timeout per step** | 30s (search), 60s (LLM), 10s (fetch) | Async timeout |
| **Retry limit** | 2 (transient errors only) | Retry policy |

### 4.3 Output Validation

Every agent output passes through **validation pipeline**:

```python
def validate_agent_output(output: AgentOutput, schema: Schema) -> ValidationResult:
    # 1. Schema validation (Pydantic)
    # 2. Citation verification (do cited URLs resolve?)
    # 3. Confidence threshold (reject if avg < 0.6)
    # 4. Hallucination check (sample claims vs. evidence)
    # 5. Cost check (within budget?)
    pass
```

---

## 5. Agent Orchestration

### 5.1 Deep Research Flow (Example)

```
USER QUESTION
    │
    ▼
┌─────────────────────┐
│  RESEARCH PLANNER   │  → Plan: 4 sub-questions, 12 sources, est. 8 steps
│  (1 step, 50k tok)  │
└─────────┬───────────┘
          │
    ┌─────┴─────┬─────┬─────┐
    ▼           ▼     ▼     ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ Agent 1│ │ Agent 2│ │ Agent 3│ │ Agent 4│  (Parallel, 5 steps each, 20k tok)
│ ArXiv  │ │ GitHub │ │ HF Hub │ │ Papers │
│ Search │ │ Search │ │ Search │ │ w Code │
└────┬───┘ └────┬───┘ └────┬───┘ └────┬───┘
     │          │          │          │
     └──────────┼──────────┼──────────┘
                ▼
    ┌─────────────────────┐
    │  SYNTHESIS AGENT    │  → Merge entities, build tables, draft narrative
    │  (3 steps, 50k tok) │
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────┐
    │  VERIFICATION AGENT │  → Check citations, flag unverified, score confidence
    │  (2 steps, 10k tok) │
    └─────────┬───────────┘
              │
              ▼
         FINAL REPORT
```

### 5.2 Ingestion Enrichment Flow

```
RAW DOCUMENT
    │
    ▼
┌─────────────────────┐
│  CLASSIFICATION     │  → Type: PAPER/MODEL/RELEASE/BENCHMARK
│  AGENT (1 step)     │     Topics: [MoE, Quantization, ...]
└─────────┬───────────┘     Tasks: [Coding, Reasoning]
          │
          ▼
┌─────────────────────┐
│  ENTITY EXTRACTION  │  → Benchmarks: [{"name": "MMLU", "score": 0.78}]
│  AGENT (2 steps)    │     Models: [{"name": "Llama-3.1-70B", "params": "70B"}]
└─────────┬───────────┘     Researchers: [{"name": "Yann LeCun", "affiliation": "Meta"}]
          │
          ▼
┌─────────────────────┐
│  ENTITY RESOLUTION  │  → Match to existing entities in KG
│  AGENT (2 steps)    │     Assign canonical IDs
└─────────┬───────────┘
          │
          ▼
    ENRICHED DOCUMENT → INDEX
```

---

## 6. LLM Strategy for Agents

### 6.1 Model Selection by Task

| Task | Model Requirements | Primary Choice | Fallback |
|------|-------------------|----------------|----------|
| **Planning** | Strong reasoning, structured output | `gpt-4o` / `claude-3-5-sonnet` | `gpt-4o-mini` |
| **Extraction** | Instruction following, JSON mode | `gpt-4o-mini` / `claude-3-5-haiku` | Local (`llama-3.1-8b-instruct`) |
| **Classification** | Speed, low cost, can be distilled | `gpt-4o-mini` → distill to BERT | `bge-reranker` as classifier |
| **Verification** | Careful reasoning, citation check | `gpt-4o` / `claude-3-5-sonnet` | `gpt-4o-mini` |
| **Synthesis** | Long context, structured output | `gpt-4o` / `claude-3-5-sonnet` (128k ctx) | `gemini-1.5-pro` |
| **Change Detection** | Diff reasoning, structured output | `gpt-4o` / `claude-3-5-sonnet` | `gpt-4o-mini` |

### 6.2 Cost Management

| Strategy | Implementation |
|----------|----------------|
| **Model routing** | Route simple tasks to cheaper models |
| **Prompt caching** | Cache planner prompts; reuse for similar questions |
| **Batch processing** | Ingestion enrichment: batch 10-20 docs per LLM call |
| **Distillation** | Classification/extraction → train smaller specialist models |
| **Token budgets** | Hard limits per agent type; fail fast if exceeded |
| **Usage logging** | Every call: model, tokens, cost, latency, task_id |

---

## 7. Agent Evaluation

### 7.1 Quality Metrics

| Agent | Metric | Target |
|-------|--------|--------|
| **Planner** | Plan completeness (covers all aspects) | >90% human rating |
| **Source Research** | Recall@10 per sub-question | >85% |
| **Entity Resolution** | Precision / Recall (vs. human labels) | >90% / >85% |
| **Classification** | F1 (multi-label) | >0.85 |
| **Extraction** | Fact accuracy (vs. ground truth) | >95% |
| **Change Detection** | Breaking change detection F1 | >85% |
| **Verification** | False positive rate (flagging correct claims) | <5% |
| **Deep Research** | Human evaluation (accuracy, completeness, citation quality) | >4.0/5.0 |

### 7.2 Regression Testing

- Golden set: 100 research questions with expected outputs
- Run nightly; alert on quality regression
- Cost tracking: alert if cost/query > 2x baseline

---

## 8. Security Considerations (See `SECURITY.md`)

### 8.1 Agent-Specific Risks

| Risk | Mitigation |
|------|------------|
| **Prompt Injection** | System prompts immutable; user input sanitized; tool outputs validated |
| **Data Exfiltration** | No external network access; tools whitelisted; output size limited |
| **Resource Exhaustion** | Step/token/time limits; circuit breakers |
| **Hallucinated Citations** | Verification agent; citation resolution check |
| **Biased Synthesis** | Multiple agent perspectives; human review for high-stakes |

### 8.2 Agent Sandbox

- Separate process/container per agent type
- No filesystem access
- No environment variables except config
- Network egress only to whitelisted APIs via internal proxy

---

## 9. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-10-06 | Initial agent architecture |

---

*This document informs `REQUIREMENTS.md` (FR-RES-*), `SYSTEM_ARCHITECTURE.md` (Research Service), `SECURITY.md`, and `ROADMAP.md` (Phase 6).*