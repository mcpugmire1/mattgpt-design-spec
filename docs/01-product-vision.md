# Product Vision & Strategy

**MattGPT: The Credibility Engine**

> This document covers the strategic rationale for MattGPT: four visitor paths, the two-layer data governance model, and the guardrails that keep responses grounded in source data. A reader who finishes it understands what the system is for, who it serves, and what it will never do.

---

## Table of Contents

1. [Who This Is For](#who-this-is-for)
2. [Data Model & Two-Layer Governance](#data-model--two-layer-governance)
3. [MattGPT AI Core: System Prompt & Integrity Mandate](#mattgpt-ai-core)
4. [Project Scope & Boundaries](#project-scope--boundaries)

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

The corpus runs on two layers. The integrity layer is the STAR structure: every story carries Situation, Task, Action, Result, and at least one metric, which is what lets any answer trace back to a sourced project. The intelligence layer is the tagging that makes retrieval work: a 5P breakdown (Person, Place, Purpose, Process, Performance) and public tags aligned to SFIA, O*NET, and LinkedIn vocabularies so the corpus connects to how roles are actually searched. The full schema is in Data Model.

---

## MattGPT AI Core

**System Prompt & Integrity Mandate**

> **Implementation Note:** The "system prompt" is implemented as a distributed architecture across multiple services (semantic_router.py, rag_service.py, backend_service.py) rather than a single prompt file. The documented principles guide routing logic, query validation, and response formatting.

Every answer is anchored to specific sourced projects and carries a clickable source chip linking back to the full STAR story, so any claim can be verified against the original. Nothing is generated that doesn't trace to source. The voice blends a strategic and a pragmatic register; the full archetype is in the Agy Voice Guide.

---

## Project Scope & Boundaries

MattGPT is scoped deliberately. It answers only from the sourced project corpus: it won't give generic career advice, synthesize opinions, or speak to anything outside the stories. Displayed metrics are read-only, never editable from the interface. The point is a focused proof engine, not a chatbot.

---

**The Mission:**

> Replace "I'm experienced" with "Here's exactly what I did, how I did it, and the measurable results."

---

**Related Documentation:**
- [Technical Architecture](/docs/02-technical-architecture) - RAG pipeline and system design
- [Audience Journeys](/docs/03-ux-design-process) - The four visitor journeys the design is derived from
- [Building MattGPT](/docs/04-building-mattgpt) - Development journey

---

*Last updated: {{ site.data.page_dates['01-product-vision'] }}*
