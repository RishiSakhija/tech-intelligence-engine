# ADR-0007: Personalization: Explicit Follows + Implicit Interest Vectors

**Status**: Accepted
**Date**: 2026-10-06
**Deciders**: Rishi Sakhija
**Technical Story**: Personalization architecture

## Context

We need personalization that actually improves search/discovery, not just UI decoration. Requirements:

- Explicit user control (follow/unfollow entities)
- Implicit learning from behavior (clicks, dwell, research)
- Transparent ("Why this result?")
- Privacy-preserving (export, delete, reset)
- Ranking influence measurable (A/B testable)

## Decision

**Dual representation**:

1. **Explicit Entity Graph** (`user_follows` table): User follows specific entities (Researcher, Company, Model, Repository, Benchmark, Topic). Weight = 10.0 (no decay).

2. **Implicit Interest Vector** (1024-dim, same space as document embeddings): Weighted average of interacted document vectors. Updated online per interaction + nightly batch. Decay: 30-day half-life.

**Ranking Boost**:
```
personalization_factor = min(0.5,
    explicit_boost (0.0-0.3) +
    implicit_boost (0.0-0.2) +
    recency_boost (0.0-0.1)
)
```

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **Collaborative filtering** (user-user / item-item) | Leverages similar users | Cold start; privacy; not entity-centric |
| **Topic-only follows** (like Twitter lists) | Simple | Too coarse; "AI" follows everything |
| **Implicit only (no explicit)** | Zero friction | No user control; opaque; hard to debug |
| **Learning-to-Rank (LTR) per user** | Optimal ranking | Needs massive data per user; overkill |
| **Profile-based (demographics/role)** | Segment-level | Not personalized; stereotyping |

## Rationale

- **Entity-level follows** match mental model: "I follow Yann LeCun" not "I follow AI topic"
- **Explicit > Implicit**: 10x weight ensures user intent dominates
- **Interest vector** enables soft matching (semantic proximity) beyond exact follows
- **Same embedding space** as documents → efficient cosine similarity
- **Decay** prevents stale interests dominating
- **Transparency**: Each signal visible in "Why this result?"

## Consequences

### Positive
- Measurable lift (A/B testable)
- User trust through control + transparency
- Cold start: explicit follows work immediately
- Exportable/portable (JSON)

### Negative
- Two systems to maintain (graph + vector)
- Implicit vector quality depends on interaction volume
- Storage: one vector per user (small: 4KB)

### Risks
- **Filter bubble**: Over-personalization narrows discovery
- **Mitigation**: Serendipity slot; diversity penalty; cap boost at 50%
- **Privacy concern**: Interest vector could reveal sensitive interests
- **Mitigation**: User can reset; local-only option future; no third-party sharing

## Implementation Notes

- `user_follows`: (user_id, entity_id, entity_type, notification_prefs, created_at)
- `user_interest_vector`: (user_id, vector[1024], updated_at) — binary or float32
- Online update: `v = (1-α)v + α*doc_vector` where `α = weight * decay`
- Nightly recompute: weighted average of last 90 days interactions
- Ranking: compute `cosine_sim(query_vector, user_vector)` at query time (cached)

## Related ADRs

- ADR-0001: Search Architecture (personalization boost in ranking)
- ADR-0003: Embedding Model (same space for interest vectors)