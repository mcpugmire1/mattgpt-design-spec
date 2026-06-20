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
7. [Product Roadmap](#product-roadmap)
8. [Future Vision](#future-vision)

---

## The Problem

Traditional portfolios are static PDFs that don't scale. Recruiters and hiring managers can't easily search {{ site.data.facts.story_count_label }} projects by methodology, outcome, or domain.

**The Core Issues:**

❌ **Generic, Unverifiable Claims**
- "Experienced in agile transformation"
- "Skilled in platform engineering"
- "Led large-scale projects"

These statements provide no proof, no context, and no measurable outcomes.

❌ **Inefficient Vetting Process**
- Recruiters spend hours manually scanning resumes
- Hiring managers can't quickly validate specific experience
- Candidates struggle to retrieve relevant stories for interviews

❌ **Lost Opportunities**
- Deep expertise gets overlooked due to poor discoverability
- Pattern recognition across projects is impossible
- No way to demonstrate methodology consistency

**The Mission:**

I wanted to create an intelligent, conversational interface that:
- ✅ Understands intent, not just keywords
- ✅ Surfaces verifiable proof from structured data
- ✅ Enables rapid filtering and comparison
- ✅ Provides auditable source references

This isn't just a portfolio showcase — it's a **functional AI application demonstrating product thinking, modern engineering practices, and hands-on implementation.**

---

## Why I Built This

In September 2023, I left Accenture. I'd built their Cloud Innovation Center from scratch, scaled it to 150+ professionals, and helped Fortune 500 companies modernize platforms worth billions. But I was burned out, underwent a few surgeries, and I needed time to figure out what came next.

Six months into my sabbatical, I started updating my resume. That's when it hit me: I had more transformation stories than I could tell in an interview, and I was trying to squeeze them into two pages of bullet points. The format was fundamentally broken.

Recruiters would spend 30 seconds scanning my PDF. How could they possibly understand the nuance of scaling engineering teams from 4 to 150? Or the pattern recognition I'd developed across 55 banking projects? They couldn't. Nobody could.

I'd been experimenting with RAG architectures and vector search for a client POC, and the idea clicked: What if I could build an AI that actually understood my work — not through keyword matching, but through semantic understanding of outcomes, methodologies, and context?

I started with a two-week MVP sprint in April 2025. That initial prototype validated the concept, but the real work — {{ site.data.facts.story_count_label }} stories, semantic search tuning, mobile design, the strangler fig refactor — took another eight months.

I named it after Agy, my late Plott Hound — short for Agador Spartacus from "The Birdcage." He was a tracker — goofy yet patient, determined, excellent at finding exactly what you were looking for. That's what I wanted this AI to be.

---

## Technical Implementation

### Tech Stack

**Core Technologies:**

| Technology | Purpose | Why This Choice |
|------------|---------|-----------------|
| 🐍 **Python 3.11** | Primary language | Rich ML/AI ecosystem, rapid prototyping |
| **Streamlit** | Frontend framework (MVP) | 2-week build time vs 3+ months for React |
| 🤖 **OpenAI GPT-4o** | LLM for response generation | State-of-the-art reasoning and synthesis |
| 🌲 **Pinecone** | Vector database | Managed solution, fast ANN search |
| **OpenAI Embeddings** | Embedding generation | text-embedding-3-small (1536-dim) |
| **Pandas** | Data manipulation | JSONL processing and metadata handling |
| **NumPy** | Numerical operations | Vector math and similarity scoring |
| 📦 **Python venv** | Dependency management | Local dev environment with requirements.txt |
| ☁️ **Streamlit Cloud** | Deployment (MVP) | Free hosting for Streamlit apps |

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
   │  Query Embedding    │
   │  + Intent Analysis  │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐
   │  Semantic Search    │
   │ (Vector Similarity  │
   │ + Confidence Score) │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐
   │  RAG Orchestrator   │
   │  (GPT-4o + Context) │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐
   │  Synthesized        │
   │  Response +         │
   │  Source Citations   │
   └─────────────────────┘
```

### The Secret Sauce: Semantic Search with Confidence Scoring

One of the core technical innovations in MattGPT is the **semantic search** approach that combines vector-based meaning matching with confidence-based filtering.

**The Algorithm:**

```python
def semantic_search(query: str, top_k: int = 10) -> Dict:
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
- **Confidence Scoring:** Three-tier system (high/low/none) ensures quality results
- **Threshold Gating:** Prevents low-quality matches from appearing (minimum 0.20 similarity)
- **Intent Recognition:** Understands user goals (interview prep, due diligence, pitch)

This approach was validated through manual testing of common query patterns, demonstrating effective semantic understanding and confidence-based filtering.

### Data Governance: The Two-Layer Model

**Layer 1: Integrity (Mandatory STAR Method)**

Every project must include:
- **Situation:** Business context and challenge
- **Task:** Specific objective or problem
- **Action:** Methodology, decisions, execution
- **Result:** Measurable outcomes with metrics

**Enforcement:** No project can be indexed without completing all four STAR fields + at least one quantifiable metric.

**Layer 2: Intelligence (Tagging Systems)**

**5P Taxonomy (Private Metadata):**
- **Person:** Role, seniority, team structure
- **Place:** Client, industry, geographic context
- **Purpose:** Capability area, transformation type

**Semantic Public Tags (Industry Standards):**
- **O*NET Competencies:** Standardized skill taxonomy
- **SFIA Framework:** IT professional skills
- **LinkedIn Skills:** Common search terms

This dual-layer approach ensures both **data integrity** (every answer is auditable) and **search intelligence** (pattern recognition and cross-project synthesis).

---

## What MattGPT Can Do

### 🧠 LLM-Powered Capabilities

**RAG (Retrieval-Augmented Generation):**
- Uses structured STAR stories and semantic search
- Grounds all responses in verifiable source data
- Prevents hallucination by constraining to known projects

**Automated Tag Generation:**
- Extracts keywords using NLP and ontology mapping
- Aligns skills to industry standards (O*NET, SFIA)
- Enables discovery through multiple taxonomies

**Embedded Classification:**
- Discovers similar or complementary projects
- Powers "Related Projects" recommendations
- Identifies methodology patterns across contexts

**Natural Language Queries:**
- Ask in plain English — no keyword guessing required
- Intent recognition understands question types (vetting, assessment, pitch)
- Conversational follow-up with context retention

---

### 🔍 Semantic Search Strategy

**Vector-Based Meaning Matching:**
- Understanding intent, not just keywords
- Example: "bootstrap it" vs "start a new project" are semantically similar
- Handles synonyms, related concepts, and contextual understanding

**Confidence-Based Filtering:**
- Three-tier system: high (≥0.25), low (≥0.20), none (<0.20)
- Prevents low-quality matches from appearing
- Ensures results are relevant and trustworthy

**Metadata Enhancement:**
- Client, industry, domain, and role filtering
- Date range and outcome type filtering
- Supports precise targeting for specific queries

**Re-ranking Algorithm:**
- Boosts results by relevance + recency + credibility
- Considers user intent (recruiter vetting vs deep assessment)
- Fine-tuned using human-in-the-loop feedback

---

### 📊 Frontend (Streamlit MVP)

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
- PDF download via print dialog (MVP gated export)
- Unique project-level URLs for sharing specific stories
- Copy-to-clipboard for quick reference

---

### 🔧 Backend & DevOps

**5-Stage RAG Pipeline (January 2026):**
- **Stage 1:** Rules-based nonsense detection (regex patterns)
- **Stage 2:** Semantic router with 15 intent families (intent classification + out-of-scope/personal detection)
- **Stage 3:** Confidence gating on Pinecone results (HIGH ≥0.25, LOW ≥0.20)
- **Stage 4:** Entity detection & story pinning (Client, Employer, Division, Title)
- **Stage 5:** Intent-aware ranking with context isolation (narrative vs entity queries)
- **Quality:** 100% eval pass rate (64/64 queries)

**Conversational Workflow:**
- Streamlit's stateful UI for session management
- Context retention across multi-turn conversations
- Semantic router handles synthesis, narrative, and out-of-scope queries

**Semantic Embeddings:**
- OpenAI text-embedding-3-small
- 1536-dimensional vector space
- ~1.2s average embedding + retrieval time

**Metadata Filtering:**
- Pinecone metadata filters for faceted search
- Tag-based discovery (5P + semantic tags)
- Client/industry/domain hierarchical filtering

**Modular Architecture:**
- Separation of concerns: data layer, search layer, presentation layer
- Easy to swap Streamlit for React in Phase 2
- Core RAG pipeline remains unchanged across UI iterations

**Logging & Observability:**
- Query analysis and performance tracking
- Response quality monitoring
- System health metrics (latency, error rate, cache hit ratio)

---

## Key Challenges & Solutions

### The Semantic Search Tuning Problem

Early semantic search was too fuzzy. Ask about "JP Morgan," and it might return stories about "banking transformation" that never mention the client by name. The challenge was balancing broad conceptual understanding with precision.

The solution was confidence-based filtering with metadata enhancement. By implementing a three-tier confidence system (high ≥ 0.25, low ≥ 0.20, none < 0.20) and combining vector similarity with client/industry/domain filters, the system now delivers both relevant and precise results. Getting the threshold values right took three weeks of tuning against real queries.

### The Hallucination Problem

Early versions of the system would confidently generate plausible-sounding details that never happened. "Matt led a team of 200 engineers," when the actual number was 150. "The project saved $50M" when I'd never quantified the savings.

The fix was two-fold: mandatory STAR structure (every story needs verifiable Situation, Task, Action, Result) and aggressive fact-checking. I went through all {{ site.data.facts.story_count_label }} stories line by line, deleting anything I couldn't defend in an interview. If I couldn't cite a specific metric or name a specific outcome, it got cut.

### The Streamlit CSS Problem

Streamlit is great for rapid prototyping. It's terrible for custom styling. Class names change between versions. CSS selectors that work on desktop break on mobile. The st-emotion-cache-* classes are generated dynamically and can't be targeted reliably.

I learned to use data-testid attributes and container keys for CSS targeting. The global_styles.py file grew to {{ site.data.facts.global_styles_line_count }} lines of carefully scoped overrides. It's not pretty, but it works.

### The January 2026 RAG Pipeline Cleanup

Early versions had redundant gates that were blocking legitimate queries. An "Entity Gate" threshold bouncer would reject narrative queries like "What's Matt's professional identity?" because they didn't mention a specific client or employer. A separate LLM intent classification call (GPT-4o-mini) was duplicating work the semantic router already did.

The cleanup removed both:
- **Entity Gate removed:** Was rejecting valid narrative/synthesis queries
- **LLM intent classification removed:** Redundant with semantic router (which uses embeddings, not LLM calls)

The semantic router now handles everything: 15 intent families (including narrative, synthesis, out-of-scope, personal), dual-threshold classification (0.80 hard accept / 0.40 soft accept), and graceful rejection with suggestion chips. At that stage (January 2026), eval quality improved to 98.4% (60/61) while reducing costs and latency; the suite has since grown to 64/64 (100%).

---

## Lessons Learned

### MVP-First Thinking Pays Off

I originally planned to build a React + FastAPI stack. I'm glad I didn't. Streamlit let me validate the core RAG architecture in two weeks instead of three months. The UI isn't perfect, but the search quality and system prompt design are solid — those are the hard parts.

### Data Quality > Fancy Algorithms

I spent more time curating STAR stories and validating metadata than I did on the ML pipeline. That discipline paid off: every AI response traces to auditable source data, and users trust the system because it never makes things up.

### The Strangler Fig Refactor

By October 2025, the main app.py file had grown to over 5,000 lines. It was unmaintainable. I used the strangler fig pattern to extract components one by one — timeline_view.py, story_detail.py, landing_view.py — until the core file was under 1,000 lines. The architecture is now modular enough that I can hand off individual components to AI coding assistants without them needing to understand the whole system.

### The "Can You Defend It?" Standard

Every claim in this portfolio passes one test: Can I defend it in an interview? If someone asks, "How do you know it was 4x faster?" I have the answer. If someone asks, "What exactly did you do versus the team?" I can explain my specific contribution. This standard eliminated much of the impressive-sounding but unverifiable content.

---

## Product Roadmap

### Strategic Roadmap Translation: Now, Next, Later

The MattGPT roadmap follows a deliberate three-phase evolution strategy, balancing immediate value delivery with long-term product vision.

---

### NOW (MVP: The Verifiable Foundation)

**Goal:** Establish data integrity, launch core filtering, and secure follow-up intelligence.

**Status:** ✅ Complete (December 2025)

**Completed Milestones:**

✅ **Phase 1 & 2 Foundations**
- Core semantic search implementation
- `echo_star_stories.jsonl` data structure finalized
- Streamlit frontend deployed
- {{ site.data.facts.story_count_label }} stories curated and validated

✅ **Phase 3: Polish & Filtering**
- Streamlit UI refinement (colors, spacing, gradients)
- "About Matt" hero panel with metrics badges
- Core filters: Role, Client, Tags, 5P taxonomy
- Three view modes: Table, Card, Timeline

✅ **Intelligence Layer**
- Gated Export/Share functionality
- Persona Scoring (session depth tracking)
- Email capture for high-intent leads

**Key Outcomes:**
- Semantic search with confidence-based filtering validated through manual testing
- 100% data compliance with STAR method
- Deployed to production at [askmattgpt.streamlit.app](https://askmattgpt.streamlit.app)

---

### NEXT (Ongoing: Feature Delivery on Streamlit)

**Goal:** Continue delivering features on the proven Streamlit platform. Enhance search quality, add capabilities, and align content with industry standards.

**Recent Deliveries (Q1–Q2 2026):**
- ✅ Role Match — paste a JD, see how Matt's experience maps to requirements (Phases 1-3 shipped April, Phase 4 slice 1 shipped May: lock icon + password gate)
- ✅ Triage agent surface — `scripts/assess_jd.py` CLI wraps the same engine (`jd_assessor.py`) for external agent orchestration
- ✅ Mobile responsive design (1,520 lines in `mobile_overrides.py`)
- ✅ Query analytics (Google Sheets logger, 32-column schema)
- ✅ BDD test suite — {{ site.data.facts.bdd_summary }}
- ✅ Eval framework expanded to 64 golden queries (100% pass rate)
- ✅ Data quality pass across 85+ stories

**Recent Deliveries (continued):**
- ✅ Public tags enrichment — `generate_public_tags.py` uses GPT-4o with SFIA/O*NET/LinkedIn taxonomy alignment. `public_tags` flows through the entire pipeline: JSONL generation, embedding construction, Pinecone metadata, app startup, filtering, scoring, and UI display.

**Open Items:**
See [BACKLOG.md](https://github.com/mcpugmire1/llm_portfolio_assistant/blob/main/BACKLOG.md) in the MattGPT repo for current open work. Key areas include Role Match Phase 4, semantic router improvements, and analytics dashboard.

---

### LATER (Phase 3)

See [BACKLOG.md](https://github.com/mcpugmire1/llm_portfolio_assistant/blob/main/BACKLOG.md) for current open work. Key areas include Role Match Phase 4 slices 2-3 (agentic bypass, private assessment view), analytics dashboard, and semantic router improvements.

---

## Future Vision

### The Bigger Picture: Credibility as a Service

MattGPT started as a personal portfolio, but the underlying thesis is universal: professionals shouldn't have to choose between depth and discoverability.

Every consultant has a career's worth of stories they can't effectively communicate. Every recruiter spends hours manually searching through resumes that all look the same. Every hiring manager wishes they could ask follow-up questions before deciding who to interview.

I envision a future where:
- Every professional has an AI that can articulate their unique value
- Hiring managers can query candidate portfolios with natural language
- Career stories are structured, auditable, and composable
- The interview process starts with verified proof, not generic claims

This isn't about replacing human judgment — it's about augmenting trust at scale.

### What's Next

A React migration remains a future consideration if a forcing function emerges. But the more interesting question is whether this becomes a product. Could consulting firms use this for their bench? Could job platforms integrate structured portfolios? Could this become the LinkedIn alternative that actually proves what people claim?

I don't know yet. But the foundation is solid, and the thesis is validated.

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
- Strangler fig pattern: 5,765 → 284 lines in app.py
- Extracted modular components
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

Building MattGPT has been equal parts technical challenge and product discovery. What started as a personal problem — "How do I make a full career of experience searchable?" — became a demonstration of modern AI product development: RAG architecture, semantic search, data governance, and user-centered design.

**The core thesis remains:**

> **Replace "I'm experienced" with "Here's exactly what I did, how I did it, and the measurable results."**

Every line of code, every STAR story, every design decision serves that mission: **proof over claims.**

---

**Related Documentation:**
- [Product Vision](/docs/01-product-vision) - Strategic positioning and user personas
- [Technical Architecture](/docs/02-technical-architecture) - RAG pipeline and system design
- [UX Design Process](/docs/03-ux-design-process) - Wireframes and interaction design
- [RAG Quality Evaluation](/docs/11-testing-and-quality) - How evals validate quality (100% pass rate, 64/64)

---

*Last Updated: June 2026 (Staleness audit: eval 100%/64q, BDD {{ site.data.facts.bdd_summary }})*
*Version: 1.5*
