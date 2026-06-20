# API Reference

**Technical reference for llm_portfolio_assistant codebase**

> This document covers the function signatures, parameters, return types, and data flow for every service, utility, and UI module in the codebase. Check here before writing any code that touches semantic routing, RAG retrieval, story filtering, or session state.

---

## Module Overview

### Services Layer (`services/`)

| Module | Purpose |
|--------|---------|
| `jd_assessor.py` | Role Match engine — JD extraction, requirement retrieval, assessment, `compute_recommendation()` |
| `query_logger.py` | 32-column event logger to Google Sheets (query, feedback, Role Match, UTM) |
| `role_match_summary.py` | Role Match summary generation and scoring helpers |
| `semantic_router.py` | Query intent classification using embedding similarity |
| `rag_service.py` | Semantic search orchestration and confidence gating |
| `pinecone_service.py` | Vector database integration for semantic search |
| `story_service.py` | Story metadata and data management (stub) |

### Utilities Layer (`utils/`)

| Module | Purpose |
|--------|---------|
| `client_utils.py` | Generic client detection and known client set helpers |
| `filters.py` | Story filtering (industry, capability, client, domain, role, tags) |
| `formatting.py` | Text formatting and STAR story presentation |
| `landing_cards.py` | Landing page card data helpers |
| `scoring.py` | Hybrid scoring (semantic + keyword relevance) |
| `search.py` | Explore Stories search and filter orchestration |
| `ui_helpers.py` | Session state management and UI rendering helpers |
| `validation.py` | Input validation and token overlap checking |

### UI Components (`ui/components/`)

| Component | Purpose |
|-----------|---------|
| `ask_mattgpt_header.py` | Ask Agy header with branding |
| `action_buttons.py` | Shared action button components |
| `story_detail.py` | Story detail modal with STAR format |
| `how_agy_dialog.py` | "How Agy Searches" dialog |
| `how_i_built_dialog.py` | "How I Built MattGPT" dialog |
| `why_agy_dialog.py` | "Why Agy?" dialog |
| `timeline_view.py` | Era-based timeline visualization |
| `category_cards.py` | Industry/capability card displays |
| `hero.py` | Homepage hero section |
| `navbar.py` | Top navigation bar |
| `lock_icon.py` | Role Match private-view lock indicator |
| `thinking_indicator.py` | AI thinking/loading indicator |
| `footer.py` | Footer component |

### UI Pages (`ui/pages/`)

| Page | Purpose |
|------|---------|
| `explore_stories.py` | My Work — story browsing with table/card/timeline views |
| `ask_mattgpt/` | Ask Agy — conversational AI interface ({{ site.data.facts.ask_mattgpt_module_file_count }}-file module) |
| `about_matt.py` | My Profile page |
| `role_match.py` | Role Match — JD-to-experience fit assessment |
| `banking_landing.py` | Banking industry landing page |
| `cross_industry_landing.py` | Cross-industry landing page |
| `home.py` | Homepage router |

**Ask Agy Submodules (ask_mattgpt/):**
- `__init__.py` - Module router
- `backend_service.py` - Chat backend and LLM orchestration
- `styles.py` - Chat-specific CSS
- `conversation_helpers.py` - Conversation logic
- `conversation_view.py` - Conversation UI rendering
- `landing_view.py` - Landing view with capability cards
- `prompts.py` - System prompts and LLM prompt templates
- `story_intelligence.py` - Story analysis and persona inference
- `shared_state.py` - Shared conversation state
- `utils.py` - Conversation utilities

---

## Key Functions

### Semantic Router (`services/semantic_router.py`)

**Intent Classification with Dual Thresholds**

```python
is_portfolio_query_semantic(
    query: str,
    hard_threshold: float = 0.80,
    soft_threshold: float = 0.40
) -> tuple[bool, float, str, str]
```

**Parameters:**
- `query` - User's query string
- `hard_threshold` - Score above this = clearly valid (default 0.80)
- `soft_threshold` - Score above this = valid but borderline (default 0.40, lowered from 0.72 in Jan 2026)

**Returns:**
- `is_valid` (bool) - True if query >= soft_threshold
- `max_similarity` (float) - Highest cosine similarity score
- `best_intent` (str) - Best matching canonical intent
- `intent_family` (str) - Intent category (e.g., "background", "behavioral")

**Intent Families (15 categories):**
- `background` - Career overview questions
- `behavioral` - STAR behavioral interview questions
- `delivery` - Delivery transformation and results
- `team_scaling` - Team building and scaling experience
- `leadership` - Leadership style and philosophy
- `technical` - Platform engineering and cloud work
- `domain_payments` - Financial services / banking projects
- `domain_healthcare` - Healthcare / life sciences projects
- `stakeholders` - Stakeholder management
- `innovation` - Innovation center and innovation leadership
- `agile_transformation` - Agile and digital transformation
- `narrative` - Professional identity and career narrative
- `synthesis` - Cross-cutting themes and patterns
- `out_of_scope` - Off-domain queries (retail, sports, etc.)
- `personal` - Personal questions (age, family, salary, identity)

**Helper Functions:**

```python
get_intent_families() -> list[str]
# Returns list of all intent family names

get_intents_by_family(family: str) -> list[str]
# Returns canonical intents for a specific family

warm_cache()
# Pre-computes and caches intent embeddings to data/intent_embeddings.json
```

---

### RAG Service (`services/rag_service.py`)

**Semantic Search Orchestration**

```python
semantic_search(
    query: str,
    filters: dict,
    *,
    enforce_overlap: bool = False,
    min_overlap: float = 0.0,
    stories: list,
    top_k: int = 10  # SEARCH_TOP_K
) -> dict
```

**Parameters:**
- `query` - Search query string
- `filters` - Filter dictionary (industry, capability, client, etc.)
- `enforce_overlap` - If True, require token overlap with known vocabulary
- `min_overlap` - Minimum overlap ratio (0.0-1.0)
- `stories` - Full list of story dictionaries
- `top_k` - Max results to return

**Returns:**
```python
{
    "results": [story, ...],    # Stories with "pc" score attached
    "confidence": str,           # "high" | "low" | "none"
    "top_score": float,         # Highest Pinecone similarity score
    "relaxed_count": int,       # Stories passing relaxed (no-filter) search
    "active_filters": dict      # Filters that were applied to the query
}
```

**Confidence Thresholds:**
- `CONFIDENCE_HIGH = 0.25` - Strong match, show "Found X stories"
- `CONFIDENCE_LOW = 0.20` - Borderline, show "Relevance may be low" (raised from 0.15 in Jan 2026)
- Below 0.20 = "none" - Show "No strong matches"

**Vocabulary Initialization:**

```python
initialize_vocab(stories: list[dict])
# Builds token vocabulary from story corpus
# Call once at app startup
```

---

### Scoring (`utils/scoring.py`)

**Hybrid Scoring (Semantic + Keyword)**

```python
_hybrid_score(
    pc_score: float,
    kw_score: float,
    w_pc: float = 1.0,
    w_kw: float = 0.0
) -> float
```

**Parameters:**
- `pc_score` - Pinecone semantic similarity (0.0-1.0)
- `kw_score` - Keyword overlap score (0.0-1.0)
- `w_pc` - Weight for semantic score (default 1.0)
- `w_kw` - Weight for keyword score (default 0.0, disabled)

**Returns:**
- Weighted hybrid score: `(pc_score * w_pc) + (kw_score * w_kw)`

**Keyword Scoring:**

```python
_keyword_score_for_story(s: dict, query: str) -> float
# BM25-ish token overlap scoring
# Returns 0.0-1.0 based on query/story field matches
# Double-weights title and domain matches
```

---

### Filters (`utils/filters.py`)

**Story Filtering with Multi-Criteria**

```python
matches_filters(s: dict, F: dict | None = None) -> bool
```

**Parameters:**
- `s` - Story dictionary (raw JSONL fields)
- `F` - Filters dictionary (if None, reads from `st.session_state["filters"]`)

**Filter Types:**

| Filter Key | Type | Field Matched | Logic |
|-----------|------|---------------|-------|
| `industry` | str | `Industry` | Exact match |
| `capability` | str | `Solution / Offering` | Exact match |
| `clients` | list[str] | `Client` | OR logic |
| `domains` | list[str] | `Sub-category` | OR logic |
| `roles` | list[str] | `Role` | OR logic |
| `tags` | list[str] | `public_tags` | OR logic (case-insensitive) |
| `has_metric` | bool | Performance bullets | Check for quantified metrics |
| `q` | str | Multiple fields | Token-based ALL match |

**Returns:**
- `True` if story passes ALL active filters
- `False` if any filter fails

---

## Data Flow Summary

### 1. User Query → Semantic Router → RAG Pipeline

```
┌─────────────────┐
│  User submits   │
│  query via UI   │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  Nonsense Filter (nonsense_filters.jsonl│
│  - Regex pattern match against query    │
│  - Blocks off-topic / abusive queries   │
└────────┬────────────────────────────────┘
         │
         └─ [match] → REJECT immediately
         │
         ▼
┌─────────────────────────────────────────┐
│  Semantic Router (semantic_router.py)   │
│  - Embed query (OpenAI text-embedding)  │
│  - Compare to canonical intents         │
│  - Return (is_valid, score, family)     │
└────────┬────────────────────────────────┘
         │
         ├─ [score >= 0.40] → ACCEPT
         └─ [score < 0.40]  → REJECT (show off-domain response)
         │
         ▼
┌─────────────────────────────────────────┐
│  RAG Service (rag_service.py)           │
│  - semantic_search(query, filters)      │
│  - Entity Detection (Client/Employer/   │
│    Division) — pins matching story #1   │
│  - Calls Pinecone with entity filters   │
│  - Applies confidence gating            │
│  - Filters results by metadata          │
└────────┬────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  Pinecone Service (pinecone_service.py) │
│  - Query vector index                   │
│  - Return top_k matches with scores     │
└────────┬────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  Filters (utils/filters.py)             │
│  - Apply industry/capability filters    │
│  - Apply advanced filters               │
│  - matches_filters() for each story     │
└────────┬────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  Response to User                       │
│  - Ranked stories with confidence       │
│  - Source citations with scores         │
│  - Related projects suggestions         │
└─────────────────────────────────────────┘
```

### 2. Explore Stories Filtering

```
┌─────────────────┐
│  User selects   │
│  filters in UI  │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  Filter State (st.session_state)        │
│  {"industry": "...",                    │
│   "capability": "...",                  │
│   "clients": [...], ...}                │
└────────┬────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  matches_filters() (utils/filters.py)   │
│  - Check each story against filters     │
│  - AND logic: all filters must pass     │
└────────┬────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  Filtered Results Display               │
│  - Table/Card/Timeline views            │
│  - Story detail modals                  │
└─────────────────────────────────────────┘
```

### 3. Confidence Gating Logic

**Query Classification (Semantic Router):**
```
Score >= 0.80  → HARD_ACCEPT (clearly on-topic)
Score >= 0.40  → SOFT_ACCEPT (borderline, log for review - lowered from 0.72 in Jan 2026)
Score <  0.40  → REJECT (off-domain, show suggestions)
```

**Result Confidence (RAG Service):**
```
Top score >= 0.25  → HIGH confidence ("Found X stories")
Top score >= 0.20  → LOW confidence ("Relevance may be low")
Top score <  0.20  → NONE ("No strong matches")
```

---

## Session State Management

Key session state variables used throughout the application:

| Key | Type | Purpose |
|-----|------|---------|
| `active_tab` | str | Current page ("Home", "My Work", "Ask Agy", "Role Match", "My Profile") |
| `filters` | dict | Active filters (industry, capability, clients, domains, etc.) |
| `__pc_last_ids__` | dict | Story ID → Pinecone score mapping |
| `__pc_snippets__` | dict | Story ID → snippet text mapping |
| `__last_ranked_sources__` | list | Ordered list of story IDs from last search |
| `__pc_suppressed__` | bool | True if Pinecone results were suppressed (low confidence) |
| `__dbg_pc_hits` | int | Number of Pinecone hits (for debugging) |
| `ask_transcript` | list | Ask Agy chat history |
| `selected_capability` | str | Selected capability for homepage filtering |

---

## Environment Variables

Required environment variables (see `.env.example`):

```bash
# OpenAI API Configuration
OPENAI_API_KEY=sk-...
OPENAI_PROJECT_ID=proj_...
OPENAI_ORG_ID=org-...

# Pinecone Configuration
PINECONE_API_KEY=...
PINECONE_INDEX_NAME=matt-portfolio-v2

# Data File (optional, defaults to echo_star_stories_nlp.jsonl)
STORIES_JSONL=echo_star_stories_nlp.jsonl
```

---

## Error Handling

**Semantic Router:**
- Fails open if embedding fails (returns `(True, 1.0, "", "error_fallback")`)
- Logs borderline queries to `data/borderline_queries.csv`

**RAG Service:**
- Returns empty results for invalid queries
- Gracefully handles Pinecone failures with local fallback
- Preserves UI state even when search fails

**Filters:**
- Handles missing fields gracefully (defaults to empty strings/lists)
- Case-insensitive tag matching
- Token-based matching with fallback to substring search

---

## Testing

**Test Coverage:** {{ site.data.facts.unit_test_file_count }} test modules covering services and utilities (selected modules below)

Run tests from project root:

```bash
# All unit tests
pytest tests/unit/

# Specific module tests
pytest tests/unit/test_semantic_router.py    # Intent classification tests
pytest tests/unit/test_filters.py            # Filter logic tests
pytest tests/unit/test_scoring.py            # Hybrid scoring tests
pytest tests/unit/test_formatting.py         # STAR formatting tests
pytest tests/unit/test_validation.py         # Input validation tests
pytest tests/unit/test_backend_service.py    # Ask MattGPT backend tests
pytest tests/unit/test_story_intelligence.py # Story analysis tests

# With coverage
pytest --cov=services --cov=utils tests/
```

---

## Performance Considerations

**Semantic Router:**
- Embeddings cached to `data/intent_embeddings.json` (2MB)
- Warm cache at startup with `warm_cache()`
- Cosine similarity: O(n) where n = number of canonical intents ({{ site.data.facts.intent_total_count }})

**RAG Service:**
- Pinecone queries: ~200-500ms
- Confidence gating prevents returning low-quality results
- Local fallback for offline/error scenarios

**Filters:**
- O(n) where n = number of stories (100+)
- Token-based matching faster than regex for keyword search
- All filters applied in single pass

---

## Related Documentation

- [Product Vision](/docs/01-product-vision) - Strategic positioning and user personas
- [Technical Architecture](/docs/02-technical-architecture) - Two-phase roadmap and RAG architecture
- [Data Model](/docs/10-data-model) - JSONL schema and field definitions

---

*Last Updated: June 2026 (Staleness Audit Refresh)*
*Version: 1.1*
