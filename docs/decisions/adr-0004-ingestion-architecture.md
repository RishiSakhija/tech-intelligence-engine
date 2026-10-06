# ADR-0004: Ingestion Pipeline: Async Workers with Message Queue

**Status**: Accepted
**Date**: 2026-10-06
**Deciders**: Rishi Sakhija
**Technical Story**: Ingestion architecture

## Context

We need to ingest from 10+ sources (ArXiv, GitHub, HF Hub, RSS, Papers with Code, Semantic Scholar, Crossref, docs sites) with different patterns: polling, webhooks, batch API. Requirements:

- High throughput (10k+ docs/hour)
- Low latency for primary sources (<1 hour median)
- Fault tolerance (retries, dead letter queue)
- Observability (lag, errors, volume)
- Independent scaling per source
- Exactly-once or at-least-once semantics

## Decision

**Async worker architecture with Redis Streams as message queue**.

```
Source Connectors → Fetch Workers → Redis Streams (per stage)
                                    ↓
                              Normalize Workers
                                    ↓
                              Dedupe Workers
                                    ↓
                              Enrich Workers (LLM)
                                    ↓
                              Index Workers (batched)
```

Each stage is a separate worker pool consuming from a Redis Stream, producing to the next stream.

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **Celery + Redis/RabbitMQ** | Mature; Python-native; retries built-in | Less visibility into stream; no native replay |
| **Temporal** | Durable execution; retries; visibility; saga support | Heavier; learning curve; overkill for linear pipeline |
| **Airflow/Dagster** | Great for batch/ETL; scheduling; UI | Not designed for streaming/real-time; high latency |
| **Kafka** | High throughput; replay; ecosystem | Operational complexity; JVM; overkill for our scale |
| **Custom asyncio + asyncpg** | Full control; lightweight | Reinventing queue semantics; no built-in retry/DLQ |

## Rationale

- **Redis Streams**: Native to our stack (already using Redis for cache/rate limit); consumer groups for scaling; XREADGROUP for exactly-once semantics; stream trimming for bounded memory; low latency
- **Stage separation**: Each stage independently scalable (more enrich workers for LLM; fewer fetch workers)
- **Backpressure**: Stream length metrics → auto-scaling signals
- **Replay**: Can reprocess from any point (re-index, fix bugs)
- **Simplicity**: Single Redis instance; no separate queue cluster needed at Phase 1-2 scale

## Consequences

### Positive
- Simple operational model (Redis only)
- Clear stage boundaries; testable in isolation
- Natural backpressure and scaling
- Built-in replay for re-indexing

### Negative
- Redis memory for stream retention (trim to 24h)
- No built-in workflow orchestration (Temporal better for complex DAGs)
- At-least-once by default; exactly-once requires idempotent consumers

### Risks
- **Redis OOM**: Stream trimming + monitoring alerts
- **Mitigation**: Max stream length 1M messages; TTL 24h; alert at 80%
- **Worker crashes mid-processing**: Idempotency keys per document (source + source_id + content_hash)

## Implementation Notes

- Stream naming: `ingest:fetch`, `ingest:normalize`, `ingest:dedupe`, `ingest:enrich`, `ingest:index`
- Consumer groups: `fetch-workers`, `normalize-workers`, etc.
- Message format: JSON with `document_id`, `payload`, `metadata`, `retry_count`
- Dead letter: `ingest:dlq` after 3 retries
- Monitoring: Stream lag (consumer group lag), processing latency, error rate per stage

## Related ADRs

- ADR-0005: Entity Resolution Strategy
- ADR-0009: Primary Database (PostgreSQL for metadata)