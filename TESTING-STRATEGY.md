# Testing Strategy for MattGPT / Ask Agy

**Purpose:** Ensure the AI assistant (Agy) behaves according to the system prompt, returns accurate results, and doesn't regress when changes are made.

**Last Updated:** October 17, 2024

---

## The Testing Challenge

**Problem:** AI systems are non-deterministic. The same query can produce different responses, making traditional unit testing difficult.

**Goals:**
1. ✅ Verify Agy follows the system prompt directives
2. ✅ Ensure responses are grounded in source STAR stories (no hallucination)
3. ✅ Detect regression when prompt or RAG pipeline changes
4. ✅ Validate expected behavior for common query types

---

## Testing Pyramid for AI Systems

```
           /\
          /  \  Manual Exploratory Testing
         /____\  (Edge cases, user experience)
        /      \
       / E2E AI \ End-to-End AI Behavior Tests
      /  Tests  \ (Golden queries → expected responses)
     /__________\
    /            \
   /  Integration \ Integration Tests
  /     Tests     \ (RAG pipeline, retrieval accuracy)
 /________________\
/                  \
/   Unit Tests      \ Unit Tests
/____________________\ (Embedding, search, filtering)
```

---

## Layer 1: Unit Tests (Deterministic Components)

**What to Test:** Non-AI components that have predictable behavior

### Embedding Generation
```python
def test_embedding_generation():
    """Verify embedding model produces consistent vector dimensions"""
    query = "Show me agile transformation projects"
    embedding = generate_embedding(query)

    assert len(embedding) == 384  # all-MiniLM-L6-v2 dimension
    assert all(isinstance(x, float) for x in embedding)
    assert -1.0 <= max(embedding) <= 1.0  # Normalized
```

### Vector Search
```python
def test_vector_search_returns_top_k():
    """Verify Pinecone search returns correct number of results"""
    query_vector = [0.1, 0.2, ...]  # Mock embedding
    results = pinecone_search(query_vector, top_k=30)

    assert len(results) == 30
    assert all('id' in r and 'score' in r for r in results)
    assert results[0]['score'] >= results[-1]['score']  # Descending order
```

### Hybrid Search Weighting
```python
def test_hybrid_search_blending():
    """Verify 80/20 weighting is applied correctly"""
    semantic_scores = [0.9, 0.8, 0.7]
    keyword_scores = [0.5, 0.6, 0.4]

    blended = blend_scores(semantic_scores, keyword_scores, alpha=0.8)

    expected = [
        0.8 * 0.9 + 0.2 * 0.5,  # 0.82
        0.8 * 0.8 + 0.2 * 0.6,  # 0.76
        0.8 * 0.7 + 0.2 * 0.4,  # 0.64
    ]
    assert blended == pytest.approx(expected, rel=0.01)
```

### STAR Story Validation
```python
def test_star_story_schema():
    """Verify all stories have mandatory STAR fields"""
    stories = load_stories_from_jsonl("echo_star_stories.jsonl")

    for story in stories:
        assert 'situation' in story and story['situation']
        assert 'task' in story and story['task']
        assert 'action' in story and story['action']
        assert 'result' in story and story['result']
        assert 'metrics' in story and len(story['metrics']) > 0
```

---

## Layer 2: Integration Tests (RAG Pipeline)

**What to Test:** Components working together (retrieval + context augmentation)

### Retrieval Accuracy
```python
def test_retrieval_accuracy_for_known_queries():
    """Verify expected stories are retrieved for known queries"""

    test_cases = [
        {
            "query": "agile transformation at JPMorgan Chase",
            "expected_stories": ["agile-jpmc-2019", "agile-jpmc-2020"],
            "min_similarity": 0.7
        },
        {
            "query": "platform engineering",
            "expected_stories": ["platform-eng-2022", "cloud-native-2021"],
            "min_similarity": 0.6
        },
    ]

    for test in test_cases:
        results = hybrid_search(test["query"], top_k=10)
        result_ids = [r['id'] for r in results]

        # Check expected stories appear in top results
        for expected_id in test["expected_stories"]:
            assert expected_id in result_ids, f"{expected_id} not found for query '{test['query']}'"

        # Check similarity threshold
        top_score = results[0]['score']
        assert top_score >= test["min_similarity"]
```

### Context Augmentation
```python
def test_context_augmentation():
    """Verify retrieved stories are properly formatted for LLM context"""

    query = "Show me banking transformation work"
    retrieved_stories = hybrid_search(query, top_k=3)
    context = format_context_for_llm(retrieved_stories)

    # Verify STAR structure is preserved
    assert "Situation:" in context
    assert "Task:" in context
    assert "Action:" in context
    assert "Result:" in context

    # Verify metadata is included
    assert "Client:" in context or "Company:" in context
    assert "Outcome:" in context or "Impact:" in context

    # Verify source references
    for story in retrieved_stories:
        assert story['id'] in context or story['title'] in context
```

---

## Layer 3: E2E AI Behavior Tests (Golden Queries)

**What to Test:** Full AI responses for representative queries

### Approach: Golden Query Testing

**Concept:** Maintain a set of "golden queries" with expected behavior characteristics (not exact text, but structural expectations).

```python
class GoldenQueryTest:
    def __init__(self, query, expectations):
        self.query = query
        self.expectations = expectations

    def validate(self, response):
        """Check if response meets expectations"""
        results = []

        for expectation in self.expectations:
            result = expectation.check(response)
            results.append(result)

        return all(results)
```

### Test Case: Recruiter Vetting Query

```python
def test_recruiter_vetting_query():
    """Verify Agy responds appropriately to recruiter-style queries"""

    query = "Does Matt have experience scaling agile teams at banks?"
    response = ask_agy(query)

    # Expectation 1: Response anchors in specific projects
    assert_contains_project_reference(response)

    # Expectation 2: Response includes quantifiable metrics
    assert_contains_metrics(response, min_count=1)

    # Expectation 3: Response cites sources
    assert_contains_source_citations(response, min_count=1)

    # Expectation 4: Response mentions specific client
    assert_mentions_client(response, ["JPMorgan", "RBC", "Capital One"])

    # Expectation 5: Response is NOT generic career advice
    assert_not_generic(response)

    # Expectation 6: Response format is concise (recruiter persona)
    assert_word_count(response, max_words=200)
```

### Test Case: Technical Deep Dive Query

```python
def test_technical_deep_dive_query():
    """Verify Agy responds appropriately to technical assessment queries"""

    query = "How did Matt implement CI/CD pipelines at JPMorgan Chase?"
    response = ask_agy(query)

    # Expectation 1: Response includes STAR breakdown
    assert_contains_star_structure(response)

    # Expectation 2: Response includes technical details
    assert_contains_technical_terms(response, ["CI/CD", "pipeline", "automation"])

    # Expectation 3: Response cites specific project
    assert_contains_project_title(response)

    # Expectation 4: Response is detailed (hiring manager persona)
    assert_word_count(response, min_words=150, max_words=400)

    # Expectation 5: Response links to full story
    assert_contains_source_link(response)
```

### Test Case: Out-of-Scope Query

```python
def test_out_of_scope_query():
    """Verify Agy declines gracefully when asked about non-portfolio topics"""

    query = "What's the best way to learn Python?"
    response = ask_agy(query)

    # Expectation 1: Response acknowledges limitation
    assert_acknowledges_limitation(response)

    # Expectation 2: Response does NOT generate generic advice
    assert_not_generic_advice(response)

    # Expectation 3: Response suggests alternative (e.g., Ask about Matt's experience)
    assert_suggests_alternative(response)
```

---

## Layer 4: System Prompt Compliance Testing

**What to Test:** Verify AI follows non-negotiable guardrails from system prompt

### Reference: System Prompt Directives

From `/docs/agy-system-prompt.md`:

1. ✅ Anchor every answer in specific projects (citing Client, Title, Outcome)
2. ✅ Lead with outcomes, then methodology
3. ✅ Infer user intent (Interview Prep, Due Diligence, Pitch)
4. ❌ No generic career advice or philosophical answers
5. ❌ No corporate buzzword soup or robotic language
6. ❌ Never pretend to know things outside Matt's portfolio
7. ✅ All answers MUST be auditable (source references)

### Compliance Test Suite

```python
class SystemPromptComplianceTest:

    @staticmethod
    def test_directive_1_project_anchoring():
        """Every response must reference specific projects"""
        queries = [
            "Does Matt have agile experience?",
            "Tell me about Matt's leadership style",
            "What technologies does Matt know?"
        ]

        for query in queries:
            response = ask_agy(query)

            # Must cite at least one specific project
            assert has_project_reference(response), \
                f"Response to '{query}' did not cite a specific project"

    @staticmethod
    def test_directive_4_no_generic_advice():
        """System must not generate generic career advice"""
        out_of_scope_queries = [
            "How do I become a good leader?",
            "What's the best agile framework?",
            "Should I learn React or Angular?"
        ]

        for query in out_of_scope_queries:
            response = ask_agy(query)

            # Must decline or redirect, not answer generically
            assert not is_generic_advice(response), \
                f"Response to '{query}' provided generic advice instead of declining"

    @staticmethod
    def test_directive_5_no_buzzword_soup():
        """Responses must use concrete language, not buzzwords"""
        query = "Tell me about Matt's transformation leadership"
        response = ask_agy(query)

        # Check for excessive buzzwords
        buzzword_density = calculate_buzzword_density(response, [
            "synergy", "leverage", "strategic", "holistic", "paradigm",
            "disruptive", "innovative", "cutting-edge", "world-class"
        ])

        assert buzzword_density < 0.05, \
            f"Response has {buzzword_density:.1%} buzzword density (max 5%)"

    @staticmethod
    def test_directive_7_source_auditability():
        """Every response must include source citations"""
        queries = [
            "Show me Matt's banking work",
            "What did Matt do at JPMorgan?",
            "Tell me about platform engineering projects"
        ]

        for query in queries:
            response = ask_agy(query)
            sources = extract_source_citations(response)

            assert len(sources) > 0, \
                f"Response to '{query}' did not include source citations"

            # Verify sources are valid project IDs or titles
            for source in sources:
                assert is_valid_project_reference(source), \
                    f"Source '{source}' is not a valid project reference"
```

---

## Layer 5: Regression Testing

**Goal:** Detect when changes to system prompt or RAG pipeline break existing behavior

### Baseline Response Capture

**Approach:**
1. Run golden query test suite on current system
2. Capture responses and key characteristics (not exact text)
3. Store as baseline
4. Re-run after changes and compare

```python
class RegressionTestSuite:

    def __init__(self, baseline_file="tests/baselines.json"):
        self.baseline = load_baseline(baseline_file)

    def run_regression_tests(self):
        """Run all golden queries and compare to baseline"""

        results = []

        for test_case in self.baseline["golden_queries"]:
            query = test_case["query"]
            expected_characteristics = test_case["characteristics"]

            # Get current response
            response = ask_agy(query)
            actual_characteristics = extract_characteristics(response)

            # Compare
            comparison = compare_characteristics(
                expected_characteristics,
                actual_characteristics
            )

            results.append({
                "query": query,
                "passed": comparison["passed"],
                "differences": comparison["differences"]
            })

        return results
```

### Characteristics to Track (Not Exact Text)

```python
def extract_characteristics(response):
    """Extract structural characteristics from response"""
    return {
        "has_project_reference": has_project_reference(response),
        "has_metrics": has_metrics(response),
        "has_source_citations": has_source_citations(response),
        "word_count": count_words(response),
        "star_structure": has_star_structure(response),
        "mentioned_clients": extract_clients(response),
        "technical_terms": extract_technical_terms(response),
        "tone": analyze_tone(response),  # professional, warm, buzzwordy, etc.
    }
```

---

## Test Execution Strategy

### CI/CD Integration

```yaml
# .github/workflows/test-agy.yml
name: Test Ask Agy Behavior

on:
  push:
    paths:
      - 'src/agy_system_prompt.txt'
      - 'src/rag_pipeline.py'
      - 'data/echo_star_stories.jsonl'

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Run Unit Tests
        run: pytest tests/unit/

      - name: Run Integration Tests
        run: pytest tests/integration/

      - name: Run Golden Query Tests
        run: pytest tests/golden_queries/

      - name: Run System Prompt Compliance Tests
        run: pytest tests/compliance/

      - name: Run Regression Tests
        run: |
          pytest tests/regression/
          python scripts/compare_to_baseline.py

      - name: Generate Test Report
        if: always()
        run: |
          python scripts/generate_test_report.py
          cat test_report.md >> $GITHUB_STEP_SUMMARY
```

### Manual Testing Checklist

**Before deploying changes to system prompt or RAG pipeline:**

- [ ] Run full unit test suite (`pytest tests/unit/`)
- [ ] Run integration tests (`pytest tests/integration/`)
- [ ] Run golden query tests (`pytest tests/golden_queries/`)
- [ ] Run system prompt compliance tests (`pytest tests/compliance/`)
- [ ] Run regression test suite (`pytest tests/regression/`)
- [ ] Manual spot-check: Ask 5 random queries and verify responses
- [ ] Review test report for any failures or warnings
- [ ] If regressions detected: Investigate, fix, or document as intentional change

---

## Monitoring in Production

**Post-deployment monitoring:**

### Logging Key Metrics

```python
def ask_agy_with_logging(query, user_context=None):
    """Ask Agy with comprehensive logging"""

    start_time = time.time()

    # Retrieve relevant stories
    retrieved_stories = hybrid_search(query, top_k=30)
    retrieval_time = time.time() - start_time

    # Generate response
    response = generate_response(query, retrieved_stories)
    response_time = time.time() - start_time - retrieval_time

    # Extract characteristics
    characteristics = extract_characteristics(response)

    # Log
    log_query({
        "query": query,
        "user_context": user_context,
        "retrieval_time_ms": retrieval_time * 1000,
        "response_time_ms": response_time * 1000,
        "num_stories_retrieved": len(retrieved_stories),
        "top_similarity_score": retrieved_stories[0]['score'] if retrieved_stories else None,
        "has_project_reference": characteristics["has_project_reference"],
        "has_metrics": characteristics["has_metrics"],
        "has_source_citations": characteristics["has_source_citations"],
        "word_count": characteristics["word_count"],
        "timestamp": datetime.utcnow().isoformat()
    })

    return response
```

### Alerting on Anomalies

```python
# Alert if system prompt compliance drops below threshold
if compliance_rate < 0.95:  # 95% threshold
    send_alert(
        "System prompt compliance below threshold",
        f"Only {compliance_rate:.1%} of responses followed directives"
    )

# Alert if retrieval accuracy drops
if avg_similarity_score < 0.6:  # Minimum similarity threshold
    send_alert(
        "Low retrieval accuracy detected",
        f"Average similarity score: {avg_similarity_score:.2f}"
    )

# Alert if response time degrades
if avg_response_time_ms > 2000:  # 2 second threshold
    send_alert(
        "Slow response time detected",
        f"Average response time: {avg_response_time_ms:.0f}ms"
    )
```

---

## Test Data Management

### Golden Query Test Suite (JSON Format)

```json
{
  "version": "1.0",
  "updated": "2025-01-15",
  "golden_queries": [
    {
      "id": "gq-001",
      "query": "Does Matt have experience scaling agile teams at banks?",
      "persona": "recruiter_vetting",
      "characteristics": {
        "has_project_reference": true,
        "has_metrics": true,
        "has_source_citations": true,
        "mentioned_clients": ["JPMorgan", "RBC", "Capital One"],
        "min_word_count": 100,
        "max_word_count": 250,
        "must_not_be_generic": true
      }
    },
    {
      "id": "gq-002",
      "query": "How did Matt implement CI/CD pipelines?",
      "persona": "technical_assessment",
      "characteristics": {
        "has_star_structure": true,
        "has_technical_details": true,
        "mentioned_technologies": ["CI/CD", "pipeline", "automation"],
        "min_word_count": 200,
        "max_word_count": 500
      }
    },
    {
      "id": "gq-003",
      "query": "What's the best way to learn Python?",
      "persona": "out_of_scope",
      "characteristics": {
        "acknowledges_limitation": true,
        "does_not_provide_generic_advice": true,
        "suggests_alternative": true
      }
    }
  ]
}
```

---

## Next Steps

### Immediate (Before Launch)

1. ✅ Document system prompt in `/docs/agy-system-prompt.md`
2. ❌ Create `/tests/` directory structure
3. ❌ Implement unit tests for deterministic components
4. ❌ Create golden query test suite (10-15 representative queries)
5. ❌ Establish baseline for regression testing

### Short-Term (Post-Launch)

1. ❌ Add integration tests for RAG pipeline
2. ❌ Implement system prompt compliance test suite
3. ❌ Set up CI/CD testing workflow
4. ❌ Add production logging and monitoring

### Long-Term (Continuous Improvement)

1. ❌ Expand golden query test suite based on user feedback
2. ❌ Implement A/B testing for prompt variations
3. ❌ Add performance benchmarking
4. ❌ Create user feedback loop (thumbs up/down on responses)

---

## References

- **System Prompt:** `/docs/agy-system-prompt.md`
- **RAG Architecture:** `/docs/02-technical-architecture.md`
- **User Flows:** `/docs/03-ux-design-process.md` (lines 96-187)
- **Search Pipeline:** `/docs/03-ux-design-process.md` (lines 227-307)

---

*Testing AI systems is hard because they're non-deterministic. This strategy focuses on structural expectations (does it cite sources, does it avoid generic advice) rather than exact text matching.*
