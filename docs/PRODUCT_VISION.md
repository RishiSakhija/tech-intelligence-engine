# Product Vision

> Describes the intended user experience, major workflows, and long-term product direction for Tech Intelligence Engine.

---

## 1. Product Vision Statement

**To become the primary intelligence layer for technology professionals**—the tool they open first each morning to understand what changed, what matters, and what to investigate deeper in the AI/ML/engineering ecosystem.

Not a search engine you visit. An intelligence engine you *rely on*.

---

## 2. User Experience Overview

### 2.1 Core Loop

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  MORNING    │────▶│  EXPLORE    │────▶│  INVESTIGATE│────▶│  TRACK      │
│  BRIEFING   │     │  SEARCH     │     │  DEEP DIVE  │     │  CHANGES    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
     │                   │                   │                   │
     ▼                   ▼                   ▼                   ▼
Personalized          Hybrid search       Agentic research    Entity follows
feed: changes,        over curated        with citations      + alerts on
new papers,           technical corpus    + verification      releases,
model releases,       with filters        + synthesis         benchmarks
benchmark updates
```

### 2.2 Three Modes

| Mode | Trigger | Output | Time Investment |
|------|---------|--------|-----------------|
| **Briefing** | App open / scheduled | Ranked feed of changes since last visit | 2-5 min |
| **Search** | Explicit query | Results + facets + "Why ranked?" | 30 sec - 5 min |
| **Research** | Complex question | Structured report with evidence | 5-30 min |

---

## 3. Major Workflows

### 3.1 Morning Briefing (Discovery)

**Entry**: App homepage or email digest

**Components**:
- **Personalized Changes Feed**: "Since your last visit, 12 things you follow changed"
  - Model releases (new versions, new variants)
  - Paper publications (arXiv, conferences)
  - Benchmark updates (new SOTA, new submissions)
  - Framework releases (breaking changes highlighted)
  - Researcher activity (new paper from followed author)
- **Trending in Your Topics**: High-velocity items in followed areas (velocity = citations + GitHub stars + mentions)
- **Surprise Me**: 1-2 high-quality items outside follows but semantically close (serendipity slot)

**Interaction**:
- Click → Detail view (paper abstract, model card, changelog diff)
- "Follow" button on any entity
- "Not relevant" → downweights similar future items
- "Deep research" → spawns research agent

### 3.2 Technical Search

**Entry**: Search bar (global, always visible)

**Query Understanding**:
```
User: "llama 3.1 70b coding benchmark"
Parsed Intent: 
  - Entities: Model(Llama-3.1-70B), Task(Coding), Metric(Benchmark)
  - Filters: Model family=Llama, Size=70B, Task=Code
  - Sort: Recency + Benchmark score descending
```

**Results Page**:
- **Primary Results**: Hybrid ranked (semantic + lexical + authority + recency + personalization)
- **Facets**: Source type (Paper/Code/Benchmark/Release), Date, Author, Company, Model, Benchmark
- **Entity Cards**: Structured snippets (not just text)
  - Paper: Title, authors, venue, TL;DR, benchmarks, code link
  - Model: Name, size, license, benchmarks, HF link, GitHub link
  - Benchmark: Name, task, metric, top scores, leaderboard link
  - Release: Version, date, breaking changes, migration guide
- **"Why this result?"**: Expandable explanation per result

**Advanced**:
- Saved searches → alerts
- Query history with re-run
- Export results (CSV, BibTeX, Markdown)

### 3.3 Deep Research (Agentic)

**Entry**: "Research this" button on any result, or standalone research mode

**Flow**:
```
1. USER QUESTION
   "Compare all open MoE models on MMLU and inference latency"
   
2. RESEARCH PLANNER AGENT
   → Decomposes into sub-questions
   → Identifies required sources
   → Estimates steps & cost
   
3. SOURCE RESEARCH AGENTS (parallel)
   → Agent 1: Query ArXiv for MoE papers 2024-2025
   → Agent 2: Query HF Hub for MoE model cards
   → Agent 3: Query Papers with Code for MMLU scores
   → Agent 4: Search blogs for latency benchmarks
   
4. SYNTHESIS AGENT
   → Normalizes entities (same model across sources)
   → Resolves conflicts (different reported scores)
   → Builds comparison table with citations
   
5. VERIFICATION AGENT
   → Checks citations resolve
   → Flags unverified claims
   → Scores confidence per cell
   
6. OUTPUT
   → Structured report: Table + Narrative + Evidence Pack
   → "Verified" / "Unverified" badges per claim
   → Export: Notion, Markdown, PDF
```

**Output Format**:
- Executive summary (3 bullets)
- Comparison table (sortable, filterable)
- Evidence pack (expandable citations per cell)
- Gaps & uncertainties (what couldn't be found)
- Follow-up questions (suggested)

---

## 4. Search Experience Details

### 4.1 Query Types & Expected Behavior

| Query Type | Example | Expected Behavior |
|------------|---------|-------------------|
| **Entity lookup** | "GPT-4o mini release date" | Direct answer + source card (not 10 blue links) |
| **Comparative** | "Llama-3.1 vs Qwen2.5 benchmarks" | Structured comparison table |
| **Exploratory** | "New attention mechanisms 2024" | Clustered results by mechanism type |
| **Tracking** | "vLLM releases last month" | Chronological release list with changelog diffs |
| **How-to** | "How to quantize Llama-3.1 to 4-bit" | Tutorial/guide results prioritized |
| **Troubleshooting** | "Flash attention compile error H100" | GitHub issues, discussions, PRs prioritized |

### 4.2 Result Presentation

**Entity Cards** (not document snippets):

```
┌─────────────────────────────────────────────────────────────┐
│ 📄  Paper: "Edge0: Serving 35B MoEs from SSD..."           │
│     Authors: Bupalinyu et al.  │  arXiv:2609.18063  │ 2026  │
│     ─────────────────────────────────────────────────────   │
│     TL;DR: Streaming MoE inference engine with prerouter... │
│     ─────────────────────────────────────────────────────   │
│     Benchmarks: 35B MoE @ 20 tok/s on 24GB (MMLU: 0.78)    │
│     Code: github.com/Edge0-AI/edge0 (3.2k ⭐)               │
│     [View Paper] [View Code] [Follow Authors] [Cite]       │
└─────────────────────────────────────────────────────────────┘
```

### 4.3 Faceted Navigation

| Facet | Values | Multi-select |
|-------|--------|--------------|
| Source Type | Paper, Code, Model, Dataset, Release, Benchmark, Blog, Video | ✅ |
| Date | Past day/week/month/year/custom | ✅ |
| Author/Org | Auto-complete from index | ✅ |
| Model Family | Llama, Qwen, Gemma, Mistral, Custom | ✅ |
| Task | Coding, Reasoning, Chat, Math, Multilingual | ✅ |
| Benchmark | MMLU, HumanEval, GSM8K, MT-Bench, Custom | ✅ |
| License | Apache-2.0, MIT, CC-BY, Custom, Proprietary | ✅ |

---

## 5. Discovery Experience Details

### 5.1 Follow System (Explicit Personalization)

**Followable Entities**:
- Researchers (by name/ORCID/GitHub)
- Companies/Organizations (Google, Meta, HF, Anthropic, etc.)
- Model Families (Llama, Qwen, Mistral, Gemma)
- Specific Models (Llama-3.1-70B-Instruct)
- Repositories (vllm-project/vllm, huggingface/transformers)
- Topics (MoE, RAG, Quantization, Long-context, RLHF)
- Benchmarks (MMLU, HumanEval, MT-Bench, SWE-bench)
- Conferences (NeurIPS, ICML, ICLR, ACL)

**Follow UI**:
- Search result: "Follow [Entity]" button
- Entity detail page: Prominent follow button
- Settings: Manage follows, import/export

### 5.2 Implicit Personalization

| Signal | Weight | Decay |
|--------|--------|-------|
| Click result | 1.0 | 30 days |
| Dwell > 30s | 2.0 | 30 days |
| Follow entity | 10.0 | None (explicit) |
| Save search | 5.0 | 60 days |
| "Not relevant" | -5.0 | 14 days |
| Deep research on topic | 3.0 | 45 days |

**Profile Representation**: Weighted vector in embedding space + explicit entity graph

### 5.3 "What Changed" Concept

For any followed entity, show semantic diffs:

| Entity Type | Change Detection |
|-------------|------------------|
| **Model** | New version, new quantization, benchmark score update, license change |
| **Repository** | Release tags, breaking changes in changelog, dependency updates |
| **Paper** | New version (v2, v3), citation count milestones, replication results |
| **Benchmark** | New SOTA, new submission, leaderboard structure change |
| **Researcher** | New paper, new affiliation, new project announcement |
| **Company** | Model release, acquisition, open-source announcement |

**Diff View Example** (Model Release):
```
Llama-3.1-70B-Instruct v1.0 → v1.1
├── 📝 Model Card: Updated benchmark scores (MMLU: 86.1 → 87.3)
├── 🔧 Code: Added quantization configs for AWQ/GPTQ
├── 📄 License: Clarified commercial use terms
└── ⚠️ Breaking: tokenizer.chat_template format changed
```

---

## 6. Personalized Dashboard

**Sections** (configurable, reorderable):

1. **My Briefing** — Changes since last visit (default top)
2. **Followed Researchers** — Latest papers (3 each)
3. **Followed Models** — New versions, benchmarks, quantizations
4. **Followed Repos** — Releases, security advisories, major PRs
5. **Trending in My Topics** — Velocity-ranked
6. **Saved Searches / Alerts** — Quick re-run
7. **Recent Research** — Past agentic reports

**Customization**:
- Drag to reorder
- Hide/show sections
- Set refresh interval per section
- Compact / comfortable density

---

## 7. Long-Term Product Direction

### 7.1 Year 1: Search + Discovery Foundation
- Hybrid search over 1M+ technical docs
- Personalized briefing with follows
- Basic change detection on key entities
- Evaluation framework proving quality > generic search

### 7.2 Year 2: Intelligence Layer
- Structured benchmark database (versioned, sourced)
- Model comparison engine (specs + benchmarks + license)
- Researcher/company profiling (expertise map, collaboration graph)
- API for programmatic access

### 7.3 Year 3: Agentic Intelligence
- Autonomous monitoring: "Alert me when any model beats GPT-4o on SWE-bench"
- Trend detection: "Quantization research shifting to 2-bit this quarter"
- Predictive: "Based on author history, this paper likely produces open model"
- Integration: IDE plugin, Slack/Discord bots, Notion sync

### 7.4 Moat Deepening
- Proprietary entity resolution graph
- Technical fact extraction at scale
- Community-contributed corrections/annotations
- Private index for enterprise (on-prem)

---

## 8. Realistic Query Examples & Expected Outputs

| # | User Query | Mode | Expected Output |
|---|------------|------|-----------------|
| 1 | "New Mixture of Experts papers this month" | Search | Clustered by: routing method, architecture, application |
| 2 | "Show me changes to Transformers library last week" | Briefing | Release diff + breaking change highlights |
| 3 | "Who are the top researchers in long-context LLMs?" | Search | Ranked list: papers, citations, code, current affiliation |
| 4 | "Compare all 7B coding models on HumanEval" | Research | Table: Model | HumanEval | License | Context | VRAM | Source |
| 5 | "Alert me when a new model beats 90% on MMLU" | Tracking | Push notification + briefing entry |
| 6 | "Find implementations of Ring Attention" | Search | Code results: repos, files, licenses, stars |
| 7 | "What did Yann LeCun publish in 2024?" | Search | Chronological paper list with links |
| 8 | "Explain the difference between GRPO and PPO for RLHF" | Research | Structured explanation with paper citations |

---

## 9. Non-Functional UX Requirements

| Requirement | Target |
|-------------|--------|
| Search results render | < 200ms (cached), < 500ms (cold) |
| Briefing load | < 1s |
| Research report (5-step) | < 60s |
| Mobile responsive | Full functionality on 375px width |
| Keyboard navigation | 100% accessible |
| Offline briefing | Cached last 7 days |
| Export formats | Markdown, JSON, CSV, PDF |

---

## 10. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-10-06 | Initial product vision |

---

*This document defines the product direction. Implementation details in `SYSTEM_ARCHITECTURE.md`, `SEARCH_STRATEGY.md`, `PERSONALIZATION.md`, `AGENT_ARCHITECTURE.md`.*