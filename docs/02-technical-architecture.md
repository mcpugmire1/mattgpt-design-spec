# Technical Architecture

**MattGPT: Architecture & Technical Evolution**

> This document outlines the technical architecture of MattGPT, demonstrating intentional trade-off awareness from rapid prototyping through production feature delivery.

---

## Architecture Roadmap Overview

MattGPT's architecture evolved through intentional phases:

1. **Phase 1 (Streamlit MVP):** Validate RAG architecture with minimal investment: Complete
2. **Production (Streamlit):** Ongoing feature delivery: Role Match, mobile responsive, eval framework, query analytics


---

## Phase 1: MVP - Rapid Validation

**Status:** Complete (December 2025)
**Timeline:** Current State
**Primary Goal:** The goal was to validate the RAG approach with minimal investment before committing to a heavier stack.

### Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Frontend/Backend** | Streamlit (Monolith) | Rapid prototyping framework |
| **LLM** | OpenAI (Ask Agy's current answer model is `ASK_ANSWER_MODEL` in `config/constants.py`) | Natural language processing |
| **Vector Database** | Pinecone | Semantic search and embeddings |
| **Dependencies** | Python venv + pip | Local development environment |

### Accepted Trade-offs

The MVP phase consciously accepted limitations to accelerate learning:

- ~~**Mobile-responsive implementation:**~~ Shipped: production mobile CSS with breakpoints at 767px (mobile), 768-1024px (tablet), 1024+ (desktop)
- **Limited scalability:** Estimated ~100 concurrent users (not load-tested; Streamlit Community Cloud limits apply)
- **No caching layer:** Direct database queries
- ~~**Modular architecture:**~~ Shipped: refactored from monolithic app.py to component-based structure
- ~~**Minimal observability:**~~ Shipped: query logger to Google Sheets capturing events, intent, UTM attribution, and Role Match outcomes

### Key Decisions

**Why Streamlit?**
- Initial MVP build time: 2 weeks vs 3+ months for React*
- Python-native (matches ML/AI ecosystem)
- Built-in state management
- Rapid iteration without frontend complexity

*Note: Continuous refinement and feature additions ongoing since launch.

**The MVP confirmed the parts that mattered:**
- RAG architecture effectiveness
- System prompt design
- STAR methodology implementation
- User experience patterns
- Search relevance tuning

### Architecture Evolutions Achieved

**October 2026 - Ask Agy on a Tool Loop (MATTGPT-275):**
- `rag_answer()` retired; Ask Agy answers through `agy_answer()`
- The answering model writes its own search queries through a `search_stories` tool, and its answer streams
- Retired with it: the Ask Agy confidence gate, entity detection and pinning, theme-filtered synthesis retrieval, and fixed temperatures
- Added: dash and comparison rules in the prompt, a stream timeout and retry, and story dates in the search tool's output
- Recorded as ADRs 047 to 051 in the app repo's `docs/ADR.md`

**January 2026 - RAG Pipeline Cleanup:**
- 5-stage RAG pipeline, gated by a golden-query eval
- Removed Entity Gate bouncer (was blocking legitimate queries)
- Removed `classify_query_intent()` LLM call (redundant with router)
- Semantic router now handles synthesis + out-of-scope + narrative detection
- Expanded to 15 intent families (was 10)
- Centralized thresholds in `config/constants.py`
- Unified SEARCH_TOP_K = 10 (was 100/7 conflict)
- Title soft filtering (semantic search ranks naturally)
- Model upgrade: GPT-4o-mini → GPT-4o

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
- Career eras with date range calculation
- Progressive disclosure pattern per era

**Mobile Implementation:**
- Responsive CSS across global_styles.py and mobile_overrides.py
- Touch-optimized controls and stacking layouts
- Horizontal scroll tables with preserved functionality

---

## Ask Agy Answer Pipeline

**Status:** Implemented October 2026 (MATTGPT-275), replacing the January 2026 5-stage pipeline
**Quality:** the golden eval ({{ site.data.facts.eval_query_count }} queries) calls `agy_answer()`; no golden eval has run on the shipped pipeline yet (see [Testing and Quality](11-testing-and-quality))

`send_to_backend()` calls `agy_answer()` in `backend_service.py`. The app repo's `ARCHITECTURE.md` (Production RAG Pipeline and Component Contracts) is the source for the values named here.

### Pipeline Overview

<div class="mermaid">
graph LR
    Q[Query] --> S1["Nonsense Rules"] --> S2["Semantic Router"] --> L["Tool Loop<br/>ASK_ANSWER_MODEL"]
    L -->|search_stories| S["semantic_search"]
    S --> L
    L --> R["Streamed Answer"]
</div>

**Step 1: Rules-Based Nonsense Detection**
- Fast regex matching against `nonsense_filters.jsonl` patterns
- Catches obvious off-domain queries (weather, sports, stocks, etc.) and garbage input (`gibberish`, `local_recommendations`), which matters more now that no confidence gate sits behind it
- Zero embedding cost; skipped for suggestion chips

**Step 2: Semantic Router (Intent + Redirects)**
- Embedding-based similarity matching against {{ site.data.facts.intent_family_count }} intent families
- Dual-threshold system: HARD_ACCEPT ({{ site.data.facts.router_hard_accept }}), SOFT_ACCEPT ({{ site.data.facts.router_soft_accept }})
- Redirects out-of-scope and personal queries, but only when the score is at least HARD_ACCEPT; below that, the query goes on to the tool loop
- The `synthesis` family switches the system prompt to synthesis mode; it does not change retrieval
- Fail-open on errors; skipped for suggestion chips

**Step 3: Tool Loop**
- The model gets the system prompt, the last `ASK_HISTORY_MAX_MESSAGES` messages of the conversation, and the question as asked
- The first call must call `search_stories`; later calls, up to `ASK_TOOL_MAX_ROUNDS`, may search again; one more call with no tools must answer
- The model writes each search query itself, from the question and the conversation
- Runs on `ASK_ANSWER_MODEL` on the Responses API, with `ASK_ANSWER_REASONING_EFFORT`; the model takes no temperature

**Step 4: Search (each `search_stories` call)**
- `semantic_search(query, {})` with no facet or entity filters: Pinecone top {{ site.data.facts.search_top_k }}, sorted by a blend of the Pinecone score and a keyword score (`W_KW`)
- Hits below LOW ({{ site.data.facts.confidence_low }}) are dropped; `_run_search_tool()` returns the top of what is left, each story wrapped in its own `<story>` tag so the model can tell them apart
- With nothing left, the tool returns "No stories found." and the model answers from MATT_DNA, the profile and the conversation, or says the record doesn't cover it
- Known issue: when Pinecone returns no hits at all, `semantic_search()` falls back to the corpus in file order (MATTGPT-292 in the app backlog)

**Step 5: Stream, Then Swap**
- Answer text streams with `normalize_dashes()` applied
- When the turn finishes, the streamed text is replaced by `_postprocess_agy_text()`'s output
- Sources are every story the searches returned, deduplicated; source cards show what was retrieved, not what the answer cited

**No longer in Ask Agy:** the confidence gate (search confidence does not decide whether Ask Agy answers), entity detection and pinning, theme-filtered synthesis retrieval, and fixed temperatures. My Work still uses the confidence tiers for its own relevance banners.

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

Retrieval is the same tool loop in both modes. Synthesis changes only the prompt:

| Aspect | Standard Mode | Synthesis Mode |
|--------|--------------|----------------|
| **System prompt** | `STANDARD_DELTA` | `SYNTHESIS_DELTA` |
| **Focus** | A random angle from `FOCUS_ANGLES` | No focus angle; breadth across stories |
| **Opener examples** | The same list in both modes: `SYNTHESIS_OPENERS` plus `STANDARD_OPENERS` | Same |
| **Verbatim identity terms** | Same rule in both modes: required for every Professional Narrative story a search returns | Same |

### Verbatim Identity Terms

For every Professional Narrative story a `search_stories` call returns, `_run_search_tool()` appends `get_verbatim_requirement()` from `prompts.py`, which scans the story's `5PSummary` for a fixed list of Matt's identity phrases (such as "builder" and "modernizer") and requires the answer to use the ones it finds, verbatim. This applies in standard and synthesis mode alike; it is not part of `SYNTHESIS_DELTA`.

**Why Strict:**
- Professional narrative must be consistent across all synthesis queries
- "Builder" is Matt's actual language (from "About Matt" story in JSONL)
- Marketing/branding consistency

**Example Synthesis Response:**

```
Matt is a **builder**, someone brought in to create what doesn't exist yet,
whether that's a team, a platform, or a capability.

Across his Accenture career, the pattern is consistent: he's brought to projects
where organizations need to build something from nothing or modernize platforms
stuck in technical debt. From scaling the Cloud Innovation Center to 150+ people
to modernizing payments across 12 countries, his work comes with evidence:
real outcomes, not just strategy decks.
```

### MATT_DNA Ground Truth

`MATT_DNA` is generated at startup from the full story corpus, not extracted from a single story. See [Data Pipeline](12-data-pipeline#january-2026-sovereignty-patterns) for the correct mechanism.

### Synthesis Prompt Structure

**BASE_PROMPT:**
- Establishes Agy as fact-relayer (not evaluator)
- Sets voice guidelines (no meta-commentary)

**SYNTHESIS_DELTA:**
Added to the base prompt for synthesis questions. In summary:
- Show breadth across the stories provided, as natural prose with no section labels
- Follow WHY (30-40%), HOW (40-50%), WHAT (10-20%) internally
- Lead with the tension from the 2-3 most relevant stories; the rest are optional
- Bold client names and key numbers; state facts rather than evaluate Matt
- Cite only the clients shown in the stories (Ask Agy passes no client list, so the prompt's `{client_list}` falls back to that wording)

### Synthesis vs Narrative Queries

**Synthesis:** High-level patterns and themes
- "What's Matt's professional identity?"
- "Summarize Matt's approach to leadership"
- "What makes Matt different from other platform leaders?"

**Narrative:** Professional story arc (different intent family)
- "What's Matt's career journey?"
- "How has Matt's work evolved across his career?"
- "Tell me Matt's background"

Only the `synthesis` family switches to synthesis mode; narrative questions get the standard prompt.

---

## Query Validation & Nonsense Detection

**Purpose:** Prevent off-domain queries and ensure Agy only answers questions grounded in Matt's portfolio

### Architecture Overview

MattGPT uses a **two-layer defense** (Steps 1-2 of the answer pipeline):

**Layer 1: Pattern-Based Filters** (Step 1)
- Fast regex patterns for common off-domain categories
- Catches edge cases before semantic routing
- Zero-shot detection without embedding costs

**Layer 2: Semantic Router** (Step 2)
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
2. Query is embedded using OpenAI `{{ site.data.facts.embedding_model }}`
3. Cosine similarity computed against canonical intent examples
4. Query classified based on similarity threshold

**Thresholds (Updated January 2026):**

```python
HARD_ACCEPT = {{ site.data.facts.router_hard_accept }}  # Clearly on-topic, no question
SOFT_ACCEPT = {{ site.data.facts.router_soft_accept }}  # Accept but log as borderline for review (lowered from 0.72)
# The router rejects on its own only for out_of_scope and personal queries
# scoring at least HARD_ACCEPT (router_rejection_reason()). Ask Agy has no
# other rejection: every other query goes to the tool loop.
```

**What Changed (January 2026):**
- **Removed:** Entity Gate threshold bouncer (was blocking legitimate narrative queries)
- **Removed:** `classify_query_intent()` LLM call (GPT-4o-mini - redundant with router)
- **Added:** Synthesis detection (intent_family == "synthesis")
- **Added:** Out-of-scope industry detection (intent_family == "out_of_scope")
- **Added:** Narrative queries (intent_family == "narrative")
- **Added:** Personal query detection (intent_family == "personal")
- **Expanded:** 10 intent families → {{ site.data.facts.intent_family_count }} intent families

**Intent Families ({{ site.data.facts.intent_family_count }} categories):**

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
| **personal** | "How old is Matt?", "Is Matt single?" (salary questions are caught earlier, by the `personal_compensation` nonsense rule) |
| **out_of_scope** | Industry queries outside Matt's domain (graceful redirect) |

**Example Classification:**

```python
# HARD_ACCEPT (score 0.85, above 0.80 threshold)
"How did Matt scale agile at JP Morgan?" → intent_family: "delivery"

# SOFT_ACCEPT (score 0.74, above 0.40 threshold)
"What's Matt's approach to team growth?" → intent_family: "team_scaling"
# Logged for review, but accepted

# LOW-CONFIDENCE ON ROUTER (score 0.35, below 0.40 threshold)
"<a question unlike any intent example>" → intent_family: None
# Router flag is False and nothing rejects here; the query goes to the
# tool loop, and the model answers from what its searches return.
```

---

### RAG Pipeline Thresholds & Constants

All thresholds are centralized in `config/constants.py` (single source of truth):

**Semantic Router Thresholds:**
```python
HARD_ACCEPT = {{ site.data.facts.router_hard_accept }}   # Clearly on-topic, no question
SOFT_ACCEPT = {{ site.data.facts.router_soft_accept }}   # Accept but log as borderline for review
```

**RAG Confidence Thresholds:**
```python
CONFIDENCE_HIGH = {{ site.data.facts.confidence_high }}  # My Work's high-relevance tier
CONFIDENCE_LOW = {{ site.data.facts.confidence_low }}   # Hits below this are dropped (My Work and Ask Agy's search tool)
```

**Pinecone Search Parameters:**
```python
PINECONE_MIN_SIM = {{ site.data.facts.pinecone_min_sim }}   # Minimum similarity for Pinecone results
SEARCH_TOP_K = {{ site.data.facts.search_top_k }}         # Stories to fetch (headroom for reranking/filtering)
                          # Centralized Jan 2026 - was 100/7 conflict
```

**Model Configuration** (values in `config/constants.py`; the answer model is not yet a synced fact):
```python
ASK_ANSWER_MODEL              # Ask Agy's answer model, Responses API, streamed
ASK_ANSWER_REASONING_EFFORT   # Reasoning effort for the answer model
ASK_TOOL_MAX_ROUNDS           # Calls that may search before the answer call
ASK_HISTORY_MAX_MESSAGES      # Conversation messages sent with each question
DEFAULT_EMBEDDING_MODEL = "{{ site.data.facts.embedding_model }}"  # 1536 dimensions
```

---

### Pattern-Based Filters: Regex Nonsense Detection

**How It Works:**

Fast regex matching against `nonsense_filters.jsonl` patterns to catch obvious off-domain queries.

**Filter Categories:**

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
| **personal_compensation** | `\b(salary|compensation|income|bonus)\b` | "What's Matt's salary?" |
| **roleplay_impersonation** | `\b(roleplay|pretend|act)\b.*\b(as|to be)\b.*(her|him|my grandmother)` | "Pretend to be my grandmother" |
| **gibberish** | Two or more words each typed from one keyboard row | "asdf jkl qwerty" |
| **local_recommendations** | `\b(best|good|cheap)\s+(pizza|restaurants?|bars?|coffee|...)\b` | "Best pizza in Atlanta" |

The table shows a sample; `nonsense_filters.jsonl` has more categories. Five rules (`creative_writing` twice, `general_knowledge` twice, `gaming`) skip any question that names Matt or uses he, his or him, so "When was he at JP Morgan?" is not refused (ADR 043 in the app repo's `docs/ADR.md`).

**Performance:**
- Pattern matching is linear in the number of patterns and executes in under 5ms
- Runs before semantic routing, so obvious nonsense is caught at zero embedding cost

---

### UX Flow for Off-Domain Queries

**When a query is rejected:**

Ask Agy shows a one-line banner with suggestion chips in place of an answer. The copy and chip sets live in `BANNER_COPY` and the `*_CHIPS` lists in `utils/ui_helpers.py`; which one appears depends on what rejected the query:

| Rejected by | Banner | Chips |
|---|---|---|
| Nonsense rule (Step 1) | "🐾 Wrong trail. I'm a Plott Hound trained to track Matt's transformation work. Give me a real scent to follow." | `RULE_CHIPS` |
| Router, personal (Step 2) | "🐾 I'm focused on Matt's professional experience." | `PERSONAL_CHIPS` |
| Router, out of scope (Step 2) | "🐾 That's outside Matt's experience." | `OUT_OF_SCOPE_CHIPS` |

Ask Agy has no low-confidence banner; that copy (`BANNER_COPY["low_confidence"]`) is now shown only by My Work.

**When the answer call fails:** a rate limit or an embedding failure returns one of the "quick breather" messages written in `agy_answer()`; any other error returns `ANSWER_ERROR_MESSAGE` (`config/constants.py`) with no sources. A stream that breaks before any answer text is sent again once.

---

### How the Two-Stage Gate Works

Off-domain queries are handled in two stages. Stage 1, the pattern gate, runs `is_nonsense()`, fast regex matching against `nonsense_filters.jsonl`, before any embedding is generated; queries caught here are rejected at zero embedding cost. Stage 2, the semantic router, embeds passing queries and scores them against {{ site.data.facts.intent_family_count }} intent families: a score above {{ site.data.facts.router_hard_accept }} is a hard accept, {{ site.data.facts.router_soft_accept }} to {{ site.data.facts.router_hard_accept }} is a soft accept (logged for review), and below {{ site.data.facts.router_soft_accept }} produces a low-confidence router signal. The router rejects on its own only for out-of-scope and personal queries at or above {{ site.data.facts.router_hard_accept }}. Every other query goes to the tool loop; a low router score does not reject a query, and no confidence gate sits downstream.

---

## Data Governance & Architecture Principles

For data governance, sovereignty patterns, entity search, anti-patterns, and ingestion workflow, see [Data Pipeline](12-data-pipeline).

## Current State Summary

### What's Live Today (October 2026)

- Production Streamlit application at [askmattgpt.streamlit.app](https://askmattgpt.streamlit.app)
- **Ask Agy answer pipeline**: nonsense rules and router redirects, then a tool loop in which the model searches the stories and streams its answer (`ASK_ANSWER_MODEL` in `config/constants.py`)
- **Golden eval** of {{ site.data.facts.eval_query_count }} queries, run by hand; no run yet on the October 2026 pipeline
- **Semantic router** with {{ site.data.facts.intent_family_count }} intent families + out-of-scope/personal detection
- **Role Match**: JD-to-experience matching (Phases 1-3: recruiter view, evidence chips, action buttons; Phase 4 slice 1: lock icon + password gate for private view)
- **Query analytics**: event logger to Google Sheets (assessment, chip click, action button, UTM attribution)
- **Triage agent surface**: `scripts/assess_jd.py` CLI wraps `jd_assessor.py` for external agent orchestration (engine-as-adapter pattern)
- 100+ STAR-structured project stories
- Semantic search: Ask Agy's search tool, and My Work's filters with confidence banners
- **Timeline View** with Era-based career progression
- **Mobile-responsive design** (global_styles.py + mobile_overrides.py; breakpoints: 767px, 1024px)
- **Dark mode support** via CSS variables
- **Modular architecture** ({{ site.data.facts.ask_mattgpt_module_file_count }}-file ask_mattgpt/ structure)
- **BDD test suite**: {{ site.data.facts.bdd_summary }}
- Conversation history: the last `ASK_HISTORY_MAX_MESSAGES` messages go to the answer model, which writes its search queries with them in view
- Related Projects UX pattern



---

**Related Documentation:**
- [Product Vision](01-product-vision) - Strategic positioning
- [UX Design Process](03-ux-design-process) - Design decisions
- [Building MattGPT](04-building-mattgpt) - Development journey
- [RAG Quality Evaluation](11-testing-and-quality) - Eval framework ({{ site.data.facts.eval_query_count }} golden queries)

---

*Last updated: {{ site.data.page_dates['02-technical-architecture'] }}*
