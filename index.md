---
title: "MattGPT: The Build"
description: "The record of how MattGPT was designed, built, and kept honest. Read it as the audit, not the advertisement."
---

![Agy](https://mcpugmire1.github.io/mattgpt-design-spec/images/logos/agy_transparent.png)

# MattGPT: The Build

This is the working record of how MattGPT was designed, built, and is kept running. If you arrived from the app, you have already seen what it does. This is the part underneath: the reasoning, the decisions, and the discipline. Read it as the audit, not the advertisement.

## How to read this

I held the build to four principles. They double as the criteria you can hold it to.

**Proof over claims.** Every capability statement traces to a sourced, structured project story. A claim that cannot cite its evidence does not reach a user.

**Honest before impressive.** The system declines cleanly when it does not know, and it will not inflate a match. Where the engineering is immature, this record says so plainly, including below.

**Design from audiences, not features.** The surfaces and the information architecture were derived from real visitor journeys, not a feature wishlist.

**Auditable governance.** Two layers: structured integrity (STAR stories with real metrics) and retrieval intelligence (tagging and semantic search), with the documentation held in sync with what is actually running, and audited when it drifts.

What follows is the evidence for each.

## The proof, up front

MattGPT runs on a retrieval-augmented pipeline: STAR-structured career stories embedded in Pinecone, retrieved through a semantic router that sorts queries into intent families, including an explicit out-of-scope path for questions it should refuse, and synthesized by GPT-4o with sources attached.

The part worth inspecting is how I know it works. The retrieval layer is gated by a behavioral eval suite. The current result is 64 of 64 on a golden-query set I maintain and extend whenever I find a question it should have handled and did not. A perfect score on a suite I curate is not the headline, and you should not read it as one. What it buys is a fast, specific signal the moment a change degrades retrieval, before any visitor sees it. Underneath the eval sit two more layers: unit tests for component isolation, and a BDD and end-to-end suite that exercises the real workflows. The full breakdown, including what the eval deliberately does not cover yet, is in the [testing and quality](docs/11-testing-and-quality) doc.

## The decisions, and what I rejected

The build involved hundreds of calls across data, retrieval, interface, voice, and governance. The full record lives in the ticketed backlog and the [audience journeys](docs/03-ux-design-process) doc. A few are worth walking, because they show the logic the rest of them followed.

Take the call that cost the most to accept. I had built the site around what it could do, organized by feature: ask the assistant, match a role, explore the work. Then I ran it past CTO and recruiter personas, and the testing told me something I did not want to hear. It was feature-led, and the visitor is job-led. A recruiter holding a job description does not care that there is a semantic search engine underneath. They care whether this person fits the role and whether they can forward it in ninety seconds. The site was answering "what is this" while the visitor was asking "can he do the job." That reframe did not fix one thing. It invalidated the hero, the navigation order, and the entry routing at once, and it became a cluster of work rather than a tidy ticket. I settled it job-led: the surfaces are named for what the visitor came to do (My Work, Ask Agy, Role Match, My Profile), not for what each page is.

A smaller one, same logic. The work-browsing surface uses a dense, sortable, filterable grid rather than cards that expand one at a time. The expanding cards looked cleaner in isolation. But the people who browse many stories at once, a recruiter triaging or a hiring manager comparing, need density and side-by-side scanning more than a prettier single-item view. I chose the surface the audience needed over the one that demoed better.

And one about what not to build. A cover-letter generator was the obvious next feature, and I closed it as Won't Do. A generator like that pulls the system toward confident prose that runs ahead of the evidence, which is the one thing a credibility engine cannot afford. Saying no to a plausible feature was the product decision.

## How it is built, and where it is honest about its limits

MattGPT runs on a deliberately small stack: Claude Code at the command line for implementation, GitHub for source and history, and a webhook that deploys to Streamlit Cloud on push. There is no enterprise CI/CD here, and claiming otherwise would be the first thing a technical reader should distrust. This is one person's system. The interesting question is what discipline survives at that scale.

What holds: the eval suite gates the retrieval layer, so a regression surfaces as a failing number rather than a user complaint. BDD scenarios run red to green before a feature counts as done. A rules file encodes the architectural conventions, and a pre-commit documentation checklist keeps these pages from drifting away from what is actually running.

Where it does not: the rules are not enforced by a pipeline, so they are not always followed. Claude Code, the AI CLI doing the implementation, will rebuild infrastructure that already exists or quietly change a protected value, and the governance is me catching it in review, not a gate stopping it at the door. Holding that line over a fast, capable, not-always-compliant AI tool is the actual daily work, and it is a closer analog to leading a team than a green pipeline would be.

To keep this honest, I ran the first systematic audit of this documentation against the running system in June 2026. It caught a stale headline metric, brand vocabulary the product had retired months earlier still sitting in test expectations, and a case where the rules file itself was wrong on two configuration values where this spec was right. Documentation drifts from code in every real system. What distinguishes a mature one is having a mechanism that finds and corrects the drift. The [audit](docs/audit-2026-06-15) is committed alongside the docs it corrects.

With a team and a budget I would put the eval and BDD suites behind required status checks, add a staging deploy between push and production, and make the architectural rules lint-enforced instead of written in prose. I have not, because for one person the cost of that machinery outruns its value at this size. Knowing where that tradeoff flips is the job.

## Go deeper

The full documentation set, in reference order:

- [Product Vision](docs/01-product-vision): the credibility-engine premise, the WHY/HOW/WHAT framing, the governance model, and the system prompt.
- [Technical Architecture](docs/02-technical-architecture): the RAG pipeline, the semantic router, and the phased evolution plan.
- [Audience Journeys](docs/03-ux-design-process): the four visitor journeys the design is derived from.
- [Building MattGPT](docs/04-building-mattgpt): the development story and what I learned building it.
- [Agy Voice Guide](docs/05-agy-voice-guide): the brand voice for Agy, the Plott Hound assistant.
- [API Reference](docs/09-api-reference): module inventory, key function signatures, session-state values, and environment configuration.
- [Data Model](docs/10-data-model): the JSONL schema, the STAR structure, the tagging taxonomy, and validation rules.
- [Testing and Quality](docs/11-testing-and-quality): the three-layer testing strategy and the eval framework.
- [Data Pipeline](docs/12-data-pipeline): the data flow from source to production retrieval.
- [Migration Architecture](docs/13-migration-architecture): the planned Phase 2 React rebuild, not yet built.

The live application is at [askmattgpt.streamlit.app](https://askmattgpt.streamlit.app). The application source lives in a [separate repository](https://github.com/mcpugmire1/llm_portfolio_assistant); this repository holds the design and the record.

## Contact

Matt Pugmire. [LinkedIn](https://www.linkedin.com/in/mattpugmire/). [Ask Agy](https://askmattgpt.streamlit.app).
