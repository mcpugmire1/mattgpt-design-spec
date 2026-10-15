# API Reference

**Technical reference for llm_portfolio_assistant codebase**

> This document covers the function signatures, parameters, return types, and data flow for every service, utility, and UI module in the codebase. Check here before writing any code that touches semantic routing, Ask Agy's answer path, RAG retrieval, story filtering, or session state.

---

## Module Overview

### Services Layer (`services/`)

| Module | Purpose |
|--------|---------|
| `jd_assessor.py` | Role Match engine: JD extraction, requirement retrieval, assessment, `compute_recommendation()` |
| `query_logger.py` | Event logger to Google Sheets (query, feedback, Role Match, UTM) |
| `matt_profile.py` | The one reader of `data/matt_profile.json`, shared by Role Match, the assessor and Ask Agy's profile facts |
| `role_match_summary.py` | Role Match summary generation and scoring helpers |
| `semantic_router.py` | Query intent classification using embedding similarity |
| `rag_service.py` | Semantic search and confidence labeling (My Work gates its banners on the label; Ask Agy does not) |
| `pinecone_service.py` | Vector database integration for semantic search |
| `story_service.py` | Story metadata and data management (stub) |

### Utilities Layer (`utils/`)

| Module | Purpose |
|--------|---------|
| `client_utils.py` | Generic client detection and known client set helpers |
| `corpus_loader.py` | Story loading and normalization without Streamlit, for eval harnesses and scripts |
| `filters.py` | Story filtering (industry, capability, client, domain, role, tags) |
| `formatting.py` | Text formatting and STAR story presentation |
| `landing_cards.py` | Landing page card data helpers |
| `scoring.py` | Hybrid scoring (semantic + keyword relevance) |
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
| `explore_stories.py` | My Work: story browsing with table/card/timeline views |
| `ask_mattgpt/` | Ask Agy: conversational AI interface ({{ site.data.facts.ask_mattgpt_module_file_count }}-file module) |
| `about_matt.py` | My Profile page |
| `role_match.py` | Role Match: JD-to-experience fit assessment |
| `banking_landing.py` | Banking industry landing page |
| `cross_industry_landing.py` | Cross-industry landing page |
| `home.py` | Homepage router |

**Ask Agy Submodules (ask_mattgpt/):**
- `__init__.py` - Module router
- `backend_service.py` - Ask Agy backend: `send_to_backend()`, `agy_answer()` and its tool loop
- `styles.py` - Chat-specific CSS
- `conversation_helpers.py` - Conversation logic
- `conversation_view.py` - Conversation UI rendering
- `landing_view.py` - Landing view with capability cards
- `prompts.py` - System prompts and LLM prompt templates
- `story_intelligence.py` - Theme inference and story context for the search tool
- `shared_state.py` - Shared conversation state
- `utils.py` - Conversation utilities

---

## Key Functions

### Semantic Router (`services/semantic_router.py`)

**Intent Classification with Dual Thresholds**

```python
is_portfolio_query_semantic(
    query: str,
    hard_threshold: float = HARD_ACCEPT,
    soft_threshold: float = SOFT_ACCEPT,
) -> tuple[bool, float, str, str]
```

**Parameters:**
- `query` - User's query string
- `hard_threshold` - Score above this = clearly valid (default `HARD_ACCEPT`, {{ site.data.facts.router_hard_accept }})
- `soft_threshold` - Score above this = valid but borderline (default `SOFT_ACCEPT`, {{ site.data.facts.router_soft_accept }}; lowered from 0.72 in Jan 2026)

**Returns:**
- `is_valid` (bool) - True if query >= soft_threshold
- `max_similarity` (float) - Highest cosine similarity score
- `best_intent` (str) - Best matching canonical intent
- `intent_family` (str) - Intent category (e.g., "background", "behavioral")

**Intent Families ({{ site.data.facts.intent_family_count }} categories):**
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
- `personal` - Personal questions (age, family, identity); salary questions are caught earlier by the `personal_compensation` nonsense rule

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

### Ask Agy Backend (`ui/pages/ask_mattgpt/backend_service.py`)

**Entry point and answer loop**

```python
send_to_backend(
    prompt: str,
    filters: dict,             # unused
    ctx: dict | None,          # unused
    stories: list[dict],
    on_delta: Callable[[str], None] | None = None,
) -> dict

agy_answer(
    question: str,
    stories: list[dict],
    history: list[dict[str, str]] | None = None,
    on_delta: Callable[[str], None] | None = None,
) -> dict
```

`send_to_backend()` builds `history` from `st.session_state["ask_transcript"]` with `_build_history_messages()` (the last `ASK_HISTORY_MAX_MESSAGES` messages, banners skipped) and calls `agy_answer()`. `agy_answer()` runs the nonsense rules and the router's out_of_scope and personal redirects, then the tool loop on `ASK_ANSWER_MODEL`. `on_delta` receives the streamed answer text, with `normalize_dashes()` applied.

**Returns:** `answer_md`, `sources` (`{"id", "title", "client"}` per story the searches returned), `modes`, `default_mode`, `intent_family`, `rejection_reason`, and on answers and errors `profile_categories` and `degraded`. `confidence` and `entity_match` are always `None`. The app repo's `ARCHITECTURE.md` (agy_answer Return Contract) is the source.

**Search tool:**

```python
_run_search_tool(arguments: str, stories: list[dict]) -> tuple[str, list[dict], float]
# One search_stories call: semantic_search(query, {}, stories=stories, top_k=SEARCH_TOP_K),
# the top results each as <story title="...">, a Professional Narrative story with
# get_verbatim_requirement() appended. Returns (tool output, stories, top score);
# the output is "No stories found." when nothing is returned.
```

**System prompt:** `build_tool_system_prompt(is_synthesis, matt_dna, profile_facts, opener_examples, focus_angle="")` in `prompts.py`: the mode's system prompt, `TOOL_STORY_NOTE`, then `build_answer_instructions()`, which carries `NO_DASH_RULE` and `COMPARISON_RULE`.

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
    top_k: int = SEARCH_TOP_K  # {{ site.data.facts.search_top_k }}
) -> dict
```

**Parameters:**
- `query` - Search query string
- `filters` - Filter dictionary (industry, capability, client, etc.)
- `enforce_overlap` - If True, require token overlap with known vocabulary (no current caller sets it)
- `min_overlap` - Minimum overlap ratio (0.0-1.0) (no current caller sets it)
- `stories` - Full list of story dictionaries
- `top_k` - Max results to return

**Returns:**
```python
{
    "results": [story, ...],    # Stories with "pc" score attached
    "confidence": str,           # "high" | "low" | "none"
    "top_score": float,         # Highest Pinecone similarity score
    "relaxed_count": int,       # Optional: results without industry/capability filters
    "active_filters": list,     # Optional: (name, value) tuples for the filters that left 0 results
    "profile_facts": ...,       # Present on every return: profile facts for the answer
}
```

`relaxed_count` and `active_filters` appear only when the industry or capability filters leave no results.

**Confidence Thresholds:**
- `CONFIDENCE_HIGH` ({{ site.data.facts.confidence_high }}) - "high"
- `CONFIDENCE_LOW` ({{ site.data.facts.confidence_low }}) - "low". My Work shows results with a "Relevance may be low" note (raised from 0.15 in Jan 2026)
- Below `CONFIDENCE_LOW` - "none". No story results: My Work shows "No strong matches", and Ask Agy's search tool returns "No stories found."
- Ask Agy reads only the results and `top_score`; the label does not decide whether it answers
- When Pinecone returns no hits, `semantic_search()` falls back to the corpus in file order with confidence "low" (MATTGPT-292 in the app backlog)

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
    w_pc: float = W_PC,
    w_kw: float = W_KW
) -> float
```

**Parameters:**
- `pc_score` - Pinecone semantic similarity (0.0-1.0)
- `kw_score` - Keyword overlap score (0.0-1.0)
- `w_pc` - Weight for semantic score (default `W_PC` in `config/constants.py`, currently 1.0)
- `w_kw` - Weight for keyword score (default `W_KW`, currently 0.15; raised from 0.0 in Aug 2026)

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
| `era` | str | `Era` | Exact match (Timeline navigation) |
| `personas` | list[str] | `personas` | OR logic (no story currently has this field) |
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

### 1. Ask Agy Query → Answer

```
┌─────────────────┐
│  User submits   │
│  query via UI   │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  Nonsense Rules (nonsense_filters.jsonl)│
│  - Regex pattern match against query    │
│  - Skipped for suggestion chips         │
└────────┬────────────────────────────────┘
         │
         ├─ [match] → REJECT (banner + chips)
         │
         ▼
┌─────────────────────────────────────────┐
│  Semantic Router (semantic_router.py)   │
│  - Embed query, compare to intents      │
│  - Return (is_valid, score, family)     │
└────────┬────────────────────────────────┘
         │
         ├─ [out_of_scope or personal, score >= HARD_ACCEPT]
         │       → REJECT (banner + chips)
         │  (any other score continues)
         ▼
┌─────────────────────────────────────────┐
│  Tool Loop (agy_answer)                 │
│  - System prompt, recent conversation,  │
│    the question                         │
│  - Model on ASK_ANSWER_MODEL writes     │
│    search_stories queries, up to        │
│    ASK_TOOL_MAX_ROUNDS                  │
└────────┬───────────────────▲────────────┘
         │ search_stories    │ <story> blocks
         ▼                   │
┌─────────────────────────────────────────┐
│  semantic_search(query, {})             │
│  - Pinecone top_k, blended with W_KW    │
│  - Hits below CONFIDENCE_LOW dropped    │
│  - No entity or facet filters, no gate  │
└─────────────────────────────────────────┘
         │ (after the last search)
         ▼
┌─────────────────────────────────────────┐
│  Response to User                       │
│  - Answer streams, then is replaced by  │
│    the post-processed text              │
│  - Sources: stories the searches        │
│    returned                             │
└─────────────────────────────────────────┘
```

### 2. My Work Filtering

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

### 3. Router and Confidence Logic

**Query Classification (Semantic Router):**
```
Score >= HARD_ACCEPT  → clearly on-topic; out_of_scope and personal are rejected here
Score >= SOFT_ACCEPT  → borderline, logged for review (lowered from 0.72 in Jan 2026)
Score <  SOFT_ACCEPT  → low router signal; no rejection, the query goes on
```

`router_rejection_reason()` in `services/semantic_router.py` is the only router rejection, shared by Ask Agy and My Work. Ask Agy has no confidence gate: every other query goes to the tool loop.

**Result Confidence (RAG Service):**
```
Top score >= CONFIDENCE_HIGH  → high
Top score >= CONFIDENCE_LOW   → low: My Work shows "Relevance may be low"
Top score <  CONFIDENCE_LOW   → none: no story results (Ask Agy's tool returns "No stories found.")
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
| `__last_ranked_sources__` | list | Ordered list of story IDs from last search (written by `rag_service.py`; no current reader) |
| `__pc_suppressed__` | bool | True if Pinecone results were suppressed (low confidence) |
| `__dbg_pc_hits` | int | Number of Pinecone hits (for debugging) |
| `ask_transcript` | list | Ask Agy chat history; the source of `_build_history_messages()` |
| `ask_last_reason` / `ask_last_query` | str | Why the last Ask Agy question was redirected, and the question; routes the banner |
| `__ask_from_suggestion__` | bool | Question came from a suggestion chip: skips the nonsense rules and the router |
| `__ask_force_answer__` | bool | Same bypass, set by landing, follow-up and banner suggestion chips |
| `__embed_failure__` | bool | The query embedding failed; Ask Agy returns a breather message |
| `seed_prompt` | str | A question handed to Ask Agy from another page (landing input, chips, story detail) |

---

## Environment Variables

The app reads configuration through `get_conf()` in `config/settings.py`: Streamlit secrets first, then environment variables. `.env.example` lists only `OPENAI_API_KEY`; the full set the app reads is:

```bash
# OpenAI
OPENAI_API_KEY=sk-...
OPENAI_PROJECT_ID=proj_...
OPENAI_ORG_ID=org-...

# Pinecone
PINECONE_API_KEY=...
PINECONE_INDEX_NAME=...
PINECONE_NAMESPACE=...      # no default in code

# Optional
STORIES_JSONL=echo_star_stories_nlp.jsonl
VECTOR_BACKEND=faiss        # default
```

---

## Error Handling

**Semantic Router:**
- Fails open if embedding fails (returns `(True, 1.0, "", "error_fallback")`)
- Logs borderline queries to `data/borderline_queries.csv`

**RAG Service:**
- Returns empty results for invalid queries
- Falls back to the local corpus when Pinecone returns no hits or fails (MATTGPT-292)
- Preserves UI state even when search fails

**Ask Agy (`agy_answer()`):**
- A stream that breaks before any answer text is sent again once; after text, the turn fails
- A rate limit or an embedding failure returns a "quick breather" message
- Any other error returns `ANSWER_ERROR_MESSAGE` (`config/constants.py`) with no sources and `degraded: True`
- The client's read timeout is `ASK_ANSWER_READ_TIMEOUT_S`

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
pytest tests/unit/test_backend_service.py    # Ask Agy backend tests
pytest tests/unit/test_ask_agy_tool_answer.py # Tool loop, streaming, errors
pytest tests/unit/test_story_intelligence.py # Story analysis tests

# With coverage
pytest --cov=services --cov=utils tests/
```

---

## Performance Considerations

**Semantic Router:**
- Embeddings cached to `data/intent_embeddings.json`
- Warm cache at startup with `warm_cache()`
- Cosine similarity: O(n) where n = number of canonical intents ({{ site.data.facts.intent_total_count }})

**RAG Service:**
- Pinecone queries: ~200-500ms
- Hits below `CONFIDENCE_LOW` are dropped
- Local fallback for offline/error scenarios

**Filters:**
- O(n) where n = number of stories (100+)
- Token-based matching faster than regex for keyword search
- All filters applied in single pass

---

## Related Documentation

- [Product Vision](01-product-vision) - Strategic positioning and user personas
- [Technical Architecture](02-technical-architecture) - Two-phase roadmap and RAG architecture
- [Data Model](10-data-model) - JSONL schema and field definitions

---

*Last updated: {{ site.data.page_dates['09-api-reference'] }}*
