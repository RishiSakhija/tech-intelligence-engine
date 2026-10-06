# Architecture Decision Records (ADRs)

> This directory contains Architecture Decision Records for Tech Intelligence Engine.

---

## ADR Format

Each ADR follows this structure:

```markdown
# ADR-XXXX: Short Title

**Status**: Proposed | Accepted | Superseded | Rejected
**Date**: YYYY-MM-DD
**Deciders**: [Names]
**Technical Story**: [Link to issue/PR]

## Context

What is the issue that motivates this decision? What are the constraints?

## Decision

What is the change we're proposing or have decided to do?

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| Option A | ... | ... |
| Option B | ... | ... |

## Rationale

Why did we choose this option? What evidence supports it?

## Consequences

### Positive
- ...

### Negative
- ...

### Risks
- ...

## Implementation Notes

Any follow-up tasks, migration steps, or technical details.

## Related ADRs

- ADR-XXXX: Related decision
```

---

## ADR Index

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [ADR-0001](adr-0001-search-architecture.md) | Hybrid Search Architecture (Lexical + Semantic + Metadata) | Accepted | 2026-10-06 |
| [ADR-0002](adr-0002-vector-database.md) | Vector Database Selection: Qdrant | Accepted | 2026-10-06 |
| [ADR-0003](adr-0003-embedding-model.md) | Embedding Model: BAAI/bge-large-en-v1.5 | Accepted | 2026-10-06 |
| [ADR-0004](adr-0004-ingestion-architecture.md) | Ingestion Pipeline: Async Workers with Message Queue | Accepted | 2026-10-06 |
| [ADR-0005](adr-0005-entity-resolution.md) | Entity Resolution Strategy: Exact Keys First, Fuzzy Later | Accepted | 2026-10-06 |
| [ADR-0006](adr-0006-agent-boundaries.md) | Agent Boundaries: Deterministic by Default, Agents for Open-Ended Tasks | Accepted | 2026-10-06 |
| [ADR-0007](adr-0007-personalization-approach.md) | Personalization: Explicit Follows + Implicit Interest Vectors | Accepted | 2026-10-06 |
| [ADR-0008](adr-0008-frontend-framework.md) | Frontend Framework: Next.js 14+ (App Router, React 18) | Accepted | 2026-10-06 |
| [ADR-0009](adr-0009-primary-database.md) | Primary Database: PostgreSQL with JSONB | Accepted | 2026-10-06 |
| [ADR-0010](adr-0010-llm-provider-strategy.md) | LLM Provider Strategy: Multi-Provider with Cost Routing | Accepted | 2026-10-06 |

---

## Creating New ADRs

1. Copy the template above
2. Name file: `adr-NNNN-short-title.md` (next sequential number)
3. Fill in all sections
4. Add entry to index above
5. Submit for review (PR)
6. On acceptance: update status to "Accepted"

## When to Create an ADR

Create an ADR when making a decision that:
- Affects multiple components/services
- Has significant trade-offs
- Is difficult to reverse
- Involves technology selection
- Defines architectural patterns
- Impacts security, scalability, or maintainability

**Don't** create ADRs for:
- Tactical code decisions
- Reversible configuration changes
- Decisions already documented in requirements/design docs

---

## Superseded ADRs

When an ADR is superseded:
1. Update old ADR status to "Superseded by ADR-NNNN"
2. New ADR references old one in "Related ADRs"
3. Keep both for history

---

*This ADR process ensures architectural decisions are explicit, reviewable, and traceable.*