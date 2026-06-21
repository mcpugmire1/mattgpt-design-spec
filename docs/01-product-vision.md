# Product Vision & Strategy

**MattGPT: The Credibility Engine**

> This document covers the strategic rationale for MattGPT: four visitor paths, the two-layer data governance model, and the guardrails that keep responses grounded in source data. A reader who finishes it understands what the system is for, who it serves, and what it will never do.

---

## Table of Contents

1. [The Credibility Engine: WHY, HOW, WHAT](#the-credibility-engine)
2. [Who This Is For](#who-this-is-for)
3. [Data Model & Two-Layer Governance](#data-model--two-layer-governance)
4. [MattGPT AI Core: System Prompt & Integrity Mandate](#mattgpt-ai-core)
5. [Project Scope & Boundaries](#project-scope--boundaries)

---

## The Credibility Engine

**Tagline:** *"Matt doesn't claim credibility — he proves it in real time."*

### WHY: Establish Credibility

**Eliminate Doubt**

The core purpose is to replace generic claims and shallow experience with instant, quantifiable proof.

Traditional portfolios rely on self-promotion and unverifiable claims. Recruiters and hiring managers waste time validating experience, and candidates struggle to differentiate themselves from generic resumes.

---

### HOW: Act as a Credibility Engine

**Automatically mining 100+ real projects to surface the right evidence instantly.**

MattGPT functions as an intelligent search and retrieval system that:

- **Surfaces Specific STAR Stories** - Structured narratives with verifiable context
- **Identifies Methodology Patterns** - e.g., "Scaled payments at 3 banks"
- **Quantifies Measurable Outcomes** - e.g., "Accelerated delivery 4x"

**Key Capability:** The AI doesn't generate claims—it **retrieves and presents proof** from structured, auditable source data.

---

### WHAT: Specific Outcomes

**The AI-Powered Guide**

MattGPT delivers verifiable outcomes for three primary use cases:

#### 1. Recruiter Vetting
- **Need:** Fast validation of skills and experience
- **Outcome:** Instant keyword matching + proof of outcomes
- **Value:** Verifiable skills evidence without back-and-forth

#### 2. Interview Prep
- **Need:** Deep preparation with specific examples
- **Outcome:** STAR-formatted stories ready to use
- **Value:** Walk into interviews with concrete narratives

#### 3. Client Pitches
- **Need:** Relevant case studies and proven results
- **Outcome:** Industry-specific examples with metrics
- **Value:** Build trust through demonstration, not declaration

---

## Who This Is For

Four visitor types shaped the surface design. Each needs something different from the same corpus.

**A recruiter with a job description** needs a fast, forwardable fit signal. Role Match takes the JD, returns a scored assessment with evidence-backed ratings and a gap summary, and produces an artifact a hiring manager can act on in under 90 seconds.

**A recruiter doing inbound triage** needs six facts in 30 seconds: level, last company, last team size, geo, current status, target titles. My Profile is the entry point. No digging required.

**A hiring CTO, VP Engineering, or Head of Platform** got the link from a trusted contact and has genuine intent. They will browse one or two stories, then navigate to Ask Agy with a specific, possibly hard question. Whether that answer is honest is the make-or-break moment. Ask Agy is not just a feature for this visitor; it is the test.

**A referrer** has already decided to make the intro. They need one-sentence positioning they can lead with, two or three substantiating facts, and confidence that what the recipient sees matches what the intro promised.

The detailed journey logic, failure modes, and surface rationale for each visitor is in [Audience Journeys](/docs/03-ux-design-process).

---

## Data Model & Two-Layer Governance

**How do you ensure the AI's answers are trustworthy?**

By creating MattGPT as a system that is **reliable, auditable, and engineered for high-quality information retrieval.**

### The Two-Layer Architecture

| Layer | Technical Name | Purpose (The WHY) | Key Data Fields |
|-------|---------------|-------------------|-----------------|
| **Layer 1: Integrity** (The Core) | Mandatory STAR Method | Guarantees every story has **verifiable context** and **measurable results (metrics)**, fulfilling the promise of "Eliminate Doubt" | Situation, Task, Action, Result (with metrics) |
| **Layer 2: Intelligence** (The AI Fuel) | The Tagging Systems (5P + Semantic) | Enables complex **Hybrid Search** and **Pattern Recognition** for the AI (e.g., "Scaled payments at 3 banks") | 5P Taxonomy (Person, Place, Purpose, Process, Performance), Semantic Public Tags (O*NET/SFIA alignment) |

---

### Layer 1: Integrity (The Foundation)

**Mandatory STAR Method**

Every project in the MattGPT corpus follows the STAR framework:

- **Situation:** Context and business challenge
- **Task:** Specific objective or problem to solve
- **Action:** Methodology, decisions, and execution
- **Result:** Measurable outcomes with metrics

STAR structure rules out generic claims, forces concrete examples with business context, and creates an auditable reference chain from every AI answer back to source data.

**Governance Rule:** No project can be indexed without completing all four STAR fields + at least one quantifiable metric.

---

### Layer 2: Intelligence (The Discovery Layer)

**The Tagging Systems**

To enable sophisticated search and pattern recognition, every project is enriched with two tagging systems:

#### 5P Taxonomy (Private Metadata)
- **Person** - Role, seniority, team structure
- **Place** - Client, industry, geographic context
- **Purpose** - Capability area, transformation type
- **Process** - How the work was done; approach and methods
- **Performance** - Measurable outcomes and results

#### Semantic Public Tags (Industry Standards)
- **O*NET Competencies** - Standardized skill taxonomy
- **SFIA Framework** - IT professional skills
- **LinkedIn Skills** - Common search terms

The tagging layer enables semantic search and cross-project pattern recognition ("Show me all payment modernization projects"), supports synthesis across related work, and aligns with industry-standard taxonomies so the vocabulary connects to how roles are actually searched.

---

## MattGPT AI Core

**System Prompt & Integrity Mandate**

> **Implementation Note:** The "system prompt" is implemented as a distributed architecture across multiple services (semantic_router.py, rag_service.py, backend_service.py) rather than a single prompt file. The documented principles guide routing logic, query validation, and response formatting.

### The Operational Mandate

**Your Purpose: The Credibility Engine**

The system exists to surface relevant STAR stories and connect patterns across 100+ projects, driving the user to the core thought:

> *"Matt consistently delivers measurable transformation results — and here's the specific proof."*

---

### Core Directive: Anchor Every Answer in Proof

- Anchor every answer in specific projects (Client, Title, Outcome)
- Lead with outcomes, then methodology ("Achieved 4x acceleration by implementing...")
- Infer user intent (Interview Prep, Due Diligence, Pitch) to tailor the response structure

**Data Logic:**
- Semantic search across STAR, 5P, and Competencies to prioritize relevance and pattern extraction

---

### The Archetype & Governance

**The Archetype: Trusted, Pragmatic Advisor**

The voice blends two registers (the full archetype is in the [Agy Voice Guide](/docs/05-agy-voice-guide)):
- **Strategic Advisor** — Executive-ready, outcome-focused
- **Pragmatic Operator** — Grounded in results, implementation-focused

**Tone:** Warm, confident, and professional—never robotic or buzzword-heavy.

---

### Non-Negotiable Guardrails

❌ **No generic career advice or philosophical answers**
- The system only discusses Matt's specific project experience
- No hypotheticals, no general industry commentary

❌ **No corporate buzzword soup or robotic language**
- Use concrete, human language
- Avoid jargon without context

❌ **Never pretend to know things outside Matt's portfolio**
- If a query falls outside the 100+ projects, acknowledge limitations
- Suggest alternative search terms or clarify scope

---

### The Integrity Mandate

**All answers MUST be auditable by providing a source reference to the underlying project data.**

**Implementation:**
- Every AI-generated response includes clickable source chips
- Each source links directly to the full STAR story
- Users can verify claims by reading the original content
- No "hallucinated" metrics or outcomes—everything traces to source

**Example Response Format:**
```
✅ Good: "Matt accelerated delivery 4x at JP Morgan Chase by implementing
         CI/CD pipelines and automated testing frameworks."
         [Source: Agile Transformation at JP Morgan Chase]

❌ Bad:  "Matt is experienced in DevOps and has worked with major banks."
         [No source, no metric, no proof]
```

---

## Project Scope & Boundaries

**MVP Definition**

### IN SCOPE: Must Do (Integrity & Proof)

#### 1. Data Structure
**Requirement:** All content MUST be schema-driven, utilizing the **STAR Method** (Situation, Task, Action, Result) as the mandatory format for every project.

**Rationale:** Ensures consistency, auditability, and proof-based responses.

---

#### 2. Verifiability
**Requirement:** The system MUST provide direct **source references (audit trail)** from any AI-generated answer back to the source Key Metric or STAR Story.

**Rationale:** Maintains trust and allows users to verify claims.

---

#### 3. Core Query
**Requirement:** The Explore Stories interface MUST provide filtering by **Industry, Technology, and Key Outcome** for rapid recruiter comparison.

**Rationale:** Enables recruiters to efficiently screen candidates.

---

#### 4. Performance
**Requirement:** The Detail View (STAR Method & Metrics) MUST load in **under 1 second** to ensure instant, reliable proof delivery.

**Rationale:** Speed builds trust; delays create doubt.

---

### OUT OF SCOPE: Must Never Do (Focus & Anti-Chatbot)

#### 1. Generative AI Scope
**Guardrail:** The system MUST NEVER generate generic career advice, synthesize opinions, or answer questions outside the scope of the **100+ verified STAR-formatted stories**.

**Rationale:** Prevents hallucination and maintains credibility.

---

#### 2. Data Mutability
**Guardrail:** All displayed Key Metrics (e.g., "Accelerated delivery 4x") MUST NEVER be interactive, editable, or changeable via the front-end interface.

**Rationale:** Protects data integrity and prevents tampering.

---

#### 3. User Tracking & Personalization
**Guardrail:** The MVP MUST NEVER include user logins, account creation, or attempt to personalize content based on browsing history.

**Rationale:** Reduces complexity; focuses on core value delivery.

---

#### 4. Monetization Features
**Guardrail:** The system MUST NEVER include payment gateways, subscriptions, or premium content features.

**Rationale:** MVP is a portfolio showcase, not a SaaS product (yet).

---

**The Mission:**

> Replace "I'm experienced" with "Here's exactly what I did, how I did it, and the measurable results."

---

**Related Documentation:**
- [Technical Architecture](/docs/02-technical-architecture) - RAG pipeline and system design
- [UX Design Process](/docs/03-ux-design-process) - User experience decisions
- [Building MattGPT](/docs/04-building-mattgpt) - Development journey

---

*Last updated: {{ site.data.page_dates['01-product-vision'] }}*
