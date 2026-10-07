# Team Onboarding Guide

> Concise onboarding guide for new contributors to Tech Intelligence Engine.

---

## 1. What This Project Is

**Tech Intelligence Engine** is a vertical search and intelligence platform for the technology ecosystem (AI/ML, papers, models, frameworks, benchmarks, releases, researchers, companies).

**We are NOT building:**
- A general web search engine
- A newsletter or news aggregator
- A chatbot wrapper around another search API
- A code completion tool

**We ARE building:**
- Search over curated technical sources (ArXiv, GitHub, HF Hub, Papers with Code, etc.)
- Entity-centric knowledge graph (papers ↔ models ↔ code ↔ benchmarks ↔ researchers)
- Personalized discovery with explicit follows + implicit interest modeling
- Semantic change detection ("what changed" diffs for releases, models, benchmarks)
- Agentic deep research with citations and verification

**Current Phase**: Phase 0 (Foundation & Research) complete. Phase 1 (Data Foundation) starts next.
**Implementation Status**: No code written yet — documentation and architecture only.

---

## 2. Important Documents — Reading Order

### Start Here (Essential Context)

| Order | Document | Why Read It |
|-------|----------|-------------|
| 1 | `README.md` | Project overview, architecture diagram, roadmap, repo structure |
| 2 | `docs/PROJECT_CONSTITUTION.md` | Highest-level source of truth: thesis, principles, non-goals, MVP boundaries |
| 3 | `docs/PROJECT_WALKTHROUGH.md` | Complete technical/product walkthrough (this replaces reading all docs individually) |

### Deep Dives (As Needed)

| Area | Document |
|------|----------|
| Product vision & UX | `docs/PRODUCT_VISION.md` |
| Requirements (P0-P3) | `docs/REQUIREMENTS.md` |
| Competitive landscape | `docs/COMPETITOR_ANALYSIS.md` |
| Data sources & ingestion | `docs/DATA_SOURCE_STRATEGY.md` |
| Search architecture | `docs/SEARCH_STRATEGY.md` |
| Personalization design | `docs/PERSONALIZATION.md` |
| Agent architecture & boundaries | `docs/AGENT_ARCHITECTURE.md` |
| System architecture & data flows | `docs/SYSTEM_ARCHITECTURE.md` |
| Data model (entities, relationships) | `docs/DATA_MODEL.md` |
| Security requirements | `docs/SECURITY.md` |
| Evaluation framework | `docs/EVALUATION.md` |
| Roadmap & phases | `docs/ROADMAP.md` |
| Architecture decisions | `docs/decisions/adr-*.md` |

### Glossary

| Document | Purpose |
|----------|---------|
| `docs/GLOSSARY.md` | Definitions of all project-specific terms |

---

## 3. Team Roles

| Role | Responsibilities |
|------|------------------|
| **Product/Tech Lead** (Rishi) | Architecture decisions, prioritization, ADR ownership, cross-team coordination |
| **Backend Engineer** | Ingestion pipeline, search service, API, PostgreSQL/ES/Qdrant/Redis, workers |
| **Frontend Engineer** | Next.js app, search UI, briefing feed, research workspace, entity pages |
| **ML/IR Engineer** | Embeddings, reranking, ranking signals, evaluation, query understanding |
| **Agent/Research Engineer** | Agent orchestration, planner, synthesis, verification, tool frameworks |

> In early phases, roles overlap. The tech lead makes final calls on architecture (ADRs) and priority.

---

## 4. How Humans and OpenCode Agents Work Together

### Human Responsibilities
- **Product decisions**: What to build, priority, trade-offs
- **Architecture decisions**: ADRs, technology selection, system design
- **Code review**: All PRs reviewed by human before merge
- **Quality gates**: Evaluation metrics, security review, performance benchmarks
- **External communication**: GitHub issues, stakeholder updates

### OpenCode Agent Responsibilities
- **Documentation**: Creating/updating docs based on human direction
- **Research**: Competitive analysis, technical fact-finding, source validation
- **Code scaffolding**: Boilerplate, templates, test structures (not business logic)
- **Refactoring**: Mechanical transformations, linting, formatting
- **Diagram generation**: Mermaid diagrams for architecture/docs

### Collaboration Rules
1. **Human initiates** — Agent doesn't start work without explicit human task
2. **Human decides** — Agent proposes options; human chooses
3. **Human reviews** — Agent output is draft; human validates before commit
3. **No secrets** — Never share API keys, tokens, or credentials with agents
4. **Document decisions** — If agent helps with research, human writes the ADR

---

## 5. Git / Branch / PR Workflow

### Branching Model

```
main (protected)
  │
  ├─ feature/ingestion-arxiv-connector
  ├─ feature/search-api-bm25
  ├─ feature/frontend-search-page
  └─ fix/rate-limit-bug
```

- **main** = deployable, passes all CI, always green
- **Feature branches** = short-lived, one logical change, rebased on main
- **No long-running release branches** — we ship from main

### PR Requirements

| Requirement | Details |
|-------------|---------|
| **Title** | `type(scope): brief description` — e.g., `feat(ingestion): add ArXiv API connector` |
| **Description** | What, why, how; link to related issue/ADR/requirement |
| **Tests** | Unit tests for new code; integration tests for API changes |
| **Docs** | Update relevant docs if behavior changes |
| **Review** | At least 1 approval; tech lead for architecture changes |
| **CI** | All checks pass (lint, type-check, tests, build) |

### Commit Message Format

```
type(scope): brief description

Longer explanation if needed. Reference issue/ADR.

- Detail 1
- Detail 2
```

**Types**: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `security`

---

## 6. Contribution Rules

### Code Standards

| Area | Standard |
|------|----------|
| **Python** | 3.11+, `ruff` (lint), `mypy` (strict), `pytest` (tests), `black` (format) |
| **TypeScript** | 5+, `eslint`, `prettier`, `vitest` (tests), strict mode |
| **API** | FastAPI + Pydantic v2; OpenAPI spec auto-generated |
| **Database** | Alembic migrations; backward-compatible only |
| **Secrets** | Never in code; `.env.example` documents all vars; use secret manager in prod |
| **Logging** | Structured JSON; correlation IDs for request tracing |

### PR Checklist (Must Pass Before Merge)

- [ ] `ruff check .` / `eslint .` — no lint errors
- [ ] `mypy .` / `tsc --noEmit` — no type errors
- [ ] `pytest` / `vitest run` — all tests pass
- [ ] `docker build` — image builds successfully
- [ ] Documentation updated if user-facing change
- [ ] ADR created/updated if architecture decision
- [ ] No secrets, no generated files, no large binaries

### What NOT to Do

- ❌ Commit directly to `main`
- ❌ Skip tests to "go faster"
- ❌ Add dependencies without evaluating alternatives (document in ADR if significant)
- ❌ Hardcode configuration — use environment variables
- ❌ Store secrets in repo (even temporarily)
- ❌ Build features not in P0/P1 without tech lead approval
- ❌ Use agents for tasks that should be deterministic code

---

## 7. How a New Developer Should Start

### Day 1: Environment & Context

1. **Clone the repo**
   ```bash
   git clone https://github.com/RishiSakhija/tech-intelligence-engine.git
   cd tech-intelligence-engine
   ```

2. **Read the essential docs** (in order):
   - `README.md`
   - `docs/PROJECT_CONSTITUTION.md`
   - `docs/PROJECT_WALKTHROUGH.md`

3. **Review the tech stack**:
   - Backend: Python 3.11, FastAPI, PostgreSQL, Elasticsearch, Qdrant, Redis
   - Frontend: Next.js 14, React 18, TypeScript, Tailwind, shadcn/ui
   - Infra: Docker, Redis Streams, (future: Kubernetes)

4. **Set up local environment** (when implementation starts):
   ```bash
   # Will be documented in Phase 1
   # docker-compose up -d  # PostgreSQL, ES, Qdrant, Redis
   # pip install -e .[dev]
   # npm install (in frontend/)
   ```

### Day 2: Pick a First Task

**Look for** `good first issue` labels or ask the tech lead for a P0 task in Phase 1.

Typical Phase 1 starter tasks:
- ArXiv API connector (fetch + normalize)
- GitHub webhook receiver for releases
- PostgreSQL schema + Alembic setup
- Elasticsearch index mappings
- Qdrant collection setup + embedding pipeline
- Basic FastAPI project scaffolding

### Week 1: First PR

1. Create feature branch: `git checkout -b feature/your-task-name`
2. Implement with tests
3. Run lint/type-check/test locally
4. Open PR with clear description
4. Address review feedback
5. Merge after approval

### Ongoing

- Attend weekly sync (if scheduled)
- Read ADRs before touching related areas
- Update docs when you learn something not documented
- Propose ADRs for new architecture decisions
- Keep PRs small and focused

---

## 8. Key Principles to Internalize

1. **Evidence over intuition** — Every architectural claim backed by research or experiment
2. **Vertical depth over horizontal breadth** — Own the tech ecosystem completely before expanding
3. **Simplicity first** — Deterministic pipelines; agents only where they add clear value
4. **Measurable quality** — Search quality evaluated against benchmarks, not vibes
5. **Source transparency** — Every result traceable to source; citations mandatory
6. **User control** — Personalization explainable and adjustable
7. **Freshness as feature** — Ingestion latency measured and optimized
8. **No vendor lock-in** — Open-source components preferred

---

## 9. Communication Channels

| Channel | Purpose |
|---------|---------|
| **GitHub Issues** | Bug reports, feature requests, task tracking |
| **GitHub Discussions** | Design discussions, RFCs, questions |
| **PR Reviews** | Code review, knowledge sharing |
| **ADRs** | Permanent record of architecture decisions |
| **Weekly Sync** | (If scheduled) Progress, blockers, priorities |

---

## 10. Questions?

**Ask the tech lead (Rishi)** for:
- Clarification on architecture/product decisions
- Priority conflicts
- ADR proposals
- Access to external APIs / secrets
- Anything not covered here

**Welcome to the team!** 🚀