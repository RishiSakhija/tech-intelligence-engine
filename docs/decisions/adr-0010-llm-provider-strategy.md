# ADR-0010: LLM Provider Strategy — Local-First, Provider-Agnostic, API-Optional

**Status**: Accepted
**Date**: 2026-10-06
**Deciders**: Rishi Sakhija
**Technical Story**: LLM provider selection and routing for zero-budget development

## Context

Agents and enrichment pipelines need LLM access for:
- Research planning, synthesis, verification
- Classification, entity extraction, change detection
- Embedding (separate, self-hosted — ADR-0003)
- Reranking (separate, self-hosted — ADR-0003)

**Hard Constraint**: Development and MVP validation must be achievable with a **$0 infrastructure/API budget** using open-source software, local execution, and genuinely free public APIs/services where available.

## Decision

**Primary Principle**: **Local-first, provider-agnostic, API-optional.**

The application MUST have an abstraction layer so models/providers can be swapped without code changes.

**Preferred Development Order**:
1. **Local open-source models** (Ollama, vLLM, llama.cpp) — primary for development
2. **Free hosted models/APIs** — only when genuinely available and genuinely free (no credit card, no rate limits that block development)
3. **Paid providers** — only as future optional upgrades for production; never a hard dependency for MVP

**Do NOT** hardcode OpenAI/Anthropic/Google/etc. as mandatory project dependencies.

**Do NOT** claim any current free provider is permanent.

## Provider Abstraction Layer

The application defines an `LLMClient` interface:

```python
class LLMClient(ABC):
    @abstractmethod
    async def complete(self, prompt: str, **kwargs) -> str: ...
    @abstractmethod
    async def complete_structured(self, prompt: str, schema: Type[BaseModel], **kwargs) -> BaseModel: ...
    @abstractmethod
    async def stream(self, prompt: str, **kwargs) -> AsyncIterator[str]: ...
```

Implementations (pluggable):
- `LocalClient` — Ollama/vLLM/llama.cpp (primary for development)
- `OpenAIClient` — optional, future
- `AnthropicClient` — optional, future
- `GoogleClient` — optional, future
- `OpenRouterClient` — optional, future

A `TaskRouter` selects implementation based on config: `task_type`, `priority`, `privacy_level`, `budget_usd`.

## Preferred Model Selection (Local-First)

| Task Category | Primary (Local) | Fallback (Local) | Notes |
|---------------|-----------------|------------------|-------|
| **Planning / Synthesis / Verification** (complex reasoning) | `llama-3.1-70b-instruct` (vLLM/Ollama) | `llama-3.1-8b-instruct` | Requires GPU RAM; 70B for quality, 8B for speed |
| **Extraction / Classification** (structured output) | `llama-3.1-8b-instruct` / `qwen2.5-7b-instruct` | `phi-3-mini` | 8B sufficient for structured output |
| **Simple QA / Formatting** | `phi-3-mini` / `gemma-2-2b` | — | Small models sufficient |

**No paid API models in the primary column.**

## Routing Logic

Configurable per agent type via YAML/JSON config:

```yaml
router:
  default_provider: "local"
  providers:
    local:
      endpoint: "http://localhost:11434"  # Ollama
      models:
        planner: "llama-3.1-70b-instruct"
        extractor: "llama-3.1-8b-instruct"
        formatter: "phi-3-mini"
    # openai: {}  # optional, disabled by default
    # anthropic: {}  # optional, disabled by default
```

**Fallback on error/timeout** → next provider in config order.

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **Single provider (OpenAI only)** | Simple; best models | Violates $0 constraint; vendor lock-in; outage risk |
| **Single provider (Anthropic only)** | Strong reasoning | Same as above |
| **Local only (Ollama/vLLM)** | Privacy; zero marginal cost | Quality gap on complex tasks; GPU ops burden |
| **OpenRouter only** | Unified API; many models | Adds latency; extra dependency; cost markup; not $0 |

## Rationale

- **$0 budget constraint is non-negotiable** for MVP development
- **Local-first** ensures development continues without internet/API keys
- **Provider-agnostic** abstraction prevents vendor lock-in
- **API-optional** means paid providers are truly optional upgrades
- **Task-based routing** matches model capability to task needs
- **No hardcoded dependencies** on any external provider

## Consequences

### Positive
- Development works offline / without API keys
- Zero marginal cost per LLM call (local)
- Full control over model versions, prompts, fine-tuning
- Resilience to provider outages / policy changes
- Future-proof: swap models as landscape evolves

### Negative
- Local models require GPU RAM (70B needs ~48GB; 8B needs ~8GB)
- Quality gap vs. frontier models on complex reasoning
- GPU ops burden (local inference setup)
- Prompt portability: prompts may need tuning per model family

### Risks
- **Prompt drift**: Model updates change behavior → Version prompts; regression tests on golden set
- **Local model quality insufficient** for complex tasks → Mitigation: Defer complex agent tasks until local models improve; use structured output + verification
- **GPU unavailable** → Mitigation: CPU inference with smaller models (phi-3-mini, gemma-2-2b); defer heavy tasks

## Implementation Notes

- **Abstraction layer**: `LLMClient` interface with `complete()`, `complete_structured()`, `stream()`
- **Providers**: `LocalClient` (Ollama/vLLM), `OpenAIClient`, `AnthropicClient`, `GoogleClient` — all optional except LocalClient
- **Router**: `TaskRouter` selects provider/model based on `task_type`, `priority`, `privacy_level`, `budget_usd`
- **Budgets**: Per-task `max_tokens`, `max_cost_usd` (0 for local), `max_latency_ms`
- **Logging**: Every call → structured log (provider, model, tokens, cost, latency, task_id, success)
- **Caching**: Prompt + response cache for deterministic tasks (classification, extraction)
- **Default config**: Local-only; paid providers commented out

## Related ADRs

- ADR-0006: Agent Boundaries (which tasks use LLMs)
- ADR-0003: Embedding Model (separate, self-hosted)
- ADR-0004: Ingestion Architecture (enrichment uses LLMs)