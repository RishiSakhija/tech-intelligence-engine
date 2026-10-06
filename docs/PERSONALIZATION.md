# Personalization

> Technical personalization approach: interest modeling, follow system, ranking influence, privacy, and user control.

---

## 1. Personalization Philosophy

**Personalization must affect useful ranking/discovery, not merely UI decoration.**

| Decorative Personalization | Functional Personalization |
|---------------------------|----------------------------|
| "Recommended for you" carousel | Boosted ranking for followed entities |
| Topic tags on profile | Interest vector influencing search scores |
| Generic "AI/ML" category follow | Entity-level follows (specific model, researcher, benchmark) |
| Opaque algorithm | "Why this result?" with personalization breakdown |
| No reset option | One-click "Forget my history" |

---

## 2. User Interest Model

### 2.1 Explicit Signals (High Weight)

| Signal | Weight | Decay | Source |
|--------|--------|-------|--------|
| Follow researcher | 10.0 | None | User action |
| Follow company/org | 8.0 | None | User action |
| Follow model | 8.0 | None | User action |
| Follow repository | 7.0 | None | User action |
| Follow benchmark | 7.0 | None | User action |
| Follow topic | 5.0 | None | User action |
| Save search / Create alert | 5.0 | 60 days | User action |
| Explicit "Not relevant" | -5.0 | 14 days | User action |

**Followable Entity Types**:
- `RESEARCHER` (by Semantic Scholar ID / ORCID / name)
- `COMPANY` (GitHub org / HF org / Crossref affiliation)
- `MODEL` (HF model ID)
- `REPOSITORY` (GitHub owner/repo)
- `BENCHMARK` (Papers with Code task/dataset/metric)
- `TOPIC` (controlled vocabulary: "MoE", "RAG", "Quantization", etc.)

### 2.2 Implicit Signals (Lower Weight, Decay)

| Signal | Weight | Half-Life | Capture Method |
|--------|--------|-----------|----------------|
| Click result | 1.0 | 30 days | Search analytics |
| Dwell > 30 seconds | 2.0 | 30 days | Frontend heartbeat |
| Dwell > 2 minutes | 3.0 | 30 days | Frontend heartbeat |
| Open entity detail | 2.0 | 30 days | Click tracking |
| Run deep research | 3.0 | 45 days | Research service |
| Share/export result | 2.0 | 60 days | UI action |
| Follow from result | 5.0 | None | Converts to explicit |

### 2.3 Interest Representation

**Dual Representation**:

1. **Explicit Entity Graph**: `UserFollows(user_id, entity_type, entity_id, weight, created_at)`
   - Direct, interpretable, exportable
   - Used for: Briefing feed, hard boosts, "You follow X" badges

2. **Implicit Interest Vector**: 1024-dim embedding (same space as document vectors)
   - Computed: Weighted average of interacted document vectors
   - Updated: Online (per interaction) + nightly batch recomputation
   - Used for: Soft ranking boost, serendipity, "Trending in your topics"

**Vector Update Formula** (per interaction):
```
user_vector = (1 - α) * user_vector + α * doc_vector
α = signal_weight * decay_factor
```

---

## 3. Follow System

### 3.1 Follow UX

**Entry Points**:
- Search results: "Follow [Entity]" button on each card
- Entity detail page: Prominent follow button
- Briefing feed: "Follow" on each change item
- Settings → Manages Follows: Search + add

**Follow Card** (in settings):
```
┌─────────────────────────────────────────────┐
│ 👤  Yann LeCun (Researcher)                 │
│    Chief AI Scientist @ Meta | 500k+ cites │
│    [Unfollow] [Notifications: All/High]    │
├─────────────────────────────────────────────┤
│ 🏢  Hugging Face (Company)                  │
│    2M+ models | 500k+ datasets              │
│    [Unfollow] [Notifications: Releases]    │
├─────────────────────────────────────────────┤
│ 🤖  Llama-3.1-70B-Instruct (Model)         │
│    Meta | Apache-2.0 | 128k ctx            │
│    [Unfollow] [Notifications: Benchmarks]  │
└─────────────────────────────────────────────┘
```

### 3.2 Notification Granularity

Per-entity notification preferences:
| Entity Type | Notification Options |
|-------------|---------------------|
| Researcher | All papers / High-impact only (citations > threshold) |
| Company | All releases / Models only / Blog posts only |
| Model | New versions / New quantizations / New benchmarks / All |
| Repository | Releases only / Security advisories / All commits |
| Benchmark | New SOTA / New submissions / Leaderboard changes |
| Topic | Trending papers / All papers / Weekly digest |

---

## 4. Personalized Ranking

### 4.1 Ranking Formula

```
final_score = base_score * (1 + personalization_factor)

personalization_factor = 
    min(0.5,                              # Cap at 50% boost
        explicit_boost +                  # 0.0 to 0.3
        implicit_boost +                  # 0.0 to 0.2
        recency_boost                     # 0.0 to 0.1
    )
```

**Explicit Boost**:
- Direct entity match: `+0.3` (followed model/paper/researcher appears)
- Related entity match: `+0.15` (followed researcher's new paper; followed company's model)
- Topic match: `+0.1` (result matches followed topic)

**Implicit Boost**:
- Cosine similarity between query+result vector and user interest vector
- Scaled: `cosine_sim * 0.2` (max +0.2)

**Recency Boost**:
- User's recent searches (last 7 days) create temporary interest boost
- Decays exponentially over 7 days

### 4.2 Personalization in Briefing Feed

**Briefing = Union of**:
1. **Explicit Changes**: All changes to followed entities since last visit (ranked by change significance)
2. **Implicit Recommendations**: Top-K from personalized search on "trending in your topics"
3. **Serendipity Slot**: 1-2 items with high authority + semantic proximity to interest vector but no explicit follow

**Ranking within Briefing**:
1. Explicit follows (by change significance: breaking > new benchmark > new version > minor)
2. Implicit recommendations (by personalized score)
3. Serendipity (fixed position)

---

## 5. Discovery Features

### 5.1 "Trending in Your Topics"

**Algorithm**:
1. Get user's explicit topics + top implicit topics (from interest vector)
2. Find documents in last 7 days matching those topics
3. Rank by **velocity**: `log(views + citations + stars + 1) / hours_since_publish`
4. Filter out already-seen / followed entities
5. Top 10 → Briefing feed

### 5.2 Serendipity Engine

**Goal**: High-quality items outside explicit follows but semantically adjacent

**Method**:
1. Sample from user's interest vector neighborhood (k-NN in vector space)
2. Filter: High authority (>80), not seen, not followed, different entity than recent clicks
3. Diversity: MMR across topics
4. Inject 1-2 per briefing

---

## 6. Privacy & User Control

### 6.1 Data Minimization

| Data Collected | Purpose | Retention |
|----------------|---------|-----------|
| Explicit follows | Core personalization | Until unfollowed |
| Click/dwell analytics | Implicit model | 90 days (aggregated after 30) |
| Search queries | Query understanding, implicit model | 90 days (hashed after 30) |
| Research history | User's research workspace | Until deleted |
| Export data | Portability | On demand |

**NOT Collected**:
- IP-based tracking across sessions
- Third-party cookies
- Behavioral profiling beyond search scope
- PII beyond email (for auth)

### 6.2 Transparency Features

1. **"Why this result?"** — Per-result breakdown including personalization contribution
2. **"Why in my briefing?"** — Per-item explanation (explicit follow / implicit / serendipity)
3. **Interest Profile View** — Visualizable: top topics, followed entities, implicit vector projection
4. **Data Export** — JSON: follows, interest vector (anonymized), search history, research history

### 6.3 Control Features

| Control | Implementation |
|---------|----------------|
| **Unfollow** | Immediate removal from explicit graph; implicit influence decays naturally |
| **Pause Personalization** | Toggle: `personalization_factor = 0` for session |
| **Reset Implicit Model** | One-click: zero out interest vector; keep explicit follows |
| **Reset All** | Nuclear option: delete all personalization data |
| **Export/Import** | JSON file: follows + interest vector (for portability) |
| **Incognito Search** | Session without personalization; no logging |

### 6.4 GDPR / CCPA Compliance

- **Right to Access**: `/api/user/data-export` endpoint
- **Right to Deletion**: `/api/user/delete` — purges all personalization + account
- **Right to Rectification**: Edit follows, correct interest signals
- **Data Processing Agreement**: For any subprocessors (embedding APIs, etc.)
- **Privacy Policy**: Plain language; updated with feature changes

---

## 7. Cold Start Strategy

**New User (0 signals)**:

| Phase | Approach |
|-------|----------|
| **Onboarding** | Optional interest selection: "Select 3-5 topics you work on" → seed explicit follows |
| **First 10 searches** | No personalization; collect implicit signals |
| **After 50 interactions** | Enable implicit boost (low weight: 0.1 max) |
| **After 5 explicit follows** | Enable explicit boost (full weight) |

**Anonymous Users**:
- Session-based implicit vector (localStorage)
- No persistence across sessions
- "Sign in to save your interests" prompt after 5 searches

---

## 8. Evaluation (See `EVALUATION.md`)

**Personalization Metrics**:
- **Lift**: nDCG@10 personalized vs. non-personalized (target: +15%)
- **Coverage**: % queries with at least 1 personalized result in top-10
- **Diversity**: Personalized results not dominated by single entity
- **User Satisfaction**: Explicit feedback on briefing relevance
- **Retention**: 7-day / 30-day return rate for personalized vs. non-personalized users

**A/B Test Design**:
- Control: No personalization (base rank only)
- Treatment: Full personalization
- Metric: Search session success (click + dwell > 30s)
- Minimum detectable effect: 5% lift

---

## 9. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-10-06 | Initial personalization design |

---

*This document informs `REQUIREMENTS.md` (FR-PERS-*), `SYSTEM_ARCHITECTURE.md` (Personalization Service), and `EVALUATION.md`.*