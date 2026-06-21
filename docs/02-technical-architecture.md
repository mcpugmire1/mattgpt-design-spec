# Technical Architecture

**MattGPT: Architecture & Technical Evolution**

> This document outlines the technical architecture of MattGPT, demonstrating intentional trade-off awareness from rapid prototyping through production feature delivery.

---

## Architecture Roadmap Overview

MattGPT's architecture evolved through intentional phases:

1. **Phase 1 (Streamlit MVP):** Validate RAG architecture with minimal investment — ✅ Complete
2. **Production (Streamlit):** Ongoing feature delivery — Role Match, mobile responsive, eval framework, query analytics


---

## Phase 1: MVP - Rapid Validation

**Status:** ✅ Complete (December 2025)
**Timeline:** Current State
**Primary Goal:** Validate product-market fit with minimal investment

### Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Frontend/Backend** | Streamlit (Monolith) | Rapid prototyping framework |
| **LLM** | OpenAI {{ site.data.facts.chat_model }} | Natural language processing |
| **Vector Database** | Pinecone | Semantic search and embeddings |
| **Dependencies** | Python venv + pip | Local development environment |

### Accepted Trade-offs

The MVP phase consciously accepted limitations to accelerate learning:

- ~~**Mobile-responsive implementation:**~~ ✅ Shipped — production mobile CSS with breakpoints at 767px (mobile), 768-1024px (tablet), 1024+ (desktop)
- **Limited scalability:** Estimated ~100 concurrent users (not load-tested; Streamlit Community Cloud limits apply)
- **No caching layer:** Direct database queries
- ~~**Modular architecture:**~~ ✅ Shipped — refactored from monolithic app.py to component-based structure
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
- Refactored monolithic ask_mattgpt.py (4,696 lines) into {{ site.data.facts.ask_mattgpt_module_file_count }}-file directory
- Separated concerns: backend_service.py, conversation_view.py, conversation_helpers.py
- Reusable component library (ui/components/)

**Design System:**
- CSS variables for light/dark mode support (global_styles.py)
- Standardized breakpoints: 767px (mobile), 768-1024px (tablet), 1024+ (desktop)
- Consistent purple brand (#8B5CF6) across all views

**Timeline View Innovation:**
- Era-based project grouping (timeline_view.py)
- 5 career eras with date range calculation
- Progressive disclosure pattern (6 most recent per era)

**Mobile Implementation:**
- Responsive CSS across global_styles.py and mobile_overrides.py
- Touch-optimized controls and stacking layouts
- Horizontal scroll tables with preserved functionality

---

## 5-Stage RAG Pipeline

**Status:** ✅ Implemented (January 2026)
**Quality:** 100% eval pass rate (64/64 queries)

MattGPT uses a **5-stage RAG (Retrieval-Augmented Generation) pipeline** to ensure accurate, grounded responses:

### Pipeline Overview

<div class="mermaid">
graph LR
    Q[Query] --> S1["Stage 1<br/>Nonsense Filter"] --> S2["Stage 2<br/>Semantic Router"] --> S3["Stage 3<br/>Confidence Gate"] --> S4["Stage 4<br/>Entity Detection"] --> S5["Stage 5<br/>Intent Ranking"] --> R["LLM Response"]
</div>

**Stage 1: Rules-Based Nonsense Detection**
- Fast regex matching against `nonsense_filters.jsonl` patterns
- Catches obvious off-domain queries (weather, sports, stocks, etc.)
- Zero embedding cost, executes in <5ms
- Blocks 60%+ of nonsense before semantic processing

**Stage 2: Semantic Router (Intent + Out-of-Scope Detection)**
- Embedding-based similarity matching against 15 intent families
- Dual-threshold system: HARD_ACCEPT ({{ site.data.facts.router_hard_accept }}), SOFT_ACCEPT ({{ site.data.facts.router_soft_accept }})
- Detects synthesis queries (no Pinecone search needed)
- Flags out-of-scope industries gracefully
- **Replaced:** Legacy LLM intent classification (removed Jan 2026)

**Stage 3: Confidence Gating**
- Pinecone semantic search with confidence scoring
- Thresholds: HIGH ({{ site.data.facts.confidence_high }}), LOW ({{ site.data.facts.confidence_low }})
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
| **Story Selection** | Top 5 with diversity | Named-clients-first across themes, mode determines final slice |
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

Across his Accenture career, the pattern is consistent: he's brought to projects
where organizations need to build something from nothing or modernize platforms
stuck in technical debt. From scaling the Cloud Innovation Center to 150+ people
to modernizing payments across 12 countries, his work comes with evidence —
real outcomes, not just strategy decks.
```

### MATT_DNA Ground Truth

`MATT_DNA` is generated at startup from the full story corpus — not extracted from a single story. See [Data Pipeline](/docs/12-data-pipeline#january-2026-sovereignty-patterns) for the correct mechanism.

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
- "How has Matt's work evolved across his career?"
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

**Filter Categories (34):**

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
- Pattern matching is O(n) where n = number of patterns (79)
- Executes in <5ms before semantic routing
- Zero embedding cost for obvious nonsense

---

### UX Flow for Off-Domain Queries

**When a query is rejected:**

1. **Semantic router** flags query as below {{ site.data.facts.router_soft_accept }} threshold
2. **Pattern filter** (optional) confirms off-domain category
3. **Agy responds** with helpful redirect:

**Example Response:**

```
🐾 I'm not finding matches for that in Matt's portfolio.

I'm focused on Matt's enterprise transformation experience—things like agile delivery, platform engineering,
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

### How the Two-Stage Gate Works

Off-domain queries are rejected in two stages before reaching Pinecone. Stage 1 — the pattern gate — runs `is_nonsense()`, fast regex matching against `nonsense_filters.jsonl`, before any embedding is generated; queries caught here are rejected at zero embedding cost. Stage 2 — the semantic router — embeds passing queries and scores them against {{ site.data.facts.intent_family_count }} intent families: a score above {{ site.data.facts.router_hard_accept }} is a hard accept, {{ site.data.facts.router_soft_accept }}–{{ site.data.facts.router_hard_accept }} is a soft accept (logged for review), and below {{ site.data.facts.router_soft_accept }} triggers the off-domain response with suggestion chips.

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
- **Lowercase:** `division`, `employer`, `project`, `place`
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

**See:** [Data Pipeline Documentation](/docs/12-data-pipeline) for complete ingestion workflow.
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
- ✅ **Mobile-responsive design** (global_styles.py + mobile_overrides.py; breakpoints: 767px, 1024px)
- ✅ **Dark mode support** via CSS variables
- ✅ **Modular architecture** ({{ site.data.facts.ask_mattgpt_module_file_count }}-file ask_mattgpt/ structure)
- ✅ **BDD test suite** — {{ site.data.facts.bdd_summary }}
- ✅ Conversation history and context management
- ✅ Related Projects UX pattern



---

**Related Documentation:**
- [Product Vision](/docs/01-product-vision) - Strategic positioning
- [UX Design Process](/docs/03-ux-design-process) - Design decisions
- [Building MattGPT](/docs/04-building-mattgpt) - Development journey
- [RAG Quality Evaluation](/docs/11-testing-and-quality) - Eval framework (64/64, 100%)

---

*Last updated: {{ site.data.page_dates['02-technical-architecture'] }}*
