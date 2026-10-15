# Testing & Quality Assurance

**How MattGPT maintains quality through 3-layer testing strategy**

> This document describes the complete testing approach: unit tests for core components, RAG evaluation framework for pipeline quality ({{ site.data.facts.eval_query_count }} golden queries), and BDD/E2E tests for UI workflows ({{ site.data.facts.bdd_summary }}).

---

## Table of Contents

1. [Overview](#overview)
2. [Unit Tests](#unit-tests)
3. [RAG Quality Evaluation](#rag-quality-evaluation)
4. [BDD/E2E Tests](#bdde2e-tests)
5. [Running Tests](#running-tests)
6. [Quality Gates](#quality-gates)

---

## Overview

**3-Layer Testing Strategy:**

<div class="mermaid">
graph TD
    L3["Layer 3: BDD/E2E Tests<br/>{{ site.data.facts.bdd_summary }}<br/>Full UI workflows"]
    L2["Layer 2: RAG Eval<br/>{{ site.data.facts.eval_query_count }} golden queries · run by hand<br/>Eval-driven development"]
    L1["Layer 1: Unit Tests<br/>{{ site.data.facts.unit_test_file_count }} unit test files · Component isolation<br/>Fast feedback (under 1 min)"]
    L3 --> L2 --> L1
</div>

**Quality Metrics:**
- **Unit Test Coverage:** {{ site.data.facts.unit_test_file_count }} unit test files testing core components
- **RAG Eval:** {{ site.data.facts.eval_query_count }} golden queries; no run yet on the October 2026 answer pipeline (MATTGPT-275)
- **BDD/E2E:** {{ site.data.facts.bdd_summary }}

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
├── test_ask_agy_tool_answer.py         # Tool loop, streaming, broken streams, error messages
├── test_ask_agy_tool_system_prompt.py  # The tool-path system prompt
├── test_ask_agy_history.py             # Conversation history sent to the model
├── test_ask_agy_post_processing.py     # Post-processing (bold spans and numbers)
├── test_ask_agy_pending_turn.py        # The conversation view's pending-turn path
├── test_no_dash_rule.py                # NO_DASH_RULE in the prompts
├── test_comparison_rule.py             # COMPARISON_RULE in the prompts
├── test_nonsense_portfolio_questions.py # Nonsense rules vs real portfolio questions
├── test_backend_service.py             # send_to_backend(), off-domain logging
├── test_rag_service.py                 # semantic_search()
├── test_semantic_router.py             # Intent classification
├── test_filters.py                     # Filter logic (Client, Theme, etc.)
├── test_scoring.py                     # Hybrid keyword scoring
├── test_validation.py                  # Input validation, nonsense detection
├── test_formatting.py                  # Story formatting and 5P summaries
├── test_story_intelligence.py          # Theme inference, story context (including dates)
└── ... (see `tests/unit/` for the full list)
```

### What Unit Tests Cover

**Component Isolation:**
- Ask Agy's tool loop: the required first search, stream retry before any text, error messages
- The answer prompt: tool system prompt, dash and comparison rules
- Post-processing of the answer (bolding, dashes)
- `semantic_search()` and hybrid keyword scoring
- Semantic router intent classification
- Filter logic
- Input validation and the nonsense rules

**Example Test** (`tests/unit/test_semantic_router.py`, `TestClassifyEmbeddingThreshold`):

```python
def test_is_valid_false_when_score_below_soft_threshold(self):
    # Orthogonal vectors -> similarity 0.0, below any positive threshold.
    query = _v(1.0, 0.0)
    anchors = {"intent_a": _v(0.0, 1.0)}
    is_valid, score, _, _ = _classify_embedding(query, anchors)
    assert score < SOFT_ACCEPT
    assert is_valid is False
```

**Running Unit Tests:**

```bash
pytest tests/unit/ -v
```

---

## RAG Quality Evaluation

**Location:** `tests/eval_rag_quality.py`
**Framework:** pytest-based golden query testing
**Coverage:** {{ site.data.facts.eval_query_count }} queries across the categories below
**Under test:** `agy_answer()`, through the `rag_fn` fixture, which calls it in `rag_answer()`'s old `(query, filters, stories)` shape and ignores `filters`
**Pass rate:** measured per run, not recorded in this spec. No golden eval has run on the shipped October 2026 pipeline yet; the last run (October 14, 2026) was on code from before the final MATTGPT-275 commit. Q59 ("Where does Matt live?") still carries an `xfail_reason` that cites the retired confidence gate; that run answered it.

### Why RAG Evals Matter

RAG evals check the final answer and its sources end to end, not just component behavior. They catch:
- **Missing substance:** answers without the expected concepts (ground-truth stems)
- **Client attribution:** a named client missing from the answer, or not bolded
- **Thin synthesis:** synthesis answers drawn from too few stories or named clients
- **Voice regressions:** meta-commentary creeping back in
- **Architecture changes:** validate major refactors (e.g., Entity Gate removal, the MATTGPT-275 tool loop)

An answer that comes back `degraded` (the error message) fails.

---

### Query Categories

Each category targets a distinct failure mode; the query counts are in `GOLDEN_QUERIES` in `tests/eval_rag_quality.py`.

### 1. Narrative

Professional identity, career story, "builder" vocabulary, and synthesis questions.

**Example Queries:**
- "Is Matt a builder or a maintainer?"
- "What's Matt's professional narrative?"
- "Summarize Matt's career themes"

**Expected Behavior:**
- Reflects Matt's identity terms; the eval checks concept stems such as "build" and "moderniz" against a minimum match count, not exact words
- Broad thematic response, not story-specific details
- No meta-commentary

### 2. Client

Named-client questions.

**Example Queries:**
- "What did Matt do at JP Morgan?"
- "Show me RBC work"

**Expected Behavior:**
- The client is named in the answer (`client_variants` accepted) and bolded (`check_client_attribution()`)

### 3. Intent

Behavioral, synthesis, redirect, blocked and profile-fact questions; `expected_behavior` on each query decides the check.

**Example Queries:**
- "Tell me about a time Matt failed"
- "How does Matt handle conflict?"
- "Describe a difficult stakeholder situation"

**Expected Behavior:**
- Synthesis: enough named clients in the answer, and enough source stories and named-client sources
- Redirect: the redirect copy; blocked: no answer
- Profile fact: the answer contains the expected words (`must_contain`)
- Other behavioral queries pass when an answer comes back

### 4. Edge

Thin themes, a multi-turn follow-up, and a synthesis question with a named client.

**Example Queries:**
- "Matt's GenAI work"
- "How did Matt transform delivery at JP Morgan?"

**Expected Behavior:**
- Synthesis with a client: the client is named and bolded
- The others pass when an answer comes back; the multi-turn case is noted, not checked

Garbage input is not tested here: the nonsense rules (including `gibberish` and `local_recommendations`) are tested in `tests/test_structural_assertions.py`.

### 5. Surgical

Delivery and technical depth questions requiring precise story retrieval.

**Example Queries:**
- "How did Matt achieve 4x faster delivery?"
- "Show me platform engineering work"
- "What agile transformation experience does Matt have?"

**Expected Behavior:**
- Ground-truth stems and, where specified, client attribution

### 6. Entity Detection

Employer and Division questions. Since MATTGPT-275 removed `detect_entity()`, this category is skipped and always passes; the queries are kept as text.

**Example Queries:**
- "Show me Accenture projects"
- "Tell me about Cloud Innovation Center work"

**Expected Behavior:** none checked (skipped)

### 7. Marketing

Landing page queries requiring zero meta-commentary.

**Example Queries:**
- "What's Matt's superpower?"
- "What makes Matt different?"
- "Why should I hire Matt?"

**Expected Behavior:**
- NO meta-commentary (hardest constraint, zero tolerance)
- Short, direct responses

### 8. Context Story

"Tell me more about: [Title]" queries, built at runtime from `story_index`.

**Expected Behavior:**
- An answer comes back with sources
- The named story is among the sources (found by the model's own searches)

---

### Evaluation Criteria

### Ground Truth and Attribution

**Question:** Does the answer carry the expected substance and name the right clients?

**Validation:**
- Ground-truth concept stems, against a minimum match count (`check_ground_truth()`)
- Client named and bolded (`check_client_attribution()`)
- Synthesis breadth: named clients in the answer, source stories and named-client sources

The eval does not check story ids or ranking.

### Meta-Commentary Detection (Voice Quality)

**Question:** Does the response maintain Agy's voice (fact-relayer, not evaluator)?

**Banned Patterns** (`META_COMMENTARY_PATTERNS` in `config/constants.py`, sample):
- "this demonstrates"
- "this reflects"
- "this showcases"
- "Matt's ability to"
- "his ability to"

Marketing and landing page queries fail HARD if meta-commentary slips through. "What's Matt's superpower?" should NOT return "This demonstrates Matt's ability to..."; it should say "Building from nothing" directly.

### Confidence and Ask Agy

Ask Agy has no confidence gate, so the eval checks no confidence tier. My Work still shows results at LOW confidence (≥{{ site.data.facts.confidence_low }}) with a "Relevance may be low" note (`explore_stories.py`); the eval does not cover My Work.

---

### Running RAG Evaluations

### Basic Usage

```bash
# Run full eval suite
python -m pytest tests/eval_rag_quality.py -v

# Run specific category
python -m pytest tests/eval_rag_quality.py -k "client" -v

# Show detailed output
python -m pytest tests/eval_rag_quality.py -v -s
```

### Output Interpretation

**PASS:** Query returned expected results, no voice violations
**FAIL:** One or more criteria failed (see assertion details)
**FLAKY:** Passed this run but known to be LLM-variance sensitive

### Common Failure Modes

1. **Thin synthesis:** too few source stories or named clients
   - Check: the model's `search_stories` queries, and what each search returned

2. **Meta-commentary detected:** response contains a `META_COMMENTARY_PATTERNS` phrase
   - Check: the system prompt's voice rules in `prompts.py`

3. **Missing client or ground truth:** the answer leaves out the client or the expected concepts
   - Check: whether the searches returned the story at all (the source cards show what was retrieved)

4. **Degraded answer:** the error message came back
   - Check: the `agy_answer fatal error` log line

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

A real entry from `GOLDEN_QUERIES`:

```python
{
    "id": 12,
    "query": "Matt's modernization work at RBC",
    "expected_client": "RBC",
    "client_variants": ["RBC", "Royal Bank"],
    "category": "client",
}
```

Other keys in use include `expected_behavior`, `ground_truth`, `min_matches`, `min_clients`, `min_source_stories`, `min_source_named_clients`, `must_contain`, `must_not_contain_meta`, `story_index` and `xfail_reason`.

### When to Add a Test Case

**Required:**
- User reported a bug (query that failed in production)
- New feature needs validation (e.g., synthesis mode)
- Threshold change (validate edge cases)

**Optional but Recommended:**
- Common query patterns (top 10 from analytics)
- Regression-prone areas (synthesis breadth, narrative queries)

### Metrics & Reporting

### Pass Rate Breakdown

The eval suite covers the categories above: narrative, client, intent, edge, surgical, entity detection (skipped), marketing, and context story. See Coverage above for Q59.

### Flaky Test Handling

**LLM Variance:** answers vary between runs: the answer model (`ASK_ANSWER_MODEL`, no temperature setting) writes its own search queries, so both what it retrieves and what it writes can change

**Known Flaky Tests:** none tracked by query. Occasional failures from LLM variance usually pass on re-run (Known Limitations in the app repo's `ARCHITECTURE.md`).

**Strategy:**
- Run evals 3 times before declaring failure
- Track flaky test rate (should be <5%)
- Investigate if flakiness increases (sign of prompt instability)

---

## BDD/E2E Tests

**Location:** `tests/bdd/`
**Framework:** pytest-bdd + Playwright
```bash
pytest tests/bdd -v
```

**Coverage ({{ site.data.facts.bdd_summary }}):** Search flow, filter combinations, view switching, story detail, Ask Agy navigation, deeplinks, pagination, navigation/reset, Role Match, responsive layout, Ask Agy's streamed answer and cross-page hand-off, edge cases, and more.

**Status:** Red → Green → Refactor by convention: scenarios are usually written and committed before implementation code. Exceptions are recorded in the commit (MATTGPT-275's hand-off scenarios were written during its Green).

**Rationale:** One doc, one truth. The old pyramid framework is conceptual fluff that doesn't reflect reality.

---

## Running Tests

### Full Test Suite

```bash
# Run all tests (unit + RAG evals + E2E)
pytest tests/ -v
```

### Individual Test Layers

```bash
# Unit tests only (<1 minute)
pytest tests/unit/ -v

# RAG evaluations only (one answer-model call per query; the slowest layer per test)
pytest tests/eval_rag_quality.py -v

# E2E tests only (needs a running app)
pytest tests/bdd/ -v
```

### CI/CD Integration

Three automation layers are active: push to main triggers a Streamlit Cloud deploy (real continuous deployment, every merge ships immediately), a GitHub Actions cron job (~10 min schedule) keeps the app warm, and a push-triggered GitHub Actions job derives volatile facts from the source code and syncs them to `_data/facts.yml` in the design-spec repo.

The unit suite is the one automated test gate, enforced twice:

- **CI:** `.github/workflows/test.yml` runs `pytest tests/unit/` on every push and pull request to main. It needs no API keys, so it can't be skipped for lack of secrets.
- **Pre-push hook:** `.pre-commit-config.yaml` runs the unit and integration tests (excluding network-bound ones) before each `git push`.

The RAG eval and the BDD suite are not automated. The eval costs API tokens and takes minutes, and the BDD suite needs a running app, so both are run by hand.

---

## Quality Gates

Unit tests are the one enforced gate: CI and the pre-push hook run them. BDD scenarios are run by hand before merge. The golden eval is run by hand on pipeline changes; no pass-rate threshold is recorded or enforced, and MATTGPT-275 merged without a golden run on its final code.

---

**Related Documentation:**
- [Technical Architecture](02-technical-architecture) - RAG pipeline details
- [Building MattGPT](04-building-mattgpt) - Development journey
- [API Reference](09-api-reference) - Function signatures and parameters

---

*Last updated: {{ site.data.page_dates['11-testing-and-quality'] }}*
