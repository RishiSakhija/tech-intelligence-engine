# ADR-0005: Entity Resolution Strategy: Exact Keys First, Fuzzy Later

**Status**: Accepted
**Date**: 2026-10-06
**Deciders**: Rishi Sakhija
**Technical Story**: Cross-source entity deduplication

## Context

Same real-world entities appear across multiple sources with different identifiers:
- Paper: ArXiv ID, DOI, Semantic Scholar ID, HF Daily Papers ID, Papers with Code ID
- Model: HF Model ID, GitHub repo, Paper reference, Blog mention
- Researcher: Semantic Scholar ID, ORCID, GitHub username, HF username, ArXiv author name
- Repository: GitHub owner/name, HF Space, Papers with Code repo link

We need a canonical entity registry (`entities` table) with aliases mapping to source IDs.

## Decision

**Tiered resolution strategy**:

1. **Exact Key Matching** (automatic, high confidence):
   - DOI → Crossref + Semantic Scholar + ArXiv (if DOI present)
   - ArXiv ID → ArXiv + Semantic Scholar + HF Daily Papers
   - HF Model ID (org/name) → HF Hub + Papers with Code + GitHub (model repos)
   - GitHub Repo (owner/name) → GitHub + HF Spaces + Papers with Code
   - ORCID → Semantic Scholar + Crossref

2. **Fuzzy Matching** (queued for LLM-assisted verification):
   - Title + Authors + Year (papers)
   - Model name + Author/Org (models)
   - Author name + Affiliation (researchers)

3. **Human Review Queue** (low confidence):
   - Ambiguous matches
   - Conflicting metadata

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **Pure fuzzy (LLM for all)** | Uniform approach | Expensive; slow; unnecessary for exact keys |
| **Pure exact keys only** | Fast; deterministic | Misses many matches (no DOI on ArXiv v1, name variations) |
| **ML-based entity resolution (dedupe.io, Zingg)** | Scalable; learned | Training data needed; black box; overkill |
| **Probabilistic (Fellegi-Sunter)** | Statistically sound | Complex; requires tuning; hard to explain |

## Rationale

- **Exact keys cover 70-80%** of matches automatically (DOI, ArXiv ID, HF ID, GitHub repo)
- **Fuzzy only for remainder** — reduces LLM calls by 5-10x
- **LLM verification** for fuzzy: structured prompt with both records → confidence score
- **Human-in-loop** for edge cases prevents error propagation
- **Canonical entity** stores merged best fields (most complete abstract, best author list, etc.)

## Consequences

### Positive
- High precision on automatic matches (>99%)
- Cost-controlled LLM usage (only fuzzy remainder)
- Audit trail: every alias has `matched_by` method
- Incremental: new sources add aliases to existing entities

### Negative
- Two-phase process adds latency for fuzzy matches
- Requires maintaining key extractors per source
- Schema evolution when new key types appear

### Risks
- **False merges**: Different papers with similar titles/authors
- **Mitigation**: Conservative fuzzy threshold; LLM verification; human review for conflicts
- **Missing merges**: Same entity, no shared keys, fuzzy fails
- **Mitigation**: Periodic re-clustering; user-reported merges

## Implementation Notes

- `entity_aliases` table: `entity_id`, `source`, `source_id`, `confidence`, `matched_by`
- `matched_by` enum: `EXACT_DOI`, `EXACT_ARXIV`, `EXACT_HF_ID`, `EXACT_GITHUB`, `EXACT_ORCID`, `FUZZY_TITLE_AUTHOR`, `FUZZY_MODEL_NAME`, `LLM_VERIFIED`, `HUMAN_VERIFIED`
- Resolution runs after normalization, before enrichment
- LLM verification prompt: structured comparison of key fields → JSON with `match: boolean`, `confidence: 0-1`, `reasoning`
- Human review UI: side-by-side comparison with accept/reject/flag

## Related ADRs

- ADR-0004: Ingestion Architecture
- ADR-0009: Primary Database (entities table)