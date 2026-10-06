# ADR-0009: Primary Database: PostgreSQL with JSONB

**Status**: Accepted
**Date**: 2026-10-06
**Deciders**: Rishi Sakhija
**Technical Story**: Primary database selection

## Context

We need a primary database for:
- User accounts, auth, API keys, settings
- Canonical entities + aliases (entity registry)
- Documents + metadata
- Relationships (entity graph)
- Personalization (follows, interest vectors, history)
- Ingestion logs, dead letter queue
- Research tasks + reports

Requirements:
- ACID transactions (user data, entity merges)
- JSONB for flexible schemas (entity metadata, document metadata, relationships)
- Mature, stable, well-understood
- Horizontal scaling path (read replicas, partitioning)
- Strong Python async support (asyncpg)
- Open-source, self-hostable

## Decision

**Choose PostgreSQL 16+ with JSONB for flexible fields**.

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **MongoDB** | Flexible schema; horizontal scaling | No ACID transactions (pre-4.0); weaker consistency; operational complexity |
| **MySQL 8+** | Mature; JSON support | JSON less powerful than JSONB; no GIN indexes on JSON |
| **CockroachDB** | Distributed SQL; horizontal scaling | Complex; latency; overkill for Phase 1-3 |
| **SQLite** | Simple; embedded | No concurrency; no horizontal scaling; dev only |
| **DynamoDB** | Serverless; scale | Vendor lock-in; no joins; complex queries hard |
| **Redis + PostgreSQL** | Redis for cache/vectors | Already using Redis; keep PostgreSQL as source of truth |

## Rationale

- **PostgreSQL** is the gold standard for relational + semi-structured data
- **JSONB + GIN indexes** = fast queries on flexible metadata (topics[], benchmarks[], entity relationships)
- **asyncpg** = high-performance async driver for FastAPI
- **Alembic** = mature migration tool
- **Read replicas** = simple horizontal read scaling
- **Partitioning** = time-based partitioning for ingestion logs, search history
- **Extensions**: `pgvector` (if we want vectors in PG), `pg_trgm` (fuzzy text), `btree_gin` (composite JSONB indexes)

## Consequences

### Positive
- Single source of truth for all relational + document data
- Strong consistency for entity resolution, user data
- Rich query capability (SQL + JSONB paths)
- Operational maturity (backups, PITR, monitoring, tuning)
- Talent availability

### Negative
- Vertical scaling limits (mitigate with read replicas + partitioning)
- JSONB not as flexible as MongoDB for deeply nested varying schemas
- Write throughput single-primary (mitigate: batch writes, async workers)

### Risks
- **JSONB query performance**: GIN indexes can be large
- **Mitigation**: Selective indexing; `jsonb_path_ops` for containment queries
- **Connection pooling**: Required at scale (PgBouncer)

## Implementation Notes

- **Schema**: See `DATA_MODEL.md` for tables
- **Migrations**: Alembic; backward-compatible only
- **Connection pool**: asyncpg pool (min=10, max=50)
- **Read replicas**: For search API reads (eventual consistency OK)
- **Partitioning**: `ingestion_log` by month; `user_search_history` by month
- **Backup**: Daily pg_dump + WAL archiving (PITR)
- **Monitoring**: `pg_stat_statements`; slow query log; connection pool usage

## Related ADRs

- ADR-0004: Ingestion Architecture (metadata storage)
- ADR-0005: Entity Resolution (entities table)
- ADR-0007: Personalization (follows, interest vectors)