# Building MattGPT

**From Problem to Production: The Story of Building an AI-Powered Portfolio**

> This document chronicles the journey of building MattGPT, from identifying the core problem through technical implementation, key learnings, and the product roadmap forward.

---

## Table of Contents

1. [The Problem](#the-problem)
2. [Why I Built This](#why-i-built-this)
3. [Technical Implementation](#technical-implementation)
4. [What MattGPT Can Do](#what-mattgpt-can-do)
5. [Key Challenges & Solutions](#key-challenges--solutions)
6. [Lessons Learned](#lessons-learned)

---

## The Problem

Traditional portfolios are static PDFs that don't scale. Recruiters and hiring managers can't easily search {{ site.data.facts.story_count_label }} projects by methodology, outcome, or domain.

**The Core Issues:**

**Generic, Unverifiable Claims**
- "Experienced in agile transformation"
- "Skilled in platform engineering"
- "Led large-scale projects"

These statements provide no proof, no context, and no measurable outcomes.

**Inefficient Vetting Process**
- Recruiters spend hours manually scanning resumes
- Hiring managers can't quickly validate specific experience
- Candidates struggle to retrieve relevant stories for interviews

    **Lost Opportunities**
- Deep expertise gets overlooked due to poor discoverability
- Pattern recognition across projects is impossible
- No way to demonstrate methodology consistency

**The Mission:**

I wanted to create an intelligent, conversational interface that:
- Understands intent, not just keywords
- Surfaces verifiable proof from structured data
- Enables rapid filtering and comparison
- Provides auditable source references

This isn't just a portfolio showcase; it's a **functional AI application demonstrating product thinking, modern engineering practices, and hands-on implementation.**

---

## Why I Built This

In September 2023, I left Accenture. I'd built their Cloud Innovation Center from scratch, scaled it to 150+ professionals, and helped Fortune 500 companies modernize platforms worth billions. But I was burned out, underwent a few surgeries, and I needed time to figure out what came next.

Six months into my sabbatical, I started updating my resume. That's when it hit me: I had more transformation stories than I could tell in an interview, and I was trying to squeeze them into two pages of bullet points. The format was fundamentally broken.

Recruiters would spend 30 seconds scanning my PDF. How could they possibly understand the nuance of scaling engineering teams from 4 to 150? Or the pattern recognition I'd developed across 55 banking projects? They couldn't. Nobody could.

I'd been experimenting with RAG architectures and vector search for a client POC, and the idea clicked: What if I could build an AI that actually understood my work, not through keyword matching, but through semantic understanding of outcomes, methodologies, and context?

I started with a two-week MVP sprint in April 2025. That initial prototype validated the concept, but the real work ({{ site.data.facts.story_count_label }} stories, semantic search tuning, mobile design, the strangler fig refactor) took another eight months.

I named it after Agy, my late Plott Hound (short for Agador Spartacus from "The Birdcage"). He was a tracker: goofy yet patient, determined, excellent at finding exactly what you were looking for. That's what I wanted this AI to be.

---

## Technical Implementation

### Tech Stack

**Core Technologies:**

| Technology | Purpose | Why This Choice |
|------------|---------|-----------------|
| **Python** | Primary language (code targets 3.11; CI and local dev run 3.13) | Rich ML/AI ecosystem, rapid prototyping |
| **Streamlit** | Frontend framework (MVP) | 2-week build time vs 3+ months for React |
| **OpenAI (`ASK_ANSWER_MODEL` in `config/constants.py`)** | LLM for Ask Agy's answers, on the Responses API, streamed | State-of-the-art reasoning and synthesis |
| **Pinecone** | Vector database | Managed solution, fast ANN search |
| **OpenAI Embeddings** | Embedding generation | text-embedding-3-small (1536-dim) |
| **Pandas** | Data manipulation | JSONL processing and metadata handling |
| **Python venv** | Dependency management | Local dev environment with requirements.txt |
| **Streamlit Cloud** | Deployment (MVP) | Free hosting for Streamlit apps |

### System Architecture

**Data Flow:**

```
1. INGESTION (Build Time)
   ┌─────────────────────┐
   │   Source Data       │
   │   (STAR/5P JSONL)  │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐
   │  Data Governance    │
   │  Validation Layer   │
   │  (Mandatory STAR)   │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐
   │  Embedding Model    │
   │  (OpenAI)           │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐
   │   Pinecone Vector   │
   │   Database Index    │
   └─────────────────────┘

2. RETRIEVAL (Run Time)
   ┌─────────────────────┐
   │    User Query       │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐
   │  Nonsense Rules +   │
   │  Semantic Router    │
   │  (redirects)        │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐        ┌─────────────────────┐
   │  Tool Loop          │ ─────▶ │  Semantic Search    │
   │  (answer model      │ ◀───── │ (Vector Similarity  │
   │  writes searches)   │        │ + Keyword Blend)    │
   └──────────┬──────────┘        └─────────────────────┘
              │
              ▼
   ┌─────────────────────┐
   │  Streamed           │
   │  Response +         │
   │  Source Citations   │
   └─────────────────────┘
```

### Semantic Search with Confidence Scoring

One of the core technical innovations in MattGPT is the **semantic search** approach that combines vector-based meaning matching with confidence-based filtering.

**The Algorithm (simplified sketch):** the real `semantic_search()` also applies metadata filters (My Work) and blends in keyword overlap; Ask Agy's answer model calls it through a search tool. See [API Reference](09-api-reference).

```python
def semantic_search(query: str, top_k: int = SEARCH_TOP_K) -> Dict:
    """
    Pure semantic retrieval with confidence-based filtering
    to ensure high-quality, relevant results.

    Args:
        query: User's natural language question
        top_k: Number of candidate results to retrieve

    Returns:
        Dict with results, confidence level, and top score
    """
    # Vector similarity search via Pinecone
    hits = pinecone_query(
        embed(query),
        top_k=top_k
    )

    # Calculate confidence from top score
    top_score = max(h.get("pc_score", 0.0) for h in hits)

    # Three-tier confidence system
    if top_score >= 0.25:
        confidence = "high"
    elif top_score >= 0.20:
        confidence = "low"
    else:
        confidence = "none"

    return {
        "results": hits,
        "confidence": confidence,
        "top_score": top_score
    }
```

**Why This Matters:**

- **Semantic Understanding:** Recognizes that "bootstrap it" and "start a new project" are conceptually similar
- **Confidence Scoring:** Three tiers (high/low/none); hits below low never reach Ask Agy's answer model, and My Work shows the tier as a relevance note
- **Threshold Gating:** Prevents low-quality matches from appearing (minimum {{ site.data.facts.pinecone_min_sim }} similarity)
- **Intent Routing:** Classifies the question's topic into one of {{ site.data.facts.intent_family_count }} intent families; mapping a visitor's goal (interview prep, due diligence, pitch) to tone is not implemented (see [Agy Voice Guide](05-agy-voice-guide#user-intent-recognition))

Answers are checked end to end by the {{ site.data.facts.eval_query_count }}-query golden eval described in [Testing and Quality](11-testing-and-quality); it has not yet run on the October 2026 pipeline.

### Data Governance: The Two-Layer Model

**Layer 1: Integrity (Mandatory STAR Method)**

Every project must include:
- **Situation:** Business context and challenge
- **Task:** Specific objective or problem
- **Action:** Methodology, decisions, execution
- **Result:** Measurable outcomes with metrics

Every story is authored with all four STAR fields and at least one metric.

**Layer 2: Intelligence (Tagging Systems)**

**5P Taxonomy:**
- **Person:** Role, seniority, team structure
- **Place:** Client, industry, geographic context
- **Purpose:** Capability area, transformation type

**Public Tags and Competencies:**
- **Public tags:** curated in the Excel source, which is authoritative (see [Data Pipeline](12-data-pipeline))
- **Competencies:** `generate_competencies.py` has an LLM suggest competencies per story, exported to a spreadsheet for review. Its prompt asks for SFIA, O*NET and LinkedIn skills terminology; that is a prompt instruction, not a mapping to those frameworks.

This dual-layer approach ensures both **data integrity** (every answer is auditable) and **search intelligence** (pattern recognition and cross-project synthesis).

---

## What MattGPT Can Do

### LLM-Powered Capabilities

**RAG (Retrieval-Augmented Generation):**
- Uses structured STAR stories and semantic search
- Answers from the stories its searches return, Matt's profile and the conversation, with the retrieved stories shown as source cards
- The prompt instructs Agy not to invent examples beyond Matt's stories, and to decline in its own voice, naming the topic, when his profile has nothing on it; code picks the decline's opening line, but no code guarantees either rule is followed

**Automated Tag Generation:**
- An LLM suggests competencies from each story's content, for human review
- The prompt asks for SFIA, O*NET and LinkedIn skills terminology; there is no ontology lookup
- A story's Competencies field is part of the text that gets embedded (`build_custom_embeddings.py`)

**Embedded Classification:**
- Discovers similar or complementary projects
- Powers "Related Projects" recommendations
- Identifies methodology patterns across contexts

**Natural Language Queries:**
- Ask in plain English: no keyword guessing required
- The semantic router classifies the question's topic; it does not detect the visitor's goal (vetting, assessment, pitch)
- Conversational follow-up with context retention

---

### Semantic Search Strategy

**Vector-Based Meaning Matching:**
- Understanding intent, not just keywords
- Example: "bootstrap it" vs "start a new project" are semantically similar
- Handles synonyms, related concepts, and contextual understanding

**Confidence-Based Filtering:**
- Three-tier system: high, low, none
- Hits below low are dropped before Ask Agy's answer model sees them
- My Work shows the tier as a relevance note

**Metadata Enhancement:**
- Client, industry, domain, and role filtering
- Era, capability, tag and has-metric filtering
- Supports precise targeting in My Work; Ask Agy's searches pass no filters

**Ranking:**
- Pinecone similarity blended with a keyword overlap score (`W_KW` in `config/constants.py`)
- The answer model chooses which of the retrieved stories to use
- No recency or credibility boost, and no ranking trained on feedback

---

### Frontend (Streamlit MVP)

**Conversational UI:**
- Context-aware chat history persistence
- Multi-turn conversations with question refinement
- Suggestion chips for common query patterns

**Multi-View Presentation:**
- **Table View:** High data density, sortable columns (recruiter preference)
- **Card View:** Visual browsing with summary previews (hiring manager preference)
- **Timeline View:** Chronological career progression (content user preference)

**Responsive Design:**
- Desktop-first UI optimized for professional vetting
- Mobile-responsive for on-the-go access
- Progressive enhancement approach

**Real-Time Filtering:**
- Filter by Industry, Client, Domain, Role
- Instant results with transparency banners
- Faceted search with result counts

**Export & Share:**
- Unique project-level URLs for sharing specific stories
- Copy-to-clipboard for quick reference

---

### Backend & DevOps

**Ask Agy Answer Pipeline (October 2026, MATTGPT-275):**
- **Step 1:** Rules-based nonsense detection (regex patterns)
- **Step 2:** Semantic router with {{ site.data.facts.intent_family_count }} intent families (out-of-scope/personal redirect at hard accept; the synthesis family switches the prompt)
- **Step 3:** Tool loop: the answer model writes its own `search_stories` queries, then answers
- **Step 4:** Each search: Pinecone similarity blended with keyword overlap, hits below low dropped
- **Step 5:** The answer streams, then is replaced by the post-processed text
- **Quality:** a {{ site.data.facts.eval_query_count }}-query golden eval, run by hand; no run yet on this pipeline
- Replaced the January 2026 5-stage pipeline (entity detection, confidence gating, entity pinning); see [Technical Architecture](02-technical-architecture)

**Conversational Workflow:**
- Streamlit's stateful UI for session management
- Context retention across multi-turn conversations
- The answer model sees the recent conversation when it writes its searches
- Semantic router switches the synthesis prompt and redirects out-of-scope and personal queries

**Semantic Embeddings:**
- OpenAI text-embedding-3-small
- 1536-dimensional vector space

**Metadata Filtering:**
- Pinecone metadata filters for faceted search
- Tag-based discovery (5P + semantic tags)
- Client/industry/domain hierarchical filtering

**Modular Architecture:**
- Separation of concerns: data layer, search layer, presentation layer
- Easy to swap Streamlit for React in Phase 2
- Core RAG pipeline remains unchanged across UI iterations

**Logging & Observability:**
- Every query is logged to a Google Sheet by `log_query()` in `services/query_logger.py`: intent family, top score, result count and redirect reason, plus the confidence tier for My Work searches
- Visitor answer ratings are logged alongside
- No latency, error-rate or cache metrics are collected

---

## Key Challenges & Solutions

### The Semantic Search Tuning Problem

Early semantic search was too fuzzy. Ask about "JP Morgan," and it might return stories about "banking transformation" that never mention the client by name. The challenge was balancing broad conceptual understanding with precision.

The solution was confidence-based filtering with metadata enhancement. By implementing a three-tier confidence system (high ≥ {{ site.data.facts.confidence_high }}, low ≥ {{ site.data.facts.confidence_low }}, none < {{ site.data.facts.confidence_low }}) and combining vector similarity with client/industry/domain filters, the system now delivers both relevant and precise results. Getting the threshold values right took three weeks of tuning against real queries.

### The Hallucination Problem

Early versions of the system would confidently generate plausible-sounding details that never happened. "Matt led a team of 200 engineers," when the actual number was 150. "The project saved $50M" when I'd never quantified the savings.

The fix was two-fold: mandatory STAR structure (every story needs verifiable Situation, Task, Action, Result) and aggressive fact-checking. I went through all {{ site.data.facts.story_count_label }} stories line by line, deleting anything I couldn't defend in an interview. If I couldn't cite a specific metric or name a specific outcome, it got cut.

### The Streamlit CSS Problem

Streamlit is great for rapid prototyping. It's terrible for custom styling. Class names change between versions. CSS selectors that work on desktop break on mobile. The st-emotion-cache-* classes are generated dynamically and can't be targeted reliably.

I learned to use data-testid attributes and container keys for CSS targeting. Most application CSS lives in global_styles.py and mobile_overrides.py: carefully scoped overrides for Streamlit's generated class names. Components and pages also carry their own scoped style blocks. It's not pretty, but it works.

### The January 2026 RAG Pipeline Cleanup

Early versions had redundant gates that were blocking legitimate queries. An "Entity Gate" threshold bouncer would reject narrative queries like "What's Matt's professional identity?" because they didn't mention a specific client or employer. A separate LLM intent classification call (GPT-4o-mini) was duplicating work the semantic router already did.

The cleanup removed both:
- **Entity Gate removed:** Was rejecting valid narrative/synthesis queries
- **LLM intent classification removed:** Redundant with semantic router (which uses embeddings, not LLM calls)

The semantic router now handles everything: {{ site.data.facts.intent_family_count }} intent families (including narrative, synthesis, out-of-scope, personal), dual-threshold classification ({{ site.data.facts.router_hard_accept }} hard accept / {{ site.data.facts.router_soft_accept }} soft accept), and graceful rejection with suggestion chips. At that stage (January 2026), eval quality improved to 98.4% (60/61) while reducing costs and latency; the suite has since grown to {{ site.data.facts.eval_query_count }} queries.

---

## Lessons Learned

### MVP-First Thinking Pays Off

I originally planned to build a React + FastAPI stack. I'm glad I didn't. Streamlit let me validate the core RAG architecture in two weeks instead of three months. The UI isn't perfect, but the search quality and system prompt design are solid; those are the hard parts.

### Data Quality > Fancy Algorithms

I spent more time curating STAR stories and validating metadata than I did on the ML pipeline. That discipline paid off: every AI response traces to auditable source data, and Agy is instructed not to invent examples beyond Matt's stories.

### Refactoring the Monolith

By October 2025, app.py had grown to 5,765 lines and ask_mattgpt.py to 4,696, both unmaintainable. I broke them apart incrementally, extracting one component at a time rather than rewriting, until app.py was 284 lines and ask_mattgpt.py a modular directory, small enough to hand individual pieces to AI coding assistants without them needing the whole system.

### The "Can You Defend It?" Standard

Every claim in this portfolio passes one test: Can I defend it in an interview? If someone asks, "How do you know it was 4x faster?" I have the answer. If someone asks, "What exactly did you do versus the team?" I can explain my specific contribution. This standard eliminated much of the impressive-sounding but unverifiable content.

---

## Appendix: Build Timeline

### April 2025: Foundation
- Curated the first 50 STAR stories from Accenture tenure
- Set up Pinecone vector database
- Implemented basic Streamlit UI with semantic search
- Validated RAG pipeline with OpenAI GPT-4

### May-June 2025: Expansion
- Grew corpus to {{ site.data.facts.story_count_label }} stories covering full career
- Added filters (client, role, domain, industry)
- Designed three view modes: Table, Card, Timeline
- Built "How Agy Searches" transparency modal

### July-August 2025: Polish
- User testing with recruiters and hiring managers
- Refined system prompt based on feedback
- Implemented semantic search with confidence scoring
- Added three-tier confidence system and source citations

### September 2025: Fact-Check Sprint
- Line-by-line audit of all stories
- Removed AI-generated fabrications
- Ensured 100% defensibility standard
- Finalized {{ site.data.facts.story_count_label }} verified stories

### October 2025: Architecture Refactor
- Refactored monolith: app.py 5,765 → 284 lines, ask_mattgpt.py 4,696 → modular directory
- Created comprehensive design specification
- Deployed to Streamlit Cloud

### November-December 2025: Mobile & Polish
- Complete mobile responsive implementation
- Era-based Timeline view (5 career phases)
- Filter redesign (Primary + Advanced)
- Related Projects UX improvements
- Design spec sync and documentation refresh

---

## Conclusion

Building MattGPT has been equal parts technical challenge and product discovery. What started as a personal problem ("How do I make a full career of experience searchable?") became a demonstration of modern AI product development: RAG architecture, semantic search, data governance, and user-centered design.

**The core thesis remains:**

> **Replace "I'm experienced" with "Here's exactly what I did, how I did it, and the measurable results."**

Every line of code, every STAR story, every design decision serves that mission: **proof over claims.**

---

**Related Documentation:**
- [Product Vision](01-product-vision) - Strategic positioning and user personas
- [Technical Architecture](02-technical-architecture) - RAG pipeline and system design
- [Audience Journeys](03-ux-design-process) - The four visitor journeys the design is derived from
- [RAG Quality Evaluation](11-testing-and-quality) - How evals validate quality ({{ site.data.facts.eval_query_count }} golden queries)

---

*Last updated: {{ site.data.page_dates['04-building-mattgpt'] }}*
