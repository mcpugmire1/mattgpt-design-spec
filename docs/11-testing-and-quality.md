# Testing & Quality Assurance

**How MattGPT maintains quality through 3-layer testing strategy**

> This document describes the complete testing approach: unit tests for core components, RAG evaluation framework for pipeline quality (98.1% pass rate), and BDD/E2E tests for UI workflows (43 scenarios).

---

## Table of Contents

1. [Overview](#overview)
2. [Unit Tests](#unit-tests)
3. [RAG Quality Evaluation](#rag-quality-evaluation)
4. [BDD/E2E Tests](#bdde2e-tests-explore-stories)
5. [Running Tests](#running-tests)
6. [Quality Gates](#quality-gates)

---

## Overview

**3-Layer Testing Strategy:**

```
┌─────────────────────────────────────────┐
│ Layer 3: BDD/E2E Tests                  │
│ - 43 Playwright scenarios               │
│ - Full UI workflows                     │
│ - ~25 minute runtime                    │
├─────────────────────────────────────────┤
│ Layer 2: RAG Quality Evaluation         │
│ - 60+ golden queries                    │
│ - 98.1% pass rate (60/61)               │
│ - Eval-driven development               │
├─────────────────────────────────────────┤
│ Layer 1: Unit Tests                     │
│ - 12 test files                         │
│ - Component isolation                   │
│ - Fast feedback (<1 min)                │
└─────────────────────────────────────────┘
```

**Quality Metrics:**
- **Unit Test Coverage:** 12 files testing core components
- **RAG Eval Pass Rate:** 98.1% (60/61 queries)
- **E2E Test Pass Rate:** 100% (43/43 scenarios)
- **Total Test Runtime:** ~30 minutes (full suite)

**Why This Matters:**

RAG systems are notoriously hard to test. LLM outputs are non-deterministic, semantic similarity is fuzzy, and "good enough" is subjective. Without systematic testing, pipeline changes become high-risk guesswork.

MattGPT's testing strategy provides:
- **Fast feedback:** Unit tests catch component issues immediately
- **Regression prevention:** RAG evals catch breaking changes before deployment
- **User experience validation:** E2E tests verify complete workflows
- **Confidence in refactoring:** Architecture changes backed by data

---

## Unit Tests

**Location:** `tests/unit/`
**Framework:** pytest
**Runtime:** <1 minute

### Test Files (12 total)

```
tests/unit/
├── test_rag_service.py           # RAG pipeline orchestration
├── test_backend_service.py       # Story loading, filtering
├── test_semantic_router.py       # Intent classification
├── test_filters.py               # Filter logic (Client, Theme, etc.)
├── test_scoring.py               # Confidence scoring, ranking
├── test_validation.py            # Input validation, nonsense detection
├── test_formatting.py            # Response formatting, source citations
├── test_story_intelligence.py    # Story analysis, metadata extraction
└── ... (4 more files)
```

### What Unit Tests Cover

**Component Isolation:**
- RAG service methods (search, ranking, context assembly)
- Semantic router intent detection
- Filter logic (exact matches, multi-field entity search)
- Confidence scoring calculations
- Input validation (pattern matching, empty queries)
- Response formatting (XML isolation, source extraction)

**Example Test:**

```python
def test_semantic_router_intent_detection():
    """Verify semantic router correctly classifies query intents"""
    test_cases = [
        ("Show me JP Morgan projects", "entity_detection"),
        ("How did Matt scale teams?", "behavioral"),
        ("Tell me about platform engineering", "technical"),
        ("What's Matt's story?", "narrative"),
    ]

    for query, expected_intent in test_cases:
        result = semantic_router.classify(query)
        assert result["intent"] == expected_intent
        assert result["score"] >= 0.40  # SOFT_ACCEPT threshold
```

**Running Unit Tests:**

```bash
pytest tests/unit/ -v
```

---

## RAG Quality Evaluation

**Location:** `tests/eval_rag_quality.py`
**Framework:** pytest-based golden query testing
**Coverage:** 60+ queries across 6 categories
**Current Pass Rate:** 98.1% (60/61)

### Why RAG Evals Matter

RAG evals validate end-to-end pipeline quality, not just component behavior. They catch:
- **Threshold calibration issues:** Queries that should pass but don't (or vice versa)
- **Entity pinning failures:** Named entities not ranked #1
- **Voice regressions:** Meta-commentary creeping back in
- **Architecture changes:** Validate major refactors (e.g., Entity Gate removal)

**Quality Gates:**
- **Pre-Merge Requirement:** Eval pass rate ≥ 95%
- **Production Deploy:** Eval pass rate ≥ 98%

---

### Query Categories

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

### Evaluation Criteria

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

### Running RAG Evaluations

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

### Eval-Driven Development

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

### Adding New Test Cases

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

### Metrics & Reporting

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

### Future Enhancements

**Planned Improvements:**

1. **Automated Regression Detection:** Run evals on every commit (GitHub Actions)
2. **Latency Benchmarks:** Track P50/P95 response times per category
3. **Cost Tracking:** Log LLM token usage per query (optimize expensive patterns)
4. **Eval Coverage Gaps:** Add "tell me more" follow-up queries (currently missing)
5. **A/B Testing Framework:** Compare threshold changes with statistical significance

**Migration note:** Eval suite is framework-agnostic (pure pytest) and would port directly to a React/FastAPI architecture if needed.

---

## BDD/E2E Tests (Explore Stories)

**Location:** `tests/bdd/`
**Framework:** pytest-bdd + Playwright
**Runtime:** ~25 minutes

```bash
pytest tests/bdd -v
```

**Coverage (43 scenarios):** Search flow, filter combinations, view switching, story detail, Ask Agy navigation, deeplinks, pagination, navigation/reset, responsive layout, edge cases.

**Status:** 43 passed, 0 skipped

**Rationale:** One doc, one truth. The old pyramid framework is conceptual fluff that doesn't reflect reality.

---

## Running Tests

### Full Test Suite

```bash
# Run all tests (unit + RAG evals + E2E)
pytest tests/ -v

# Runtime: ~30 minutes
```

### Individual Test Layers

```bash
# Unit tests only (<1 minute)
pytest tests/unit/ -v

# RAG evaluations only (~3 minutes)
pytest tests/eval_rag_quality.py -v

# E2E tests only (~25 minutes)
pytest tests/bdd/ -v
```

### CI/CD Integration

```yaml
# .github/workflows/test.yml (example)
- name: Run Unit Tests
  run: pytest tests/unit/ -v

- name: Run RAG Evaluations
  run: pytest tests/eval_rag_quality.py -v

- name: Run E2E Tests
  run: pytest tests/bdd/ -v
```

---

## Quality Gates

**Pre-Merge Requirements:**
- ✅ All unit tests passing (12/12)
- ✅ RAG eval pass rate ≥ 95% (57+/61)
- ✅ E2E tests passing (43/43)

**Production Deploy Requirements:**
- ✅ All unit tests passing (12/12)
- ✅ RAG eval pass rate ≥ 98% (60+/61)
- ✅ E2E tests passing (43/43)
- ✅ Manual smoke test (5 queries + 2 UI workflows)

**Rationale:** 60 queries is small enough that 1-2 failures indicate real issues, not statistical noise. E2E tests validate complete user workflows work end-to-end.

---

## Key Takeaways

1. **3-layer strategy catches everything:** Unit tests (components), RAG evals (pipeline), E2E (workflows)
2. **Fast feedback loop:** Unit tests run in <1 minute, catch issues immediately
3. **Eval-driven development prevents regressions:** Entity Gate removal validated through evals
4. **Threshold tuning requires data:** 0.72 → 0.40 was guided by query score analysis
5. **Voice quality is measurable:** Meta-commentary detection is binary (no subjective "feels right")
6. **E2E validates reality:** 43 scenarios ensure complete user workflows work end-to-end

**The Bottom Line:**

> Without systematic testing, quality is guesswork. With 3-layer testing, architectural changes become low-risk, data-driven decisions backed by 100+ test cases.

---

**Related Documentation:**
- [Technical Architecture](/mattgpt-design-spec/docs/02-technical-architecture) - RAG pipeline details
- [Building MattGPT](/mattgpt-design-spec/docs/04-building-mattgpt) - Development journey
- [API Reference](/mattgpt-design-spec/docs/09-api-reference) - Function signatures and parameters

---

*Last Updated: January 30, 2026*
*Version: 1.0 (Initial Documentation)*
