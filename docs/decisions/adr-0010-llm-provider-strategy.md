# ADR-0010: LLM Provider Strategy: Multi-Provider with Cost Routing

**Status**: Accepted
**Date**: 2026-10-06
**Deciders**: Rishi Sakhija
**Technical Story**: LLM provider selection and routing

## Context

Agents and enrichment pipelines need LLM access for:
- Research planning, synthesis, verification
- Classification, entity extraction, change detection
- Embedding (separate, self-hosted)
- Reranking (separate, self-hosted)

Requirements:
- Cost optimization (different models for different tasks)
- Redundancy (no single provider outage blocks all agents)
- Data privacy option (local models for sensitive content)
- Latency SLAs (fast models for user-facing, slower for batch)
- Token budget enforcement per task

## Decision

**Multi-provider strategy with task-based routing**:

| Task Category | Primary | Fallback | Local Option |
|---------------|---------|----------|--------------|
| **Planning / Synthesis / Verification** (complex reasoning) | `gpt-4o` / `claude-3-5-sonnet` | `gpt-4o-mini` / `claude-3-5-haiku` | `llama-3.1-70b-instruct` (vLLM) |
| **Extraction / Classification** (structured output) | `gpt-4o-mini` / `claude-3-5-haiku` | `gemini-1.5-flash` | `llama-3.1-8b-instruct` (Ollama) |
| **Simple QA / Formatting** | `gpt-4o-mini` | `gemini-1.5-flash` | `phi-3-mini` |

**Routing Logic**: Configurable per agent type; cost-aware (prefer cheaper if quality sufficient); fallback on error/timeout.

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **Single provider (OpenAI only)** | Simple; best models | Vendor lock-in; outage risk; cost |
| **Single provider (Anthropic only)** | Strong reasoning; large context | Same risks; no GPT-4o equivalent for some tasks |
| **Local only (Ollama/vLLM)** | Privacy; zero marginal cost | Quality gap on complex tasks; GPU ops burden |
| **OpenRouter only** | Unified API; many models | Adds latency; extra dependency; cost markup |

## Rationale

- **Task-based routing** matches model capability to task needs (don't use GPT-4o for classification)
- **Cost control**: 10x cost difference between `gpt-4o` and `gpt-4o-mini`; route aggressively
- **Redundancy**: Provider outage → automatic fallback (circuit breaker pattern)
- **Privacy path**: Local models for enterprise / sensitive research future
- **Experimentation**: Easy to swap models as landscape evolves

## Consequences

### Positive
- Optimized cost/quality per task
- Resilience to provider outages
- Future-proof (swap models as landscape evolves)
- Local option for compliance

### Negative
- Complexity: multiple APIs, auth, rate limits, response formats
- Prompt portability: prompts tuned per model family
- Observability: need unified logging across providers

### Risks
- **Prompt drift**: Model updates change behavior
- **Mitigation**: Version prompts; regression tests on golden set
- **Cost surprise**: Runaway agent loops
- **Mitigation**: Hard token budgets per task; alerting on spend

## Implementation Notes

- **Abstraction layer**: `LLMClient` interface with `complete()`, `complete_structured()`, `stream()`
- **Providers**: `OpenAIClient`, `AnthropicClient`, `GoogleClient`, `LocalClient` (vLLM/Ollama compatible)
- **Router**: `TaskRouter` selects provider/model based on `task_type`, `priority`, `privacy_level`
- **Budgets**: Per-task `max_tokens`, `max_cost_usd`, `max_latency_ms`
- **Logging**: Every call → structured log (provider, model, tokens, cost, latency, task_id, success)
- **Caching**: Prompt + response cache for deterministic tasks (classification, extraction)

## Related ADRs

- ADR-0006: Agent Boundaries (which tasks use LLMs)
- ADR-0003: Embedding Model (separate, self-hosted)
- ADR-0004: Ingestion Architecture (enrichment uses LLMs)