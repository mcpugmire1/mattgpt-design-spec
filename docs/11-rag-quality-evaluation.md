# RAG Quality Evaluation Framework

**How MattGPT maintains 98.1% accuracy through eval-driven development**

> This document describes the evaluation framework that validates RAG pipeline quality, prevents regressions, and guides architectural decisions through systematic testing of 60+ golden queries.

---

## Table of Contents

1. [Overview](#overview)
2. [Query Categories](#query-categories)
3. [Evaluation Criteria](#evaluation-criteria)
4. [Running Evaluations](#running-evaluations)
5. [Eval-Driven Development](#eval-driven-development)
6. [Adding New Test Cases](#adding-new-test-cases)
7. [Metrics & Reporting](#metrics--reporting)

---

## Overview

**Purpose:** Prevent regressions, validate pipeline changes, and ensure consistent RAG quality

**Framework:** pytest-based golden query testing
**Coverage:** 60+ queries across 6 categories
**Current Pass Rate:** 98.1% (60/61)
**Location:** `tests/eval_rag_quality.py`

### Why This Matters

RAG systems are notoriously hard to test. LLM outputs are non-deterministic, semantic similarity is fuzzy, and "good enough" is subjective. Without systematic evaluation, pipeline changes become high-risk guesswork.

MattGPT's eval framework provides:
- **Regression prevention:** Catch breaking changes before deployment
- **Threshold calibration:** Data-driven tuning of confidence scores
- **Architecture validation:** Quantify impact of major refactors (e.g., Entity Gate removal)
- **Quality gates:** Block merges if eval pass rate drops below 95%

---

## Query Categories

### 1. Entity Detection (Client, Division, Employer)

Tests the system's ability to detect and pin stories based on named entities.

**Example Queries:**
- "Show me Accenture projects"
- "What did Matt do at JP Morgan?"
- "Tell me about Cloud Innovation Center work"

**Expected Behavior:**
- Correct entity detected (Client, Employer, Division)
- Matching story pinned to #1 in results
- High confidence score (≥0.25)

### 2. Behavioral Questions

Interview-style behavioral questions requiring STAR-formatted responses.

**Example Queries:**
- "Tell me about a time Matt failed"
- "How does Matt handle conflict?"
- "Describe a difficult stakeholder situation"

**Expected Behavior:**
- Returns stories with strong behavioral examples
- No meta-commentary ("this demonstrates," "this shows")
- Preserves STAR structure in response

### 3. Technical/Delivery Queries

Questions about technical depth, methodologies, and delivery outcomes.

**Example Queries:**
- "How did Matt achieve 4x faster delivery?"
- "Show me platform engineering work"
- "What agile transformation experience does Matt have?"

**Expected Behavior:**
- Returns stories with technical details or delivery metrics
- Confidence-based filtering (only HIGH confidence for vague queries)
- Correct intent family classification

### 4. Marketing/Landing Page Questions

Queries that should trigger concise, punchy responses suitable for landing pages.

**Example Queries:**
- "What's Matt's superpower?"
- "What makes Matt different?"
- "Why should I hire Matt?"

**Expected Behavior:**
- NO meta-commentary (critical for marketing copy)
- Synthesizes themes across multiple stories
- Short, direct responses (not exhaustive lists)

### 5. Synthesis/Narrative Queries

High-level questions about professional identity and career themes.

**Example Queries:**
- "What's Matt's professional narrative?"
- "Is Matt a builder or a maintainer?"
- "Summarize Matt's career themes"

**Expected Behavior:**
- Triggers synthesis mode (no Pinecone search needed for some)
- Uses verbatim sacred vocabulary ("builder," "modernizer," "complexity to clarity")
- Broad thematic response, not story-specific details

### 6. Edge Cases (Nonsense, Out-of-Scope)

Tests query validation and graceful rejection.

**Example Queries:**
- "What's the weather in New York?"
- "Tell me about Matt's experience in aerospace" (out-of-scope industry)
- "asdfghjkl" (gibberish)

**Expected Behavior:**
- Semantic router rejects (score <0.40)
- Graceful off-domain response with suggestion chips
- No attempt to fabricate answers

---

## Evaluation Criteria

### Source Relevance

**Question:** Did the RAG pipeline return the correct stories?

**Validation:**
- Expected story IDs match actual results
- Ranking order is appropriate (pinned stories at #1)
- No irrelevant stories in top-k results

### Entity Pinning Accuracy

**Question:** When a query mentions a specific entity, does the matching story get pinned to #1?

**Validation:**
- Client query → Client-matching story at position #1
- Employer query → Employer-matching story at position #1
- Division query → Division-matching story at position #1

**Edge Cases:**
- Context exclusion prefixes ("after leaving Accenture" → don't pin Accenture stories)
- Generic client names ("Multiple Clients" → no pinning)

### Meta-Commentary Detection (Voice Quality)

**Question:** Does the response maintain Agy's voice (fact-relayer, not evaluator)?

**Banned Patterns:**
- "this demonstrates"
- "this reflects"
- "this showcases"
- "Matt's ability to"
- "his ability to"

**Why This Matters:**
Marketing and landing page queries fail HARD if meta-commentary slips through. "What's Matt's superpower?" should NOT return "This demonstrates Matt's ability to..." — it should say "Building from nothing" directly.

### Confidence Threshold Behavior

**Question:** Are confidence thresholds applied correctly?

**Validation:**
- HIGH confidence (≥0.25) → "Found X stories"
- LOW confidence (≥0.20) → "Relevance may be low" warning
- NONE (<0.20) → "No strong matches" with suggestions

---

## Running Evaluations

### Basic Usage

```bash
# Run full eval suite
python -m pytest tests/eval_rag_quality.py -v

# Run specific category
python -m pytest tests/eval_rag_quality.py -k "entity" -v

# Show detailed output
python -m pytest tests/eval_rag_quality.py -v -s
```

### Output Interpretation

**PASS:** Query returned expected results, no voice violations
**FAIL:** One or more criteria failed (see assertion details)
**FLAKY:** Passed this run but known to be LLM-variance sensitive

### Common Failure Modes

1. **Entity pinning miss:** Story with matching entity NOT at #1
   - Check: Is entity detection firing? (debug logs)
   - Check: Is Pinecone metadata filter working? (field casing)

2. **Meta-commentary detected:** Response contains banned phrases
   - Check: System prompt (is BANNED_PHRASES enforcement active?)
   - Check: LLM temperature (higher = more creative violations)

3. **Confidence threshold miss:** Expected HIGH, got LOW
   - Check: Pinecone similarity scores (top_score < 0.25?)
   - Check: Query embedding quality (is it finding semantically similar stories?)

---

## Eval-Driven Development

### How Evals Guided the January 2026 Pipeline Cleanup

**Problem Identified:** 8/60 queries failing due to Entity Gate false rejections

**Example Failures:**
- "What's Matt's professional identity?" → Rejected (no entity detected, low semantic score)
- "Is Matt a builder or a maintainer?" → Rejected (narrative query, no client/employer mention)

**Eval-Driven Solution:**
1. Ran eval suite → 52/60 pass rate (86.7%)
2. Analyzed failures → Entity Gate blocking narrative queries
3. Removed Entity Gate → 60/61 pass rate (98.1%)
4. Validated with regression suite → No new failures introduced

**Key Insight:** Without evals, the Entity Gate would have stayed in place, silently degrading UX for 13% of queries.

### Threshold Calibration Process

**Initial State:** SOFT_ACCEPT = 0.72 (from early semantic router tuning)

**Eval Findings:**
- 5 queries in 0.40-0.72 range (false rejections)
- "How does Matt handle stakeholders?" → 0.58 similarity, REJECTED
- "What's Matt's approach to team scaling?" → 0.63 similarity, REJECTED

**Solution:**
1. Lowered SOFT_ACCEPT to 0.40
2. Re-ran evals → 2 additional passes
3. Monitored borderline cases (0.40-0.50) for 2 weeks
4. Confirmed: No quality degradation, better recall

**Result:** 98.1% pass rate with fewer false rejections

### LLM Intent Classification Removal

**Before:** GPT-4o-mini classified query intent ($0.0001/query, 200ms latency)

**Eval Test:**
1. Disabled LLM intent classification
2. Relied on semantic router only
3. Ran full eval suite → 60/61 pass rate (unchanged)

**Conclusion:** LLM call was redundant. Semantic router (embedding-based) achieved same accuracy at 1/10th the cost and 1/5th the latency.

---

## Adding New Test Cases

### Test Case Format

```python
{
    "query": "Show me Accenture projects",
    "expected_intent": "entity_detection",
    "expected_entity": {
        "type": "Client",
        "value": "Accenture"
    },
    "expected_stories": ["story_id_1", "story_id_2"],
    "pinned_story": "story_id_1",  # Must be at position #1
    "min_confidence": "high",
    "allow_meta_commentary": False  # Strict for marketing queries
}
```

### When to Add a Test Case

**Required:**
- User reported a bug (query that failed in production)
- New feature needs validation (e.g., synthesis mode)
- Threshold change (validate edge cases)

**Optional but Recommended:**
- Common query patterns (top 10 from analytics)
- Regression-prone areas (entity pinning, narrative queries)

### Category Guidelines

| Category | Min Cases | Focus |
|----------|-----------|-------|
| Entity Detection | 15+ | Client, Employer, Division coverage |
| Behavioral | 10+ | Interview prep scenarios |
| Technical/Delivery | 10+ | Methodologies, outcomes, metrics |
| Marketing | 5+ | ZERO meta-commentary tolerance |
| Synthesis/Narrative | 5+ | Sacred vocabulary enforcement |
| Edge Cases | 10+ | Graceful failure modes |

---

## Metrics & Reporting

### Pass Rate Breakdown

```
Total Queries: 61
PASS: 60 (98.1%)
FAIL: 1 (1.6%)

By Category:
- Entity Detection: 16/16 (100%)
- Behavioral: 10/10 (100%)
- Technical/Delivery: 12/12 (100%)
- Marketing: 8/8 (100%)
- Synthesis/Narrative: 10/10 (100%)
- Edge Cases: 4/5 (80%)  ← Only failure: Q29 out-of-scope detection
```

### Flaky Test Handling

**LLM Variance:** GPT-4o responses vary slightly between runs (temperature = 0.4 for standard queries)

**Known Flaky Tests:**
- Q14: "Tell me about a time Matt failed" (rephrasing variance)
- Q27: "What's Matt's superpower?" (meta-commentary borderline cases)

**Strategy:**
- Run evals 3 times before declaring failure
- Track flaky test rate (should be <5%)
- Investigate if flakiness increases (sign of prompt instability)

### Quality Gates

**Pre-Merge Requirement:** Eval pass rate ≥ 95%
**Production Deploy:** Eval pass rate ≥ 98%

**Rationale:** 60 queries is small enough that 1-2 failures indicate real issues, not statistical noise.

---

## Future Enhancements

**Planned Improvements:**

1. **Automated Regression Detection:** Run evals on every commit (GitHub Actions)
2. **Latency Benchmarks:** Track P50/P95 response times per category
3. **Cost Tracking:** Log LLM token usage per query (optimize expensive patterns)
4. **Eval Coverage Gaps:** Add "tell me more" follow-up queries (currently missing)
5. **A/B Testing Framework:** Compare threshold changes with statistical significance

**Phase 2 Integration:** Eval suite will port directly to React/FastAPI architecture (framework-agnostic pytest tests).

---

## Key Takeaways

1. **Eval-driven development prevents regressions:** Entity Gate removal validated through evals
2. **Threshold tuning requires data:** 0.72 → 0.40 was guided by query score analysis
3. **Voice quality is measurable:** Meta-commentary detection is binary (no subjective "feels right")
4. **Small test suites catch big issues:** 60 queries found 13% failure rate from Entity Gate

**The Bottom Line:**

> Without systematic evaluation, RAG quality is guesswork. With evals, architectural changes become low-risk, data-driven decisions.

---

**Related Documentation:**
- [Technical Architecture](/mattgpt-design-spec/docs/02-technical-architecture) - RAG pipeline details
- [Building MattGPT](/mattgpt-design-spec/docs/04-building-mattgpt) - Development journey
- [API Reference](/mattgpt-design-spec/docs/09-api-reference) - Function signatures and parameters

---

*Last Updated: January 30, 2026*
*Version: 1.0 (Initial Documentation)*
