# ADR-0006: Agent Boundaries: Deterministic by Default, Agents for Open-Ended Tasks

**Status**: Accepted
**Date**: 2026-10-06
**Deciders**: Rishi Sakhija
**Technical Story**: When to use agents vs. deterministic code

## Context

LLM agents are powerful but introduce non-determinism, cost, latency, and security risks. We need clear boundaries on where agents are justified.

## Decision

**Principle**: Deterministic by default. Agents only where they solve problems that deterministic code cannot.

### Agent-Justified Tasks (Open-Ended, Judgment Required)

| Task | Why Agent |
|------|-----------|
| **Research Planning** | Decomposing ambiguous questions into sub-questions requires reasoning |
| **Source Research** | Choosing which sources, queries, and filters for a sub-question |
| **Entity Resolution (Fuzzy)** | Judging "same entity" across messy real-world data |
| **Classification/Extraction** | Understanding context to extract structured facts from unstructured text |
| **Change Detection (Semantic)** | Interpreting "breaking change" vs "minor fix" in changelogs |
| **Synthesis** | Combining evidence from multiple sources into coherent answer |
| **Verification** | Checking if a claim is supported by cited evidence |

### Deterministic Tasks (No Agent)

| Task | Deterministic Approach |
|------|------------------------|
| Ingestion scheduling | Cron / Temporal schedules |
| Document normalization | Pure functions (schema mapping) |
| Exact deduplication | Key lookup (DOI, ArXiv ID, HF ID) |
| BM25 / Vector search | Algorithm + index |
| Cross-encoder reranking | Batched model inference |
| Ranking score computation | Pure function (weighted sum) |
| Metadata filtering | ES/PostgreSQL query |
| Database migrations | Alembic |
| Auth / Rate limiting | Standard libraries |
| Health checks | HTTP probes |

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **Agent-first (LangGraph for everything)** | Uniform paradigm; flexible | Unpredictable; expensive; hard to debug; security risk |
| **No agents (pure deterministic)** | Simple; fast; cheap | Can't handle open-ended research; no fuzzy judgment |
| **Agents for "AI" tasks only** | Clear boundary | "AI" is vague; classification/extraction borderline |

## Rationale

- **Cost control**: Deterministic code is ~1000x cheaper per operation
- **Reliability**: Deterministic = testable, reproducible, debuggable
- **Security**: Agents consume untrusted content → prompt injection risk
- **Latency**: Deterministic < 10ms; Agent > 1s
- **Observability**: Deterministic logs are clear; agent traces are complex
- **Maintainability**: Pure functions > prompt engineering

## Consequences

### Positive
- Clear architecture: agents only in Research Service + Ingestion Enrichment
- Predictable costs (agents only on user-initiated research + batch enrichment)
- Security surface minimized (agents sandboxed)
- Team can build deterministic expertise first

### Negative
- Some tasks (classification) could go either way — need judgment
- Ingestion enrichment uses agents → adds latency to indexing
- Research service more complex than pure search

### Risks
- **Scope creep**: "Let's use an agent for X" — enforce via code review + this ADR
- **Mitigation**: PR template requires "Why not deterministic?" for new agent tasks

## Implementation Notes

- **Agent Framework**: Custom orchestrator (not LangChain/LangGraph) for control
- **Tool Whitelisting**: Each agent type has explicit allowed tools (see `AGENT_ARCHITECTURE.md`)
- **Validation Pipeline**: Every agent output → schema validation → citation check → confidence threshold
- **Fallback**: Every agent task has deterministic alternative (e.g., keyword search)

## Related ADRs

- ADR-0001: Search Architecture (reranker = deterministic model inference)
- ADR-0004: Ingestion Architecture (enrichment stage uses agents)