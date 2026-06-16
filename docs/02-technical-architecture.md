# Technical Architecture

**MattGPT: Architecture & Technical Evolution**

> This document outlines the technical architecture of MattGPT, demonstrating intentional trade-off awareness from rapid prototyping through production feature delivery.

---

## Architecture Roadmap Overview

MattGPT's architecture evolved through intentional phases:

1. **Phase 1 (Streamlit MVP):** Validate RAG architecture with minimal investment — ✅ Complete
2. **Production (Streamlit):** Ongoing feature delivery — Role Match, mobile responsive, eval framework, query analytics
3. **Future consideration:** React + FastAPI migration if a forcing function emerges (see [Migration Architecture](/mattgpt-design-spec/docs/13-migration-architecture))

---

## Phase 1: MVP - Rapid Validation

**Status:** ✅ Complete (December 2025)
**Timeline:** Current State
**Primary Goal:** Validate product-market fit with minimal investment

### Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| 🐍 **Frontend/Backend** | Streamlit (Monolith) | Rapid prototyping framework |
| 🤖 **LLM** | OpenAI GPT-4 | Natural language processing |
| 🌲 **Vector Database** | Pinecone | Semantic search and embeddings |
| 📦 **Dependencies** | Python venv + pip | Local development environment |

### Accepted Trade-offs

The MVP phase consciously accepted limitations to accelerate learning:

- ~~**Mobile-responsive implementation:**~~ ✅ Shipped — 1,520 lines of production mobile CSS with breakpoints at 767px (mobile), 768-1024px (tablet), 1024+ (desktop)
- **Limited scalability:** Estimated ~100 concurrent users (not load-tested; Streamlit Community Cloud limits apply)
- **No caching layer:** Direct database queries
- ~~**Modular architecture:**~~ ✅ Shipped — refactored from monolithic app.py to component-based structure (118 Python files)
- ~~**Minimal observability:**~~ ✅ Shipped — 32-column query logger to Google Sheets capturing events, intent, UTM attribution, and Role Match outcomes

### Key Decisions

**Why Streamlit?**
- ✅ Initial MVP build time: 2 weeks vs 3+ months for React*
- ✅ Python-native (matches ML/AI ecosystem)
- ✅ Built-in state management
- ✅ Rapid iteration without frontend complexity

*Note: Continuous refinement and feature additions ongoing since launch.

**What We Validated:**
- RAG architecture effectiveness
- System prompt design
- STAR methodology implementation
- User experience patterns
- Search relevance tuning

### Architecture Evolutions Achieved

**January 2026 - RAG Pipeline Cleanup:**
- ✅ 5-stage RAG pipeline with 100% eval pass rate (64/64)
- ❌ Removed Entity Gate bouncer (was blocking legitimate queries)
- ❌ Removed `classify_query_intent()` LLM call (redundant with router)
- ✅ Semantic router now handles synthesis + out-of-scope + narrative detection
- ✅ Expanded to 15 intent families (was 10)
- ✅ Centralized thresholds in `config/constants.py`
- ✅ Unified SEARCH_TOP_K = 10 (was 100/7 conflict)
- ✅ Title soft filtering (semantic search ranks naturally)
- ✅ Model upgrade: GPT-4o-mini → GPT-4o

**December 2025 - Modular Architecture:**
- Refactored monolithic ask_mattgpt.py (4,696 lines) into 9-file directory
- Separated concerns: backend_service.py, conversation_view.py, conversation_helpers.py
- Reusable component library (ui/components/)

**Design System:**
- CSS variables for light/dark mode support (global_styles.py:28-122)
- Standardized breakpoints: 767px (mobile), 768-1024px (tablet), 1024+ (desktop)
- Consistent purple brand (#8B5CF6) across all views

**Timeline View Innovation:**
- Era-based project grouping (timeline_view.py)
- 5 career eras with date range calculation
- Progressive disclosure pattern (6 most recent per era)

**Mobile Implementation:**
- 200 lines of responsive CSS (global_styles.py:399-596)
- Touch-optimized controls and stacking layouts
- Horizontal scroll tables with preserved functionality

---

## 5-Stage RAG Pipeline

**Status:** ✅ Implemented (January 2026)
**Quality:** 100% eval pass rate (64/64 queries)

MattGPT uses a **5-stage RAG (Retrieval-Augmented Generation) pipeline** to ensure accurate, grounded responses:

### Pipeline Overview

```
Query → Stage 1 → Stage 2 → Stage 3 → Stage 4 → Stage 5 → LLM Response
```

**Stage 1: Rules-Based Nonsense Detection**
- Fast regex matching against `nonsense_filters.jsonl` patterns
- Catches obvious off-domain queries (weather, sports, stocks, etc.)
- Zero embedding cost, executes in <5ms
- Blocks 60%+ of nonsense before semantic processing

**Stage 2: Semantic Router (Intent + Out-of-Scope Detection)**
- Embedding-based similarity matching against 15 intent families
- Dual-threshold system: HARD_ACCEPT (0.80), SOFT_ACCEPT (0.40)
- Detects synthesis queries (no Pinecone search needed)
- Flags out-of-scope industries gracefully
- **Replaced:** Legacy LLM intent classification (removed Jan 2026)

**Stage 3: Confidence Gating**
- Pinecone semantic search with confidence scoring
- Thresholds: HIGH (0.25), LOW (0.20)
- Filters phantom similarity noise
- Returns 0 results below minimum threshold

**Stage 4: Entity Detection & Pinning**
- Detects: Client, Employer, Division, Title
- Hard filters: Client, Employer, Division, Project, Place
- Soft filters: Title (semantic search ranks naturally)
- **Removed:** Entity Gate threshold bouncer (was blocking valid queries)

**Stage 5: Intent-Aware Ranking**
- Narrative queries: Trust Pinecone semantic ranking
- Entity queries: Pin matching story to #1
- Context isolation via XML tags prevents cross-story bleed
- Multi-field entity search across 6 fields

---

## Production RAG Flow (8-Layer Detail)

**Complete query-to-response flow with all decision points:**

```
User Question: "How did Matt scale engineering teams?"
      ↓
[Layer 1: Validation]
├─ is_nonsense() → reject if regex match (weather, sports, etc.)
├─ Semantic router → reject if score < 0.40
└─ Returns intent_family (15 families: narrative, synthesis, personal, technical, etc.)
      ↓
[Layer 2: Fast Exit Checks]
├─ Out-of-scope check: if intent_family == "out_of_scope" → graceful redirect
├─ Synthesis check: if intent_family == "synthesis" → skip Pinecone, use theme filter
└─ Entity detection → (field, value) for scoped retrieval
      ↓
[Layer 3: Semantic Search]
├─ Embed query with text-embedding-3-small (1536 dims)
├─ Vector search in Pinecone (top 10, similarity > 0.15)
├─ Apply entity metadata filters (Client, Employer, Division, Project, Place)
└─ NOTE: Title entities use SOFT filtering (no Pinecone filter, semantic ranks naturally)
      ↓
[Layer 4: Confidence Gate]
├─ CONFIDENCE_HIGH (≥0.25) → proceed normally
├─ CONFIDENCE_LOW (≥0.20, <0.25) → proceed with warning
└─ Below 0.20 → "I couldn't find relevant stories" + suggestion chips
      ↓
[Layer 5: Retrieval Strategy (Intent-Based)]
├─ STANDARD MODE: entity pin → diversify_results() → top 7 with client variety
├─ NARRATIVE MODE: sort by Pinecone score (skip diversity, trust semantic ranking)
└─ SYNTHESIS MODE: theme-filtered parallel search → named-clients-first (up to 9)
      ↓
[Layer 6: Context Assembly]
├─ XML isolation: <primary_story> + <supporting_story> tags prevent cross-story bleed
├─ Build prompt with STAR narratives + theme guidance
├─ Include MATT_DNA ground truth (dynamic from JSONL)
└─ Context window: ~2,000-4,000 tokens
      ↓
[Layer 7: LLM Generation (GPT-4o)]
├─ STANDARD: Primary story focus → human stakes → methodology → outcomes
├─ SYNTHESIS: Theme/pattern → evidence across projects → insight
└─ Temperature: 0.4 (standard) / 0.2 (synthesis for consistency)
      ↓
[Layer 8: Response Formatting]
├─ Extract answer + sources from LLM response
├─ Render with citations (story titles, confidence scores)
├─ Display Related Projects (3-5 semantically similar stories)
└─ Generate follow-up question
      ↓
User receives cited, STAR-formatted answer with sources
```

*Components removed during the January 2026 RAG pipeline cleanup (Entity Gate, classify_query_intent LLM call, Title Hard Filtering) are documented in [HISTORY.md](https://github.com/mpugmire/llm_portfolio_assistant/blob/main/HISTORY.md) in the codebase repo.*

---

## Synthesis Mode

**Purpose:** Answer high-level narrative and thematic questions by synthesizing patterns across multiple stories.

### When Synthesis Mode Triggers

**Intent Family Detection:**
- Semantic router classifies query as `intent_family == "synthesis"`
- Examples: "What's Matt's professional narrative?", "Summarize Matt's career themes", "What patterns do you see across projects?"

**Alternative Trigger:**
- Explicitly requested: "Synthesize insights across banking projects"

### How Synthesis Differs from Standard

| Aspect | Standard Mode | Synthesis Mode |
|--------|--------------|----------------|
| **Search Strategy** | Pinecone vector search | Theme-filtered parallel search |
| **Story Selection** | Top 7 with diversity | Up to 9, named-clients-first |
| **Ranking** | Pinecone score + entity pinning | Named clients prioritized over generic |
| **Temperature** | 0.4 | 0.2 (lower for consistency) |
| **Focus** | Single story deep-dive | Patterns across stories |
| **Sacred Vocabulary** | Optional | **Required** ("builder") |

### Synthesis Search Strategy

**Theme-Filtered Parallel Search:**

```python
# Don't rely on single query - search by themes
themes = ["Platform Modernization", "Team Scaling", "Agile Transformation"]
all_results = []

for theme in themes:
    results = semantic_search(theme, filters={})
    all_results.extend(results[:3])  # Top 3 per theme

# Prioritize named clients over generic
named_first = sorted(all_results, key=lambda s: is_generic_client(s["Client"]))
return named_first[:9]  # Max 9 stories for synthesis
```

**Why Parallel Search:**
- Single query might miss thematic breadth
- Searching by themes ensures coverage across career
- Named clients (JP Morgan, Johnson & Johnson) more credible than "Multiple Clients"

### Sacred Vocabulary Enforcement

**Verbatim phrase required in synthesis responses:**

- **"builder"** — the one term that must appear verbatim

Other vocabulary (modernizer, complexity to clarity, proof over promises, maintenance role) was removed from story data during March 2026 data quality cleanup. Only "builder" remains as a verbatim requirement.

**Why Strict:**
- Professional narrative must be consistent across all synthesis queries
- "Builder" is Matt's actual language (from "About Matt" story in JSONL)
- Marketing/branding consistency

**Example Synthesis Response:**

```
Matt is a **builder** — someone brought in to create what doesn't exist yet,
whether that's a team, a platform, or a capability.

Across 20 years at Accenture, the pattern is consistent: he's brought to projects
where organizations need to build something from nothing or modernize platforms
stuck in technical debt. From scaling the Cloud Innovation Center to 150+ people
to modernizing payments across 12 countries, his work comes with evidence —
real outcomes, not just strategy decks.
```

### MATT_DNA Ground Truth

**Dynamic extraction from JSONL:**

```python
# Extracted at startup from "About Matt – My Leadership Journey" story
MATT_DNA = {
    "identity": [
        "builder"  # Only verbatim-required term (March 2026 cleanup)
    ],
    "focus_areas": [
        "solving ambiguous problems",
        "building high-trust engineering cultures",
        "modernizing platforms",
        "shifting how organizations think about technology"
    ]
}
```

**Why Dynamic:**
- Single source of truth: "About Matt" story in JSONL
- Update narrative in Excel → auto-updates everywhere
- No hardcoded identity statements in code

### Synthesis Prompt Structure

**BASE_PROMPT:**
- Establishes Agy as fact-relayer (not evaluator)
- Sets voice guidelines (no meta-commentary)

**SYNTHESIS_DELTA:**
```python
+ """
When answering synthesis/narrative queries:
- Use MATT_DNA verbatim phrases: {sacred_vocab}
- Synthesize patterns across projects (not individual deep-dives)
- Focus on themes, career arc, professional identity
- Ground in concrete examples (name clients, cite metrics)
- Lower temperature (0.2) for consistency
"""
```

### Synthesis vs Narrative Queries

**Synthesis:** High-level patterns and themes
- "What's Matt's professional identity?"
- "Summarize Matt's approach to leadership"
- "What makes Matt different from other platform leaders?"

**Narrative:** Professional story arc (different intent family)
- "What's Matt's career journey?"
- "How did Matt's work evolve over 20 years?"
- "Tell me Matt's background"

**Both use synthesis mode, but:**
- Synthesis → thematic patterns
- Narrative → chronological career story

---

## Query Validation & Nonsense Detection

**Purpose:** Prevent off-domain queries and ensure Agy only answers questions grounded in Matt's portfolio

### Architecture Overview

MattGPT uses a **two-layer defense** (Stages 1-2 of the RAG pipeline):

**Layer 1: Pattern-Based Filters** (Stage 1)
- Fast regex patterns for common off-domain categories
- Catches edge cases before semantic routing
- Zero-shot detection without embedding costs

**Layer 2: Semantic Router** (Stage 2)
- Embedding-based similarity matching against canonical intent examples
- Dual-threshold system for confident classification
- Handles intent classification, synthesis detection, and out-of-scope flagging
- Returns best matching intent family for telemetry and debugging

**Implementation Files:**
- `nonsense_filters.jsonl` - Regex pattern definitions
- `services/semantic_router.py` - Intent classification logic
- `data/intent_embeddings.json` - Cached embeddings for intent families

---

### Semantic Router: Dual-Threshold Intent Classification

**How It Works:**

1. User submits query
2. Query is embedded using OpenAI text-embedding-3-small
3. Cosine similarity computed against canonical intent examples
4. Query classified based on similarity threshold

**Thresholds (Updated January 2026):**

```python
HARD_ACCEPT = 0.80  # Clearly on-topic, no question
SOFT_ACCEPT = 0.40  # Accept but log as borderline for review (lowered from 0.72)
# Below 0.40 = router rejects (search fallback may still attempt)
```

**What Changed (January 2026):**
- ❌ **Removed:** Entity Gate threshold bouncer (was blocking legitimate narrative queries)
- ❌ **Removed:** `classify_query_intent()` LLM call (GPT-4o-mini - redundant with router)
- ✅ **Added:** Synthesis detection (intent_family == "synthesis")
- ✅ **Added:** Out-of-scope industry detection (intent_family == "out_of_scope")
- ✅ **Added:** Narrative queries (intent_family == "narrative")
- ✅ **Added:** Personal query detection (intent_family == "personal")
- ✅ **Expanded:** 10 intent families → 15 intent families

**Intent Families (15 categories):**

| Family | Example Queries |
|--------|----------------|
| **background** | "Tell me about Matt's career", "Who is Matt Pugmire?" |
| **behavioral** | "Tell me about a time Matt failed", "How do you handle conflict?" |
| **delivery** | "How did Matt achieve 4x faster delivery?", "Show me delivery wins" |
| **team_scaling** | "How did Matt scale teams from 4 to 150+?" |
| **leadership** | "What's Matt's leadership style?", "How does Matt coach people?" |
| **technical** | "Show me Matt's platform engineering work", "Cloud modernization?" |
| **domain_payments** | "Show me payments modernization work", "Experience in banking?" |
| **domain_healthcare** | "Show me healthcare projects", "Life sciences experience?" |
| **stakeholders** | "How does Matt handle difficult stakeholders?" |
| **innovation** | "Tell me about the innovation center work" |
| **agile_transformation** | "Tell me about agile transformation", "Scaling agile delivery" |
| **narrative** | "What's Matt's professional identity?", "Builder vs maintainer?" |
| **synthesis** | "What's Matt's professional narrative?", "Summarize Matt's career themes" |
| **personal** | "How old is Matt?", "What's Matt's salary?", "Where does Matt live?" |
| **out_of_scope** | Industry queries outside Matt's domain (graceful redirect) |

**Example Classification:**

```python
# HARD_ACCEPT (score 0.85, above 0.80 threshold)
"How did Matt scale agile at JP Morgan?" → intent_family: "delivery"

# SOFT_ACCEPT (score 0.74, above 0.40 threshold)
"What's Matt's approach to team growth?" → intent_family: "team_scaling"
# Logged for review, but accepted

# REJECTED (score 0.35, below 0.40 threshold)
"What's the weather in New York?" → intent_family: None
# Triggers off-domain response
```

---

### RAG Pipeline Thresholds & Constants

All thresholds are centralized in `config/constants.py` (single source of truth):

**Semantic Router Thresholds:**
```python
HARD_ACCEPT = 0.80   # Clearly on-topic, no question
SOFT_ACCEPT = 0.40   # Accept but log as borderline for review
```

**RAG Confidence Thresholds:**
```python
CONFIDENCE_HIGH = 0.25  # Strong match - show "Found X stories"
CONFIDENCE_LOW = 0.20   # Raised from 0.15 to filter phantom similarity noise
```

**Pinecone Search Parameters:**
```python
PINECONE_MIN_SIM = 0.15   # Minimum similarity for Pinecone results
SEARCH_TOP_K = 10         # Stories to fetch (headroom for reranking/filtering)
                          # Centralized Jan 2026 - was 100/7 conflict
```

**Entity Detection:**
```python
ENTITY_GATE_THRESHOLD = 0.30  # Semantic scoring threshold (lowered from 0.55)
                               # Note: Entity Gate as bouncer removed Jan 2026
```

**Model Configuration:**
```python
DEFAULT_CHAT_MODEL = "gpt-4o"              # Primary LLM (upgraded from gpt-4o-mini)
DEFAULT_EMBEDDING_MODEL = "text-embedding-3-small"  # 1536 dimensions
SEARCH_TOP_K = 10                          # Was 100 (explore) / 7 (ask) - now unified
```

---

### Pattern-Based Filters: Regex Nonsense Detection

**How It Works:**

Fast regex matching against `nonsense_filters.jsonl` patterns to catch obvious off-domain queries.

**Filter Categories (10+):**

| Category | Pattern Examples | Sample Rejected Queries |
|----------|-----------------|------------------------|
| **weather** | `\b(weather|forecast|temperature|rain|snow)\b` | "What's the weather today?" |
| **sports** | `\b(NBA|NFL|Yankees|score|odds|betting)\b` | "Who won the Super Bowl?" |
| **stocks_crypto** | `\b(stock|NASDAQ|BTC|crypto|price chart)\b` | "What's Bitcoin's price?" |
| **personal_sensitive** | `\b(SSN|social security|passport|credit card)\b` | "What's Matt's social security number?" |
| **retail_price** | `\b(how much|price|cost)\\b.*\\b(walmart|amazon)\b` | "How much does this cost at Walmart?" |
| **shopping_merch** | `\b(hat|shirt|hoodie)\\b.*\\b(buy|purchase)\b` | "Where can I buy a hat?" |
| **random_fun** | `\b(lottery|horoscope|zodiac|pick-up line)\b` | "What's my horoscope?" |
| **personal_trivia** | `favorite (color|food|movie|song)` | "What's Matt's favorite color?" |
| **creative_writing** | `\b(write|compose)\\b.*(poem|story|haiku)\b` | "Write me a poem" |
| **general_knowledge** | `\b(capital of|president of|who is the)\b` | "Who is the president of France?" |

**Performance:**
- Pattern matching is O(n) where n = number of patterns (~20)
- Executes in <5ms before semantic routing
- Zero embedding cost for obvious nonsense

---

### UX Flow for Off-Domain Queries

**When a query is rejected:**

1. **Semantic router** flags query as below 0.40 threshold
2. **Pattern filter** (optional) confirms off-domain category
3. **Agy responds** with helpful redirect:

**Example Response:**

```
🐾 I'm not finding matches for that in Matt's portfolio.

I'm focused on Matt's 20 years of digital transformation
experience—things like agile delivery, platform engineering,
team scaling, and stakeholder management.

Try asking about:
• "How did Matt scale teams at JP Morgan?"
• "Show me Matt's platform engineering work"
• "What's Matt's approach to innovation leadership?"

What would you like to know about Matt's experience?
```

**Suggestion Chips Displayed:**
- "Show me agile transformation projects"
- "Tell me about Matt's leadership style"
- "How did Matt scale engineering teams?"

---

### Accepted vs. Rejected Query Examples

**✅ ACCEPTED Queries:**

| Query | Why Accepted | Intent Family |
|-------|-------------|---------------|
| "How do you scale agile?" | 0.87 similarity to "team_scaling" family | team_scaling |
| "Tell me about Matt's biggest failure" | 0.82 similarity to "behavioral" family | behavioral |
| "Show me banking projects" | 0.91 similarity to "domain_payments" family | domain_payments |
| "What's Matt's technical background?" | 0.85 similarity to "technical" family | technical |
| "How does Matt handle difficult stakeholders?" | 0.89 similarity to "stakeholders" family | stakeholders |

**❌ REJECTED Queries:**

| Query | Why Rejected | Detection Method |
|-------|-------------|------------------|
| "What's the weather in New York?" | 0.32 similarity + weather pattern match | Semantic + Pattern |
| "Who won the Super Bowl?" | 0.28 similarity + sports pattern match | Semantic + Pattern |
| "What's Bitcoin's price?" | 0.19 similarity + crypto pattern match | Semantic + Pattern |
| "Write me a poem about leadership" | 0.41 similarity + creative_writing pattern | Semantic + Pattern |
| "What's Matt's favorite color?" | 0.35 similarity + personal_trivia pattern | Semantic + Pattern |

**Borderline Cases (0.40-0.80):**

| Query | Similarity | Action |
|-------|-----------|--------|
| "What's Matt's management philosophy?" | 0.74 | Accepted (soft), logged for review |
| "How does someone become a platform engineer?" | 0.58 | Accepted (soft), may redirect to Matt's experience |
| "Tell me about product-market fit" | 0.35 | Rejected, suggest Matt's product innovation work |

---

### Why This Approach Works

**Prevents Poor User Experience:**
- No wasted searches on irrelevant queries
- No confusing "here's what I found" with zero results
- Immediate feedback with helpful suggestions

**Maintains Credibility:**
- Agy never pretends to know things outside Matt's domain
- Clear boundaries reinforce "proof, not promises" positioning
- Honest limitations build trust

**Reduces Costs:**
- Pattern filters catch 60%+ of nonsense before embedding
- Semantic router catches another 35% before RAG pipeline
- Only ~5% of queries reach expensive LLM generation unnecessarily

**Enables Learning:**
- Soft accept threshold (0.40-0.80) logs borderline queries
- Manual review improves intent family coverage over time
- Telemetry shows what users are actually asking

---

### Previously Considered Enhancements

The following were evaluated and **decided against** on architectural grounds. The data pipeline (Excel → JSONL → embeddings → Pinecone) is one-directional by design with no write-back path. This constraint makes closed-loop approaches impractical at current scale.

- **~~Dynamic intent expansion~~** — Would require a write-back path to update intent embeddings from accepted queries. Eval-driven manual iteration serves this purpose at current traffic volume. (MATTGPT-050, Decided Against)
- **~~User feedback loop retraining~~** — No write-back path for closed-loop retraining. Google Sheets logger captures feedback for manual analysis instead. (MATTGPT-051, Decided Against)
- **~~A/B testing on thresholds~~** — Traffic volume insufficient for statistical significance. Eval suite with 64 golden queries provides deterministic threshold validation. (MATTGPT-052, Decided Against)
- **Intent routing** — Different response templates per intent family. Spirit partially exists via random focus angles; deterministic mapping tracked as MATTGPT-043.
- **Analytics dashboard** — Tracked as MATTGPT-045 (Open, Low priority). Query logger captures the data; visualization is a separate effort.

**Architecture note:** The semantic router logic is framework-agnostic and would port directly to a React/FastAPI architecture if a forcing function emerges.

---

## Data Governance & Architecture Principles

### Single Source of Truth

**Excel is the master data source.** All story content, STAR fields, and metadata originate in the Excel workbook (`MPugmire - STAR Stories - [DATE].xlsx`).

**Why Excel:**
- Familiar editing environment (no JSON syntax errors)
- Version control via OneDrive/Dropbox/Git
- Easy bulk edits and data validation
- Copy/paste from interview prep notes

**Data Flow:**
```
Excel (Master) → JSONL (Runtime Format) → Pinecone (Search Index) → Application
```

### Hybrid Sovereignty Model

**Excel Owns:**
- All story content (STAR fields, titles, clients, metadata)
- Structural changes (add/remove stories, field updates)
- Business logic (what stories exist, their content)

**JSONL Owns:**
- Derived fields: `public_tags`, `Interview Questions`, `content`
- Runtime state (enrichment artifacts)
- Semantic metadata

**Scripts Preserve:**
- Merge strategy: Excel updates flow in, JSONL-only fields preserved
- Backup on write: `.bak` files prevent data loss
- No destructive overwrites

### January 2026 Sovereignty Patterns

**1. Dynamic Identity (MATT_DNA)**

Professional narrative and sacred vocabulary derived from JSONL at startup:

```python
# Extracted from "About Matt" story in JSONL
MATT_DNA = {
    "identity": ["builder"]  # Only verbatim-required term (March 2026 cleanup)
}
```

**Why Dynamic:**
- Single source of truth (JSONL "About Matt" story)
- No hardcoded narrative in code
- Update story → updates identity everywhere

**2. Multi-Field Entity Search**

Entity detection searches across 6 fields via Pinecone `$or` operator:

```python
# Query: "Show me Accenture projects"
# Searches: Client OR Employer OR Division OR Project OR Place OR Title
pinecone_filter = {
    "$or": [
        {"client": {"$eq": "Accenture"}},
        {"employer": {"$eq": "Accenture"}},
        {"division": {"$eq": "accenture"}},  # lowercase
        {"project": {"$eq": "accenture"}},   # lowercase
        {"place": {"$eq": "accenture"}},     # lowercase
        {"title": {"$eq": "Accenture"}}      # PascalCase
    ]
}
```

**Field-Specific Casing (Pinecone Metadata):**
- **Lowercase:** `division`, `employer`, `project`, `place`, `industry`, `complexity`
- **PascalCase:** `client`, `role`, `title`, `domain`

**3. UI Hydration**

All landing page counts and lists derived dynamically from story data:

```python
# No hardcoded counts - derive from loaded stories
banking_count = len([s for s in stories if s.get("Industry") == "Financial Services"])
clients = {s.get("Client") for s in stories if not is_generic_client(s.get("Client"))}
```

**Why Dynamic:**
- Add story to Excel → counts update automatically
- No stale "130 projects" text to maintain
- Data-driven, self-documenting

### Anti-Patterns (Don't Do This)

**❌ Hardcoded Lists:**
```python
# BAD: Will drift out of sync with data
CLIENTS = ["JP Morgan", "Capital One", "RBC"]
```

**✅ Derived from Data:**
```python
# GOOD: Always accurate
clients = {s.get("Client") for s in stories if not is_generic_client(s.get("Client"))}
```

**❌ Manual JSONL Editing for Content:**
- Don't hand-edit JSONL files for story content changes
- Content changes belong in Excel (master source)
- JSONL editing allowed ONLY for derived fields: `public_tags`, `Interview Questions`, `content`

**✅ Excel-First Workflow:**
1. Edit story in Excel
2. Run `generate_jsonl_from_excel.py`
3. Review diff, commit if correct

**❌ Hardcoded Story Titles in Tests:**
```python
# BAD: Titles change frequently
expected_title = "Implementing Responsible AI Governance Framework"
```

**✅ Index-Based Test Selection:**
```python
# GOOD: Stable reference
story_index = 0  # Select by index, build query from actual title
```

### Data Refresh Workflow

**Full Refresh (Content Changes):**
1. Edit Excel master sheet
2. Run Stage 1: `generate_jsonl_from_excel.py`
3. Run Stage 2: `generate_public_tags.py`
4. Run Stage 3: `build_custom_embeddings.py`
5. Restart app

**Time:** ~2 minutes end-to-end

**See:** [Data Pipeline Documentation](/mattgpt-design-spec/docs/12-data-pipeline) for complete ingestion workflow.

---

## Future Consideration: React Rebuild

**Status:** Deferred — no forcing function currently justifies a rewrite
**Purpose:** Better performance and maintainability if scaling or design constraints emerge

The Streamlit MVP validated the RAG architecture and UX patterns, and has continued to absorb new features (Role Match, mobile responsive, query analytics, eval framework) without hitting structural walls. A React rebuild remains a future option if specific requirements arise that Streamlit can't meet. See [Migration Architecture](/mattgpt-design-spec/docs/13-migration-architecture) for the detailed approach.

### Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| ⚛️ **Frontend** | React + Next.js | Modern, performant UI framework |
| 🚀 **Backend** | FastAPI | High-performance async API |
| 🔍 **RAG Pipeline** | Same as Phase 1 | Proven architecture, no changes needed |

### What Changes

- **UI Framework:** Streamlit → React (better performance, component reusability)
- **API Layer:** Monolithic app.py → FastAPI microservices
- **Mobile:** Responsive CSS → Mobile-first from the start

### What Stays the Same

- Semantic search with confidence scoring
- Semantic router and nonsense detection
- STAR data model and 5P taxonomy
- Pinecone vector database
- OpenAI embeddings and LLM

**Why React?** Modern component architecture, better mobile experience, easier to maintain long-term.

---

## Strategic Rationale

### 1. De-Risk with MVP-First Thinking

**Decision:** Start with Streamlit monolith instead of production-ready stack

**Rationale:**
- Validated RAG architecture effectiveness before infrastructure investment
- Tested system prompt design with real users
- Refined user experience patterns
- Minimized wasted effort on premature optimization

**Result:** 2-week build time vs 3+ months for React-based solution

---

### 2. Preserve Core IP Across Phases

**Decision:** Keep data pipeline architecture unchanged across all phases

**What Stays Constant:**
- Embedding generation process
- Pinecone indexing strategy
- STAR taxonomy and metadata
- Search relevance algorithms

**What Evolves:**
- Presentation layer (Streamlit → React)
- API architecture (monolith → microservices)
- Infrastructure (local → cloud)

**Benefit:** Minimizes technical debt and preserves investment in core IP

---

### 3. Demonstrate Trade-Off Awareness

**MVP Limitations Were Intentional Shortcuts:**

| Limitation | Business Justification |
|-----------|----------------------|
| Desktop-only | Target users (recruiters, hiring managers) primarily use desktop |
| 100-user limit | Portfolio showcase doesn't require scale |
| Monolithic architecture | Faster development, simpler deployment |
| No caching | Query latency acceptable for MVP validation |

**vs. Accidental Technical Debt:**
- Not "ran out of time to implement mobile"
- Not "didn't know how to scale"
- **Consciously traded features for speed**

---

### 4. Align Architecture to Business Goals

**Phase 1: Portfolio Showcase (Current)**
- **Purpose:** Demonstrate technical capability to employers
- **Stack:** Streamlit monolith - fast to build, sufficient for use case
- **Result:** Validated RAG architecture and UX in 2 weeks vs 3+ months

**Future: React + FastAPI (if forcing function emerges)**
- **Purpose:** Better performance and maintainability at scale
- **Stack:** React + FastAPI — modern tooling, mobile-first
- **Benefit:** Preserve core IP (RAG pipeline), improve presentation layer
- **Current status:** Deferred — Streamlit continues to meet all functional requirements

**Principle:** Start simple, evolve intentionally — and don't rewrite what's working

---

## Cost & Performance

### Embedding Generation

**OpenAI text-embedding-3-small:**
- **Rate:** $0.02 per 1M tokens
- **Story Size:** ~300 tokens average (after text composition)
- **100+ Stories:** ~30,000 tokens = $0.0006 per full re-index
- **Time:** ~30 seconds

**Annual Cost (4 full refreshes/month):**
- 4 refreshes × 12 months × $0.0006 = **$0.029/year**
- Effectively free

### Pinecone Vector Database

**matt-portfolio-v2 Index:**
- **Tier:** Starter (free tier, 100K vectors)
- **Usage:** 100+ vectors (<0.1% of quota)
- **Dimensions:** 1536
- **Cost:** $0/month

### LLM Generation (GPT-4o)

**Per Query:**
- **Input tokens:** ~2,000-4,000 (context + prompt)
- **Output tokens:** ~200-600 (response)
- **Cost per query:** ~$0.01-0.03

**Monthly Cost (100 queries/month):**
- 100 queries × $0.02 avg = **$2/month**

**Production Scale (1000 queries/month):**
- 1000 queries × $0.02 avg = **$20/month**

### Processing Performance

**Full Pipeline (Excel → Production):**
- Stage 1 (Excel → JSONL): ~5 seconds
- Stage 2 (Enrichment): ~10 seconds
- Stage 3 (Embeddings): ~30 seconds
- **Total:** ~45 seconds

**Semantic Search (Runtime):**
- Query embedding: ~200ms
- Pinecone vector search: ~300ms
- LLM generation: ~3-5 seconds (GPT-4o)
- **Total query latency:** ~4-6 seconds

**Key Insight:** Cost-optimized architecture with negligible infrastructure costs. Primary expense is LLM generation at ~$0.02 per query. Full data refresh costs less than $0.001.

---

## Current State Summary

### What's Live Today (June 2026)

- ✅ Production Streamlit application at [askmattgpt.streamlit.app](https://askmattgpt.streamlit.app)
- ✅ **5-stage RAG pipeline** with 100% eval pass rate (64/64)
- ✅ **GPT-4o** primary LLM (upgraded from GPT-4o-mini)
- ✅ **Semantic router** with 15 intent families + out-of-scope/personal detection
- ✅ **Role Match** — JD-to-experience matching (Phases 1-3: recruiter view, evidence chips, action buttons; Phase 4 slice 1: lock icon + password gate for private view)
- ✅ **Query analytics** — 32-column event logger to Google Sheets (assessment, chip click, action button, UTM attribution)
- ✅ **Triage agent surface** — `scripts/assess_jd.py` CLI wraps `jd_assessor.py` for external agent orchestration (engine-as-adapter pattern)
- ✅ 100+ STAR-structured project stories
- ✅ Semantic search with confidence scoring and metadata filtering
- ✅ **Timeline View** with Era-based career progression
- ✅ **Mobile-responsive design** (1,520 lines in `mobile_overrides.py`; breakpoints: 767px, 1024px)
- ✅ **Dark mode support** via CSS variables
- ✅ **Modular architecture** (9-file ask_mattgpt/ structure)
- ✅ **BDD test suite** — 219 Playwright scenarios across 30 feature files
- ✅ Conversation history and context management
- ✅ Related Projects UX pattern

### Future Consideration: React Migration

If a forcing function emerges (scaling limit, design constraint Streamlit can't meet, external requirement):

- React + Next.js frontend rebuild
- FastAPI backend
- Mobile-first design from the start
- Same RAG pipeline, better UI

See [Migration Architecture](/mattgpt-design-spec/docs/13-migration-architecture) for the detailed approach.

---

## Technical Diagrams

### Site Navigation Flow

![Site Navigation Diagram](../images/architecture/site-navigation-diagram.png)

### Explore Stories Views

![Explore Stories Diagram](../images/architecture/explore-stories-diagram.png)

### Ask MattGPT Flow

![Ask MattGPT Diagram](../images/architecture/askmatt-flow-diagram.png)

### Additional Resources

- [RAG Architecture Diagram](../images/architecture/tech_rag_architecture.png) - Complete RAG lifecycle
- [Site Architecture](../images/architecture/site_architecture_updated.md) - Page hierarchy and navigation (December 2025)
- [Interactive Wireframes](../wireframes/) - Complete UI/UX wireframe set
- [Architecture Evolution Slide](../wireframes/architecture_evolution_slide_wireframe.html) - Visual roadmap timeline

---

## Key Takeaways

1. **MVP-first approach** validated RAG architecture in weeks instead of months
2. **Core IP preservation** - RAG pipeline remains unchanged, only UI evolves
3. **Conscious trade-offs** - Streamlit limitations were intentional, not accidental
4. **Honest roadmap** - Phase 2 React rebuild for better UX, not fantasy enterprise features

---

**Related Documentation:**
- [Product Vision](/mattgpt-design-spec/docs/01-product-vision) - Strategic positioning
- [UX Design Process](/mattgpt-design-spec/docs/03-ux-design-process) - Design decisions
- [Building MattGPT](/mattgpt-design-spec/docs/04-building-mattgpt) - Development journey
- [RAG Quality Evaluation](/mattgpt-design-spec/docs/11-testing-and-quality) - Eval framework (64/64, 100%)

---

*Last Updated: June 2026 (Staleness audit: eval 100%/64q, sacred vocab builder-only, MATT_DNA, BDD 219/30, story count 100+)*
*Version: 1.5*
