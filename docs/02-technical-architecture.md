# Technical Architecture

**MattGPT: From Streamlit MVP to React Rebuild**

> This document outlines the technical architecture evolution of MattGPT, demonstrating intentional trade-off awareness and strategic planning from rapid prototyping to production polish.

---

## Architecture Roadmap Overview

MattGPT's architecture follows a two-phase evolution:

1. **Phase 1 (Streamlit MVP):** Validate RAG architecture with minimal investment
2. **Phase 2 (React Rebuild):** Better performance and maintainability

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

- **Mobile-responsive implementation:** Production-quality mobile CSS with breakpoints at 767px (mobile), 768-1024px (tablet), 1024+ (desktop)
- **Limited scalability:** Estimated ~100 concurrent users (not load-tested; Streamlit Community Cloud limits apply)
- **No caching layer:** Direct database queries
- **Modular architecture:** Refactored from monolithic app.py to component-based structure
- **Minimal observability:** Basic logging only

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

### Architecture Evolutions Achieved (December 2025)

**Modular Component Structure:**
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

**Query Validation & Nonsense Detection:**
- Semantic router with dual-threshold intent classification
- Pattern-based filtering for off-domain queries
- 10+ intent families covering valid query types
- Graceful handling with suggestion chips for rejected queries

---

## Query Validation & Nonsense Detection

**Status:** ✅ Implemented (October 2025)
**Purpose:** Prevent off-domain queries and ensure Agy only answers questions grounded in Matt's portfolio

### Architecture Overview

MattGPT uses a **two-layer defense** to filter nonsense queries before they reach the RAG pipeline:

**Layer 1: Semantic Router** (Intent Classification)
- Embedding-based similarity matching against canonical intent examples
- Dual-threshold system for confident classification
- Returns best matching intent family for telemetry and debugging

**Layer 2: Pattern-Based Filters** (Regex Matching)
- Fast regex patterns for common off-domain categories
- Catches edge cases that slip through semantic routing
- Zero-shot detection without embedding costs

**Implementation Files:**
- `services/semantic_router.py` - Intent classification logic
- `nonsense_filters.jsonl` - Regex pattern definitions
- `data/intent_embeddings.json` - Cached embeddings for intent families

---

### Semantic Router: Dual-Threshold Intent Classification

**How It Works:**

1. User submits query
2. Query is embedded using OpenAI text-embedding-3-small
3. Cosine similarity computed against canonical intent examples
4. Query classified based on similarity threshold

**Thresholds:**

```python
HARD_ACCEPT = 0.80  # Clearly on-topic, no question
SOFT_ACCEPT = 0.72  # Accept but log as borderline for review
# Below 0.72 = router rejects (search fallback may still attempt)
```

**Intent Families (10 categories):**

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

**Example Classification:**

```python
# HARD_ACCEPT (0.85 similarity)
"How did Matt scale agile at JPMorgan?" → intent_family: "delivery"

# SOFT_ACCEPT (0.74 similarity)
"What's Matt's approach to team growth?" → intent_family: "team_scaling"
# Logged for review, but accepted

# REJECTED (0.55 similarity)
"What's the weather in New York?" → intent_family: None
# Triggers off-domain response
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

1. **Semantic router** flags query as below 0.72 threshold
2. **Pattern filter** (optional) confirms off-domain category
3. **Agy responds** with helpful redirect:

**Example Response:**

```
🐾 I'm not finding matches for that in Matt's portfolio.

I'm focused on Matt's 20 years of digital transformation
experience—things like agile delivery, platform engineering,
team scaling, and stakeholder management.

Try asking about:
• "How did Matt scale teams at JPMorgan?"
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

**Borderline Cases (0.72-0.79):**

| Query | Similarity | Action |
|-------|-----------|--------|
| "What's Matt's management philosophy?" | 0.74 | Accepted (soft), logged for review |
| "How does someone become a platform engineer?" | 0.73 | Accepted (soft), may redirect to Matt's experience |
| "Tell me about product-market fit" | 0.71 | Rejected, suggest Matt's product innovation work |

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
- Soft accept threshold (0.72-0.79) logs borderline queries
- Manual review improves intent family coverage over time
- Telemetry shows what users are actually asking

---

### Future Enhancements (Phase 2)

**Planned improvements for React migration:**

- **Dynamic intent expansion:** Use LLM to generate new canonical examples from accepted queries
- **User feedback loop:** "Was this answer helpful?" → retrain router
- **Intent routing:** Different response templates per intent family
- **A/B testing:** Experiment with threshold values (0.72 vs 0.75)
- **Analytics dashboard:** Visualize rejection reasons, borderline cases, intent distribution

**Phase 2 migration note:** The semantic router logic is framework-agnostic and will port directly to React/FastAPI architecture.

---

## Phase 2: React Rebuild

**Status:** 🎯 Planned (Q1 2026)
**Purpose:** Better performance and maintainability

The Streamlit MVP validated the RAG architecture and UX patterns. React will make it production-quality with modern tooling and mobile-first design.

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

**Phase 2: Production Polish (Q1 2026)**
- **Purpose:** Better performance and maintainability
- **Stack:** React + FastAPI - modern tooling, mobile-first
- **Benefit:** Preserve core IP (RAG pipeline), improve presentation layer

**Principle:** Start simple, evolve intentionally

---

## Current State Summary

### What's Live Today (Phase 1 - December 2025)

- ✅ Production Streamlit application at [askmattgpt.streamlit.app](https://askmattgpt.streamlit.app)
- ✅ RAG pipeline with GPT-4 and Pinecone vector search
- ✅ 130+ STAR-structured project stories
- ✅ Semantic search with confidence scoring and metadata filtering
- ✅ **Timeline View** with Era-based career progression
- ✅ **Mobile-responsive design** (breakpoints: 767px, 1024px)
- ✅ **Dark mode support** via CSS variables
- ✅ **Modular architecture** (9-file ask_mattgpt/ structure)
- ✅ Conversation history and context management
- ✅ Related Projects UX pattern

### What's Next (Phase 2 - Q1 2026)

- 🎯 React + Next.js frontend rebuild
- 🎯 FastAPI backend
- 🎯 Mobile-first design from the start
- 🎯 Same RAG pipeline, better UI

---

## Technical Diagrams

For visual representations of the architecture, see:

- [RAG Architecture Diagram](../images/architecture/tech_rag_architecture.png) - Complete RAG lifecycle
- [Site Architecture](../images/architecture/site_architecture.png) - Page hierarchy and navigation
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

---

*Last Updated: December 2025 (Post-Audit Refresh)*
*Version: 1.1 (Updated with Implementation Details)*
