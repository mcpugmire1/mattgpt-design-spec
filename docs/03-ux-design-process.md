# Audience Journeys

> This document describes the four audience journeys that shaped MattGPT's design and the site architecture derived from them.

---

## Table of Contents

1. [Audience Journeys](#audience-journeys)
2. [Site Architecture](#site-architecture--information-hierarchy)

---

## Audience Journeys

MattGPT is designed around four audience journeys, not surfaces. The navigation order and page surfaces fall out of these journeys — not from a feature wishlist.

---

### Journey 1: Cold Recruiter, JD in Hand

**Who:** High-volume recruiter triaging candidates for a specific open role. Paid on placements. 90-second decision window.

**Trigger:** Has a JD and needs to validate fit fast.

**Path:**
1. Lands on Home, passes 3-second test
2. Locates Role Match in navigation (fourth item)
3. Pastes JD → receives fit assessment with evidence-backed ratings
4. Reviews matched experience, gap assessment, and logistics (location, work model, availability)
5. Exports candidate brief → forwards to hiring manager in under 30 seconds

**Goal:** Answer "Can he do the job?" and "Can we hire him?" in under 90 seconds. Export artifact must be self-contained, honest, and hiring-manager-readable.

**Failure modes:** Hero loses the 3-second test with no adjacent substance signal. Role Match buried or hard to find. Scorer over-claims relative to actual experience. Logistics not surfaced.

---

### Journey 2: Cold Recruiter, Inbound Triage

**Who:** Recruiter doing inbound triage — Matt's name surfaced via DM, referral, or past contact. No JD in hand.

**Trigger:** Needs six facts in 30 seconds: level, last company, last team size, geo/relocation, current status, target titles.

**Path:**
1. Lands on Home or navigates directly to My Profile
2. Scans signals panel and professional summary
3. Either saves URL for a future req or moves on

**Goal:** Six facts, 30 seconds. No digging required.

**Failure modes:** My Profile uses pitch-register prose instead of scannable specifics. Signals panel missing target titles or availability status. Home doesn't telegraph "30-second answer here."

---

### Journey 3: Warm-Intro Decision-Maker

**Who:** Hiring CTO, VP Engineering, or Head of Platform. Got the link from a trusted contact with a framing line. Has 5-10 minutes and genuine intent.

**Trigger:** A mutual contact forwarded the site with context.

**Path:**
1. Lands on Home, passes 3-second composition test
2. Scans past the stats row (registers as deck claims)
3. Browses one or two stories in My Work to form a question
4. Navigates to Ask Agy with a specific, possibly hard question
5. Evaluates whether the answer is honest — this is the make-or-break moment
6. Optionally cross-checks Role Match against their own opening
7. Commits to a screening call or forwards the link internally

**Goal:** Commit to a 30-minute screening call with a sharp agenda, or amplify as a secondary referrer.

**Failure modes (by severity):** Methodology context absent — reads as sales/consulting figure, not engineering leader; CTO disqualifies without articulating why. Role Match vs Ask Agy inconsistency (credibility hit). Brand-identity-first hero with no adjacent substance signal.

**Design principle:** Ask Agy is not just a feature for this audience — it's the test. How it handles the hard question shapes the hiring decision more than the corpus content does.

---

### Journey 4: Referrer

**Who:** Someone in Matt's network making a deliberate outbound intro. Three flavors: primary referrer (former colleague), secondary referrer (the warm-intro CTO who decided to amplify), and the two-degree-away referrer who doesn't have specific Matt language.

**Trigger:** They've decided to make the intro. The question is how to compose it.

**What they need:**
- One-sentence positioning they can lead with
- Two or three substantiating facts
- A clean URL to embed
- Confidence that what the recipient sees matches what the intro promised

**Failure modes:** No clear "this is the language about Matt" surface. Voice block uses pitch-register or consulting-deck language the referrer can't reuse in a Slack DM. No copy-intro-language affordance — PDF export and URL copy exist, but pre-composed third-person text formatted for pasting does not.

---

## Site Architecture & Information Hierarchy

### Site Structure

```
Home → capability entry points routing visitors to the right surface

Banking Landing → client and capability filters into banking project stories

Cross-Industry Landing → transformation capability browser across industries

My Work → filterable story browser (Table, Card, Timeline views; inline STAR detail)

Ask Agy → conversational semantic search with source citations and confidence scoring

Role Match → JD-to-experience fit assessment with evidence chips and gap summary

My Profile → fast-read profile; links to the live app
```

For the full search and retrieval pipeline, see [Technical Architecture](/docs/02-technical-architecture).

---

**Related Documentation:**
- [Product Vision](/docs/01-product-vision) - Strategic positioning and user personas
- [Technical Architecture](/docs/02-technical-architecture) - RAG pipeline and system design
- [Building MattGPT](/docs/04-building-mattgpt) - Development journey and lessons learned

---

*Last updated: {{ site.data.page_dates['03-ux-design-process'] }}*
