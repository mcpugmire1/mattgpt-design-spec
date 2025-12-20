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

Traditional portfolios are static PDFs that don't scale. Recruiters and hiring managers can't easily search 130+ projects by methodology, outcome, or domain.

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

In September 2023, I left Accenture after 18.5 years. I'd built their Cloud Innovation Center from scratch, scaled it to 150+ professionals, and helped Fortune 500 companies modernize platforms worth billions. But I was burned out, underwent a few surgeries, and I needed time to figure out what came next.

Six months into my sabbatical, I started updating my resume. That's when it hit me: I had 20 years of transformation stories, and I was trying to squeeze them into two pages of bullet points. The format was fundamentally broken.

Recruiters would spend 30 seconds scanning my PDF. How could they possibly understand the nuance of scaling engineering teams from 4 to 150? Or the pattern recognition I'd developed across 55 banking projects? They couldn't. Nobody could.

I'd been experimenting with RAG architectures and vector search for a client POC, and the idea clicked: What if I could build an AI that actually understood my work — not through keyword matching, but through semantic understanding of outcomes, methodologies, and context?

I gave myself two weeks to build an MVP. The goal was simple: answer one question better than a resume ever could — "Show me examples of Matt scaling agile transformations with measurable outcomes."

I named it after Agy, my late Plott Hound — short for Agador Spartacus from "The Birdcage." He was a tracker — goofy yet patient, determined, excellent at finding exactly what you were looking for. That's what I wanted this AI to be.

---

## Technical Implementation

### Tech Stack

**Core Technologies:**

| Technology | Purpose | Why This Choice |
|------------|---------|-----------------|
| 🐍 **Python 3.11** | Primary language | Rich ML/AI ecosystem, rapid prototyping |
| **Streamlit** | Frontend framework (MVP) | 2-week build time vs 3+ months for React |
| 🤖 **OpenAI GPT-4** | LLM for response generation | State-of-the-art reasoning and synthesis |
| 🌲 **Pinecone** | Vector database | Managed solution, fast ANN search |
| **Sentence Transformers** | Embedding generation | all-MiniLM-L6-v2 (384-dim, fast) |
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
   │  (Sentence-BERT)    │
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
   │   Hybrid Search     │
   │  (80% Semantic +    │
   │   20% Keyword)      │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐
   │  RAG Orchestrator   │
   │  (GPT-4 + Context)  │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐
   │  Synthesized        │
   │  Response +         │
   │  Source Citations   │
   └─────────────────────┘
```

### The Secret Sauce: Hybrid Retrieval Strategy

One of the core technical innovations in MattGPT is the **hybrid search** approach that balances semantic understanding with keyword precision.

**The Algorithm:**

```python
def hybrid_search(query: str, alpha: float = 0.8) -> List[Dict]:
    """
    Blends semantic similarity (80%) with keyword matching (20%)
    to balance conceptual understanding with exact-term precision.

    Args:
        query: User's natural language question
        alpha: Weight for semantic similarity (0.8 = 80%)

    Returns:
        List of ranked stories with blended relevance scores
    """
    # Semantic retrieval: Understanding meaning
    semantic_results = pinecone_query(
        embed(query),
        top_k=30
    )

    # Keyword retrieval: Exact term matching
    keyword_results = bm25(
        query,
        corpus,
        top_k=30
    )

    # Blend and re-rank
    return blend_scores(
        semantic_results,
        keyword_results,
        alpha=alpha
    )
```

**Why This Matters:**

- **Semantic Search (80%):** Understands that "bootstrap it" and "start a new project" are conceptually similar
- **Keyword Precision (20%):** Ensures exact matches for critical terms (client names, specific technologies, role titles)
- **Re-ranking:** Boosts results by relevance + recency + credibility

This approach achieved **87% retrieval accuracy** with an average response time of **1.2 seconds**.

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

### 🔍 Hybrid Retrieval Strategy

**80% Semantic Search:**
- Understanding meaning, not just keywords
- Example: "bootstrap it" vs "start a new project" are semantically similar
- Handles synonyms, related concepts, and contextual understanding

**20% Keyword Precision:**
- Ensures exact matches for critical terms
- Client names, specific technologies, role titles
- Prevents false positives from pure semantic matching

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

**Conversational Workflow:**
- Streamlit's stateful UI for session management
- Context retention across multi-turn conversations
- Intent classification for query routing

**Semantic Embeddings:**
- Sentence-BERT (all-MiniLM-L6-v2)
- 384-dimensional vector space
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

### The Hybrid Search Problem

Pure semantic search was too fuzzy. Ask about "JPMorgan," and it might return stories about "banking transformation" that never mention the client. Pure keyword search was too rigid — "CI/CD pipelines" wouldn't match "continuous delivery automation."

The solution was hybrid retrieval: 80% semantic similarity for conceptual understanding, 20% keyword matching for exact terms. Getting that balance right took three weeks of tuning.

### The Hallucination Problem

Early versions of the system would confidently generate plausible-sounding details that never happened. "Matt led a team of 200 engineers," when the actual number was 150. "The project saved $50M" when I'd never quantified the savings.

The fix was two-fold: mandatory STAR structure (every story needs verifiable Situation, Task, Action, Result) and aggressive fact-checking. I went through all 130+ stories line by line, deleting anything I couldn't defend in an interview. If I couldn't cite a specific metric or name a specific outcome, it got cut.

### The Streamlit CSS Problem

Streamlit is great for rapid prototyping. It's terrible for custom styling. Class names change between versions. CSS selectors that work on desktop break on mobile. The st-emotion-cache-* classes are generated dynamically and can't be targeted reliably.

I learned to use data-testid attributes and container keys for CSS targeting. The global_styles.py file grew to 600+ lines of carefully scoped overrides. It's not pretty, but it works.

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
- 130+ stories curated and validated

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
- 87% retrieval accuracy
- 1.2s average response time
- 100% data compliance with STAR method
- Deployed to production at [askmattgpt.streamlit.app](https://askmattgpt.streamlit.app)

---

### NEXT (Phase 2: Efficiency and Search Intelligence)

**Goal:** Enhance search quality, streamline information delivery, and align content with industry standards.

**Timeline:** Q2 2025

**Planned Features:**

🎯 **Public Tags Enrichment**
- Complete SFIA / O\*NET / LinkedIn taxonomy mapping
- Enable discoverability through industry-standard skills
- Power cross-portfolio pattern recognition

🎯 **Hybrid Keyword + Semantic Search Optimization**
- Fine-tune alpha weighting based on query intent
- Implement query rewriting for common patterns
- Add spell-check and autocomplete

🎯 **Enhanced UX**
- Copy-to-clipboard for quick story sharing
- Portfolio integration (Notion, LinkedIn sync)
- Improved mobile responsiveness

🎯 **React + FastAPI Migration**
- Decouple frontend from Streamlit
- Implement micro-frontend architecture
- Horizontal scalability for 10K+ concurrent users

**Success Metrics:**
- Retrieval accuracy: 87% → 92%
- Mobile user engagement: +40%
- Lead qualification rate: >20%

---

### LATER (Phase 3: Productization and Disruption)

**Goal:** Unlock high-value use cases through deep AI integration, matching, and platform scalability.

**Timeline:** 2026+

**Planned Features:**

🔮 **Job Fit & Matching (Disruptive Feature)**
- Paste job description → generate tailored response
- Auto-extract requirements and match to relevant stories
- Synthesize custom "Why Matt is a fit" narrative
- Export as formatted cover letter or talking points

🔮 **Advanced RAG Infrastructure**
- Upgrade to LangChain/LlamaIndex for orchestration
- Implement agent-like assistant persona
- Context-aware follow-up suggestions
- Multi-project synthesis ("Connect these 3 stories")

🔮 **Platform Scalability**
- Local embedding storage option
- Multi-region deployment for global users
- Real-time collaboration (share sessions)
- White-label SaaS offering for consulting firms

🔮 **Governance & Stability**
- Script version tagging and rollback capability
- Automated JSONL backup routines
- A/B testing infrastructure for prompt optimization
- Cost optimization monitoring

**Success Metrics:**
- Support 100K+ monthly active users
- 99.9% uptime SLA
- Job Fit feature: >60% conversion to personalized outreach
- White-label revenue: First 5 enterprise customers

---

## Future Vision

### The Bigger Picture: Credibility as a Service

MattGPT started as a personal portfolio, but the underlying thesis is universal: professionals shouldn't have to choose between depth and discoverability.

Every consultant has 10-20 years of stories they can't effectively communicate. Every recruiter spends hours doing manual keyword searches through resumes that all look the same. Every hiring manager wishes they could ask follow-up questions before deciding who to interview.

I envision a future where:
- Every professional has an AI that can articulate their unique value
- Hiring managers can query candidate portfolios with natural language
- Career stories are structured, auditable, and composable
- The interview process starts with verified proof, not generic claims

This isn't about replacing human judgment — it's about augmenting trust at scale.

### What's Next

The immediate roadmap is a React migration to improve performance and scalability. But the more interesting question is whether this becomes a product. Could consulting firms use this for their bench? Could job platforms integrate structured portfolios? Could this become the LinkedIn alternative that actually proves what people claim?

I don't know yet. But the foundation is solid, and the thesis is validated.

---

## Appendix: Build Timeline

### April 2025: Foundation
- Curated the first 50 STAR stories from Accenture tenure
- Set up Pinecone vector database
- Implemented basic Streamlit UI with semantic search
- Validated RAG pipeline with OpenAI GPT-4

### May-June 2025: Expansion
- Grew corpus to 115+ stories covering full career
- Added filters (client, role, domain, industry)
- Designed three view modes: Table, Card, Timeline
- Built "How Agy Searches" transparency modal

### July-August 2025: Polish
- User testing with recruiters and hiring managers
- Refined system prompt based on feedback
- Implemented hybrid search (semantic + keyword)
- Added confidence scoring and source citations

### September 2025: Fact-Check Sprint
- Line-by-line audit of all stories
- Removed AI-generated fabrications
- Ensured 100% defensibility standard
- Finalized 130+ verified stories

### October 2025: Architecture Refactor
- Strangler fig pattern: 5,000 → 1,000 lines in app.py
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

Building MattGPT has been equal parts technical challenge and product discovery. What started as a
solution to a personal problem — "How do I make 20 years of experience searchable?" — evolved into
a demonstration of modern AI product development: RAG architecture, hybrid retrieval strategies,
data governance, and user-centered design.

**The core thesis remains:**

> **Replace "I'm experienced" with "Here's exactly what I did, how I did it, and the measurable results."**

Every line of code, every STAR story, and every design decision serves that mission: **credibility
through proof, not claims.**

---

**Related Documentation:**
- [Product Vision](/mattgpt-design-spec/docs/01-product-vision) - Strategic positioning and user personas
- [Technical Architecture](/mattgpt-design-spec/docs/02-technical-architecture) - RAG pipeline and system design
- [UX Design Process](/mattgpt-design-spec/docs/03-ux-design-process) - Wireframes and interaction design

---

*Last Updated: December 2025*
*Version: 1.1 (Complete with Personal Narrative)*
