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

Four visitor types shaped the surface design. Each needs something different from the same story corpus.

**A recruiter with a job description** needs a fast, forwardable fit signal. Role Match takes the JD, rates each requirement with the evidence behind it, summarizes the gaps, and produces an artifact a hiring manager can act on in under 90 seconds.

**A recruiter doing inbound triage** needs six facts in 30 seconds: level, last company, last team size, geo, current status, target titles. My Profile is the entry point. No digging required.

**A hiring CTO, VP Engineering, or Head of Platform** got the link from a trusted contact and has genuine intent. They will browse one or two stories, then navigate to Ask Agy with a specific, possibly hard question. Whether that answer is honest is the make-or-break moment. Ask Agy is not just a feature for this visitor; it is the test.

**A referrer** has already decided to make the intro. They need one-sentence positioning they can lead with, two or three substantiating facts, and confidence that what the recipient sees matches what the intro promised.

The detailed journey logic, failure modes, and surface rationale for each visitor is in [Audience Journeys](03-ux-design-process).

---

## Data Model & Two-Layer Governance

The corpus runs on two layers. The integrity layer is the STAR structure: every story carries Situation, Task, Action, Result, and at least one metric, which is what lets any answer trace back to a sourced project. The intelligence layer is the tagging that makes retrieval work: a 5P breakdown (Person, Place, Purpose, Process, Performance) and public tags aligned to SFIA, O*NET, and LinkedIn vocabularies so the corpus connects to how roles are actually searched. The full schema is in Data Model.

---

## MattGPT AI Core

**System Prompt & Integrity Mandate**

> **Implementation Note:** The system prompt is built by `build_tool_system_prompt()` in `prompts.py`, with MATT_DNA and Matt's profile facts supplied from `backend_service.py`. Routing and query validation live outside the prompt, in `semantic_router.py` and `nonsense_filters.jsonl`. The documented principles guide all three.

Every answer draws on the stories its searches return, Matt's profile and the conversation. The retrieved stories appear as clickable source chips linking back to the full STAR story, so a claim can be checked against the original. The prompt instructs Agy not to invent examples beyond Matt's stories; no code guarantees it. The voice blends a strategic and a pragmatic register; the full archetype is in the Agy Voice Guide.

---

## Project Scope & Boundaries

MattGPT is scoped deliberately. It answers only from Matt's stories, his profile and the conversation: it won't give generic career advice, synthesize opinions, or speak to anything outside them. Displayed metrics are read-only, never editable from the interface. The point is a focused proof engine, not a chatbot.

---

**The Mission:**

> Replace "I'm experienced" with "Here's exactly what I did, how I did it, and the measurable results."

---

**Related Documentation:**
- [Technical Architecture](02-technical-architecture) - RAG pipeline and system design
- [Audience Journeys](03-ux-design-process) - The four visitor journeys the design is derived from
- [Building MattGPT](04-building-mattgpt) - Development journey

---

*Last updated: {{ site.data.page_dates['01-product-vision'] }}*
