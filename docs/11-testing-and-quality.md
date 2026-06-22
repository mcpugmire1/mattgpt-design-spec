# Testing & Quality Assurance

**How MattGPT maintains quality through 3-layer testing strategy**

> This document describes the complete testing approach: unit tests for core components, RAG evaluation framework for pipeline quality (100% pass rate, 64/64), and BDD/E2E tests for UI workflows ({{ site.data.facts.bdd_summary }}).

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

<div class="mermaid">
graph TD
    L3["Layer 3: BDD/E2E Tests<br/>{{ site.data.facts.bdd_summary }}<br/>Full UI workflows"]
    L2["Layer 2: RAG Eval<br/>{{ site.data.facts.eval_query_count }} golden queries · {{ site.data.facts.eval_summary }} pass rate<br/>Eval-driven development"]
    L1["Layer 1: Unit Tests<br/>{{ site.data.facts.unit_test_file_count }} unit test files · Component isolation<br/>Fast feedback (under 1 min)"]
    L3 --> L2 --> L1
</div>

**Quality Metrics:**
- **Unit Test Coverage:** {{ site.data.facts.unit_test_file_count }} unit test files testing core components
- **RAG Eval Pass Rate:** 100% (64/64 queries)
- **BDD/E2E Pass Rate:** {{ site.data.facts.bdd_summary }}
- **Total Test Runtime:** ~30 minutes (full suite)

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

### Test Files ({{ site.data.facts.unit_test_file_count }} total)

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
└── ... (12 more files)
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
        ("Show me Matt's platform engineering work", "technical"),
        ("How did Matt scale teams?", "behavioral"),
        ("Tell me about platform engineering", "technical"),
        ("What's Matt's story?", "narrative"),
    ]

    for query, expected_intent in test_cases:
        is_valid, score, intent, family = is_portfolio_query_semantic(query)
        assert intent == expected_intent
        assert score >= 0.40  # SOFT_ACCEPT threshold
```

**Running Unit Tests:**

```bash
pytest tests/unit/ -v
```

---

## RAG Quality Evaluation

**Location:** `tests/eval_rag_quality.py`
**Framework:** pytest-based golden query testing
**Coverage:** 64 queries across 8 categories
**Current Pass Rate:** 100% (64/64)

### Why RAG Evals Matter

RAG evals validate end-to-end pipeline quality, not just component behavior. They catch:
- **Threshold calibration issues:** Queries that should pass but don't (or vice versa)
- **Entity pinning failures:** Named entities not ranked #1
- **Voice regressions:** Meta-commentary creeping back in
- **Architecture changes:** Validate major refactors (e.g., Entity Gate removal)

**Quality Gates:**
- **Pre-Merge Requirement:** Eval pass rate ≥ 95% (61+/64)
- **Production Deploy:** Eval pass rate ≥ 98% (63+/64)

---

### Query Categories

The 8 categories below match the Pass Rate Breakdown. Each targets a distinct failure mode.

### 1. Narrative (13 queries)

Professional identity, career story, "builder" vocabulary, and synthesis questions.

**Example Queries:**
- "Is Matt a builder or a maintainer?"
- "What's Matt's professional narrative?"
- "Summarize Matt's career themes"

**Expected Behavior:**
- Uses verbatim "builder" vocabulary (sacred term as of March 2026)
- Broad thematic response, not story-specific details
- No meta-commentary

### 2. Client (6 queries)

Named client entity detection and story pinning.

**Example Queries:**
- "What did Matt do at JP Morgan?"
- "Show me RBC work"

**Expected Behavior:**
- Correct Client entity detected and pinned to #1
- High confidence score (≥0.25)

### 3. Intent (9 queries)

Behavioral and stakeholder questions requiring STAR-formatted responses.

**Example Queries:**
- "Tell me about a time Matt failed"
- "How does Matt handle conflict?"
- "Describe a difficult stakeholder situation"

**Expected Behavior:**
- Returns stories with strong behavioral examples
- No meta-commentary
- Preserves STAR structure

### 4. Edge (4 queries)

Nonsense detection and graceful out-of-scope rejection.

**Example Queries:**
- "What's the weather in New York?"
- "asdfghjkl"

**Expected Behavior:**
- Semantic router rejects (score < 0.40)
- Graceful off-domain response with suggestion chips

### 5. Surgical (11 queries)

Delivery and technical depth questions requiring precise story retrieval.

**Example Queries:**
- "How did Matt achieve 4x faster delivery?"
- "Show me platform engineering work"
- "What agile transformation experience does Matt have?"

**Expected Behavior:**
- Returns stories with technical details or delivery metrics
- Confidence-based filtering for vague queries

### 6. Entity Detection (11 queries)

Employer and Division entity pinning (complements Client category).

**Example Queries:**
- "Show me Accenture projects"
- "Tell me about Cloud Innovation Center work"

**Expected Behavior:**
- Correct Employer or Division entity detected and pinned to #1
- High confidence score (≥0.25)

### 7. Marketing (7 queries)

Landing page queries requiring zero meta-commentary.

**Example Queries:**
- "What's Matt's superpower?"
- "What makes Matt different?"
- "Why should I hire Matt?"

**Expected Behavior:**
- NO meta-commentary (hardest constraint, zero tolerance)
- Short, direct responses

### 8. Context Story (3 queries)

"Tell me more about [Title]" queries: tests Title soft-filter detection and retrieval.

**Expected Behavior:**
- Story retrieved by Title detection (soft-filter, not entity gate)
- Sources returned and not blocked
- Response references the named story

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

Marketing and landing page queries fail HARD if meta-commentary slips through. "What's Matt's superpower?" should NOT return "This demonstrates Matt's ability to..."; it should say "Building from nothing" directly.

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

**Problem Identified:** 8/60 queries failing due to Entity Gate false rejections (January 2026)

**Example Failures:**
- "What's Matt's professional identity?" → Rejected (no entity detected, low semantic score)
- "Is Matt a builder or a maintainer?" → Rejected (narrative query, no client/employer mention)

**Eval-Driven Solution:**
1. Ran eval suite → 52/60 pass rate (86.7%)
2. Analyzed failures → Entity Gate blocking narrative queries
3. Removed Entity Gate → 60/61 pass rate, then expanded to 64/64 (100%) through suite growth
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

**Result:** Fewer false rejections with no quality degradation

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
    "expected_intent": "innovation",
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

### Metrics & Reporting

### Pass Rate Breakdown

```
Total Queries: 64
PASS: 64 (100%)
FAIL: 0

By Category:
- Narrative:        13/13 (100%)
- Client:            6/6  (100%)
- Intent:            9/9  (100%)
- Edge:              4/4  (100%)
- Surgical:         11/11 (100%)
- Entity Detection: 11/11 (100%)
- Marketing:         7/7  (100%)
- Context Story:     3/3  (100%)
```

### Flaky Test Handling

**LLM Variance:** GPT-4o responses vary slightly between runs (temperature = 0.4 for standard queries, 0.2 for synthesis)

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

**Rationale:** 64 queries is small enough that 1-2 failures indicate real issues, not statistical noise.

---

## BDD/E2E Tests (Explore Stories)

**Location:** `tests/bdd/`
**Framework:** pytest-bdd + Playwright
**Runtime:** ~25 minutes

```bash
pytest tests/bdd -v
```

**Coverage ({{ site.data.facts.bdd_summary }}):** Search flow, filter combinations, view switching, story detail, Ask Agy navigation, deeplinks, pagination, navigation/reset, Role Match, responsive layout, edge cases, and more.

**Status:** Red → Green → Refactor cycle enforced per feature. Scenarios written and committed before implementation code.

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

Three automation layers are active: push to main triggers a Streamlit Cloud deploy (real continuous deployment, every merge ships immediately), a GitHub Actions cron job (~10 min schedule) keeps the app warm, and a scheduled GitHub Actions job keeps `_data/facts.yml` in sync with the running code.

Test gates are not currently enforced by the pipeline. They run manually before merge and deploy (see Quality Gates below). If automated enforcement were added, the workflow would look like this:

```yaml
# .github/workflows/test.yml
- name: Run Unit Tests
  run: pytest tests/unit/ -v

- name: Run RAG Evaluations
  run: pytest tests/eval_rag_quality.py -v

- name: Run E2E Tests
  run: pytest tests/bdd/ -v
```

---

## Quality Gates

These gates run manually before merge and before production deploy. They are conventions, not automated checks.

**Pre-Merge:**
- All {{ site.data.facts.unit_test_file_count }} unit test files passing
- RAG eval pass rate ≥ 95% (61+/64)
- BDD scenarios passing

**Production Deploy:**
- All {{ site.data.facts.unit_test_file_count }} unit test files passing
- RAG eval pass rate ≥ 98% (63+/64)
- BDD scenarios passing
- Manual smoke test (5 queries + 2 UI workflows)

**Rationale:** 64 queries is small enough that 1-2 failures indicate real issues, not statistical noise. BDD scenarios validate complete user workflows end-to-end.

---

**Related Documentation:**
- [Technical Architecture](02-technical-architecture) - RAG pipeline details
- [Building MattGPT](04-building-mattgpt) - Development journey
- [API Reference](09-api-reference) - Function signatures and parameters

---

*Last updated: {{ site.data.page_dates['11-testing-and-quality'] }}*
