---
title: "MattGPT: The Build"
description: "The record of how MattGPT was designed, built, and kept honest. Read it as the audit, not the pitch."
---

# MattGPT: The Build

> This is the engineering and design record behind MattGPT. The live portfolio is at [askmattgpt.streamlit.app](https://askmattgpt.streamlit.app).

This is the record of how MattGPT was designed, built, and kept running. If you came from the app, you have seen what it does. This is what sits underneath: the decisions, the discipline, and where it falls short. Read it as the audit, not the pitch.

## How to read this

I held the build to four principles. They are also the bar you can hold it to.

**Proof over claims.** Every claim traces to a real, sourced story. If it cannot cite its evidence, it does not reach a user.

**Honest before impressive.** The system says when it does not know and will not inflate a match. Where the engineering is thin, this record says so.

**Design from the audience, not the feature.** The surfaces came from real visitor journeys, not a feature wishlist.

**Auditable governance.** Two layers, structured stories with real metrics and a tagged retrieval layer, with the documentation kept in sync with what is running and audited when it drifts.

## What it is made of

MattGPT has four surfaces, each built for a different visitor:

- **My Work** is the project record, browsable and filterable.
- **Ask Agy** answers questions from sourced career stories.
- **Role Match** takes a job description and returns an honest fit assessment.
- **My Profile** is the fast read for someone deciding in under a minute.

Under Ask Agy is a retrieval pipeline: stories embedded in Pinecone, routed by intent with an explicit out-of-scope path for questions it should refuse, and answered by GPT-4o with sources attached.

## The proof

How do I know it works? The retrieval layer is gated by an eval suite. Right now it is 64 of 64 on a golden-query set I keep adding to whenever I find a question it should have handled and did not. A perfect score on a set I curate is not a brag. It is a fast signal when a change breaks retrieval before anyone sees it. Under the eval sit unit tests and a BDD and end-to-end suite that exercises the real workflows. The full breakdown is in the [testing and quality](docs/11-testing-and-quality) doc.

## The decisions

Hundreds of calls went into this across data, retrieval, the surfaces, voice, and governance. The full record is in the backlog and the [audience journeys](docs/03-ux-design-process) doc. A few show the logic behind the rest.

**Role Match scores honestly, not generously.** Role Match takes a job description and rates how well I fit it. A tool like that has an obvious pull: tune the score so I look like a strong match for everything. I built it the other way. It scores straight, surfaces the gaps, and shows the evidence behind the fit. A flattering match tool is worthless to a recruiter and a liability to me. The honest version is the only one worth building.

**The surfaces are named for the visitor, not the feature.** I first built the site around what it does, organized by feature. Testing it against real recruiter and hiring personas told me it was feature-led when the visitor is job-led. A recruiter with a job description does not care that there is a search engine underneath. They care whether I fit and whether they can forward it in ninety seconds. So I named the surfaces for what the visitor came to do, My Work, Ask Agy, Role Match, My Profile, not for what each page is.

**The work browser is dense, not pretty.** It is a sortable, filterable grid, not cards that expand one at a time. The cards looked cleaner. But someone comparing many projects at once needs density and side-by-side scanning more than a nicer single view. I built for the person scanning, not for the demo.

**A cover-letter generator was cut.** It was the obvious next feature, and I closed it as Won't Do. A generator like that pulls the system toward confident writing that runs ahead of the evidence, which is the one thing this cannot afford. Saying no was the decision.

## How it is built, and where it is thin

Small stack on purpose: Claude Code at the command line for the build, GitHub for source and history, and a webhook that deploys to Streamlit Cloud on push. There is no enterprise CI/CD here, and claiming otherwise would be the first thing to distrust. One person built this. The question worth asking is what discipline holds at that size.

What holds: the eval gates the retrieval layer, so a regression shows up as a failing number, not a user complaint. BDD scenarios run red to green before a feature counts as done. A rules file sets the architectural conventions, and a pre-commit check keeps these pages from drifting off what is running.

What does not: the rules are not enforced by a pipeline, so they are not always followed. Claude Code, the AI doing the implementation, will rebuild things that already exist or quietly change a protected value, and I catch it in review, not at a gate. Holding that line over a fast, capable, not-always-compliant tool is the real daily work, and it is closer to leading a team than a green pipeline would be.

To keep this honest, I audited the docs against the running system in June 2026. It caught a stale metric, retired wording still sitting in the tests, and the rules file itself wrong on two values where this spec was right. Docs drift from code in every real system. What matters is having something that catches it. The [audit](docs/audit-2026-06-15) is committed alongside the docs it corrects.

With a team I would put the eval and BDD suites behind required checks, add a staging deploy, and lint-enforce the rules instead of writing them in prose. I have not, because for one person that machinery costs more than it returns. Knowing where that tradeoff flips is the job.

## Go deeper

{% include doc-nav.html descriptions=true %}

The live application is at [askmattgpt.streamlit.app](https://askmattgpt.streamlit.app). The application source lives in a [separate repository](https://github.com/mcpugmire1/llm_portfolio_assistant); this repository holds the design and the record.

## Contact

Matt Pugmire. [LinkedIn](https://www.linkedin.com/in/matt-pugmire). [Ask Agy](https://askmattgpt.streamlit.app).
