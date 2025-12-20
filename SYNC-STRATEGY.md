# Documentation Sync Strategy

## The Problem

We maintain two repositories:
- **llm_portfolio_assistant/** - Implementation (source of truth for technical details)
- **mattgpt-design-spec/** - Documentation (source of truth for strategy/vision)

When implementation changes, docs fall out of sync.

---

## Solution: Automated Sync Pipeline

### Strategy 1: Configuration-Driven Docs (Recommended)

**Concept:** Extract technical details from code config files into documentation.

**Implementation:**

```python
# llm_portfolio_assistant/config/system_config.py
SYSTEM_CONFIG = {
    "search": {
        "type": "semantic",  # not "hybrid"
        "embedding_model": "text-embedding-3-small",
        "embedding_dims": 1536,
        "vector_db": "pinecone",
        "confidence_thresholds": {
            "high": 0.25,
            "low": 0.15,
            "none": 0.0
        }
    },
    "semantic_router": {
        "hard_accept": 0.80,
        "soft_accept": 0.72,
        "intent_families": [
            "background", "behavioral", "delivery",
            "team_scaling", "leadership", "technical"
        ]
    },
    "rag": {
        "llm_model": "gpt-4o-mini",
        "top_k": 30,
        "story_count": 130
    }
}
```

**Sync Script:**

```python
# scripts/sync_docs.py
"""
Auto-generates technical sections of documentation from code config.
Run this before committing docs to ensure accuracy.
"""

import json
from pathlib import Path
from config.system_config import SYSTEM_CONFIG

def update_tech_architecture():
    """Update docs/02-technical-architecture.md with current config."""
    config = SYSTEM_CONFIG

    tech_stack = f"""
| Component | Technology | Purpose |
|-----------|-----------|---------|
| 🤖 **LLM** | OpenAI {config['rag']['llm_model']} | Natural language processing |
| 🔍 **Embeddings** | OpenAI {config['search']['embedding_model']} ({config['search']['embedding_dims']} dims) | Semantic search |
| 🌲 **Vector Database** | {config['search']['vector_db'].title()} | Vector similarity search |
| 📊 **Story Count** | {config['rag']['story_count']}+ projects | STAR-formatted case studies |
"""

    # Update markdown file
    # ... (template replacement logic)

def update_ux_design_process():
    """Update docs/03-ux-design-process.md search implementation details."""
    config = SYSTEM_CONFIG

    search_details = f"""
**{config['search']['type'].title()} Search:**
- Pinecone cosine similarity (vector matching)
- Minimum similarity threshold: {config['search']['confidence_thresholds']['low']}
- Top-k pool: {config['rag']['top_k']} candidates before ranking
- Confidence-based result filtering
  - High: ≥ {config['search']['confidence_thresholds']['high']}
  - Low: ≥ {config['search']['confidence_thresholds']['low']}
  - None: < {config['search']['confidence_thresholds']['low']}
"""

    # Update markdown file
    # ... (template replacement logic)

if __name__ == "__main__":
    update_tech_architecture()
    update_ux_design_process()
    print("✅ Docs synced from code config")
```

**Usage:**

```bash
# Before committing docs
cd llm_portfolio_assistant
python scripts/sync_docs.py

# Or add to pre-commit hook
git add docs/
```

---

### Strategy 2: CI/CD Validation

**Concept:** Automated tests that fail if docs are out of sync.

**Implementation:**

```python
# tests/test_docs_sync.py
"""
Test that documentation matches implementation.
Fails CI if technical details are out of sync.
"""

import pytest
from config.system_config import SYSTEM_CONFIG
from pathlib import Path

def test_tech_architecture_search_type():
    """Verify docs/02-technical-architecture.md matches search type."""
    arch_doc = Path("../mattgpt-design-spec/docs/02-technical-architecture.md").read_text()

    if SYSTEM_CONFIG['search']['type'] == 'semantic':
        assert 'Semantic search' in arch_doc
        assert 'hybrid search' not in arch_doc.lower()
    elif SYSTEM_CONFIG['search']['type'] == 'hybrid':
        assert 'Hybrid search' in arch_doc

def test_ux_design_confidence_thresholds():
    """Verify docs/03-ux-design-process.md has correct confidence values."""
    ux_doc = Path("../mattgpt-design-spec/docs/03-ux-design-process.md").read_text()

    high_threshold = SYSTEM_CONFIG['search']['confidence_thresholds']['high']
    low_threshold = SYSTEM_CONFIG['search']['confidence_thresholds']['low']

    assert f"high: ≥ {high_threshold}" in ux_doc or f"≥ {high_threshold}" in ux_doc
    assert f"low: ≥ {low_threshold}" in ux_doc or f"≥ {low_threshold}" in ux_doc

def test_embedding_model_sync():
    """Verify all docs reference correct embedding model."""
    model = SYSTEM_CONFIG['search']['embedding_model']
    dims = SYSTEM_CONFIG['search']['embedding_dims']

    docs_to_check = [
        "../mattgpt-design-spec/docs/02-technical-architecture.md",
        "../mattgpt-design-spec/docs/03-ux-design-process.md",
        "../mattgpt-design-spec/docs/04-building-mattgpt.md"
    ]

    for doc_path in docs_to_check:
        doc_text = Path(doc_path).read_text()
        assert model in doc_text, f"{doc_path} missing {model}"
        assert str(dims) in doc_text, f"{doc_path} missing embedding dims {dims}"
```

**GitHub Action:**

```yaml
# .github/workflows/docs-sync-check.yml
name: Documentation Sync Check

on: [pull_request, push]

jobs:
  check-docs-sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          path: llm_portfolio_assistant

      - uses: actions/checkout@v3
        with:
          repository: mcpugmire1/mattgpt-design-spec
          path: mattgpt-design-spec

      - name: Run sync validation tests
        run: |
          cd llm_portfolio_assistant
          pytest tests/test_docs_sync.py
```

---

### Strategy 3: Living Documentation with Badges

**Concept:** Add "last verified" badges to docs that link to implementation.

**Implementation:**

Add to top of technical docs:

```markdown
# Technical Architecture

> **Implementation Source:** [llm_portfolio_assistant/](https://github.com/mcpugmire1/llm_portfolio_assistant)
> **Last Verified:** December 20, 2025 (commit `02c261a`)
> **Config Reference:** [system_config.py](https://github.com/mcpugmire1/llm_portfolio_assistant/blob/main/config/system_config.py)

**Search Type:** ![semantic](https://img.shields.io/badge/search-semantic-blue)
**Embedding Model:** ![text-embedding-3-small](https://img.shields.io/badge/embeddings-text--embedding--3--small-green)
**Vector DB:** ![pinecone](https://img.shields.io/badge/vectordb-pinecone-orange)
```

---

### Strategy 4: Quarterly Audit Checklist

**Concept:** Scheduled manual review with checklist.

```markdown
# Quarterly Documentation Audit Checklist

Run this every 3 months or before major releases.

## Technical Accuracy

- [ ] Search type matches implementation (semantic vs hybrid)
- [ ] Embedding model and dimensions correct
- [ ] Confidence thresholds match code
- [ ] Story count accurate (currently 130+)
- [ ] Tech stack table reflects current dependencies
- [ ] Nonsense detection documented if implemented
- [ ] All code examples compilable/runnable

## Cross-Reference Check

- [ ] Compare `services/rag_service.py` with docs/03-ux-design-process.md
- [ ] Compare `utils/scoring.py` with confidence scoring docs
- [ ] Compare `semantic_router.py` with intent detection docs
- [ ] Compare `ARCHITECTURE.md` with docs/02-technical-architecture.md

## Implementation-to-Docs Mapping

| Implementation File | Documentation Section | Status |
|---------------------|----------------------|--------|
| `services/rag_service.py` | `03-ux-design-process.md` (Search Pipeline) | ✅ |
| `utils/scoring.py` | `03-ux-design-process.md` (Confidence Scoring) | ✅ |
| `services/semantic_router.py` | **MISSING** - needs documentation | ❌ |
| `nonsense_filters.jsonl` | **MISSING** - needs documentation | ❌ |
| `agy_system_prompt.txt` | `agy-system-prompt.md` | ✅ |

## Action Items

- [ ] Document semantic router
- [ ] Document nonsense filters
- [ ] Update story count
- [ ] Run sync script
- [ ] Commit with "docs: quarterly sync [date]"
```

---

## Recommended Approach

**Hybrid Strategy:**

1. ✅ **Implement Strategy 1** - Config-driven docs (automate what you can)
2. ✅ **Implement Strategy 2** - CI/CD validation (catch drift early)
3. ✅ **Implement Strategy 4** - Quarterly manual audit (catch edge cases)
4. ⚠️ **Strategy 3** - Use badges sparingly (visual indicator only)

**Benefits:**
- Automated sync for 80% of technical details
- CI catches drift before merge
- Quarterly audit ensures nothing slips through
- Clear ownership: code is source of truth

---

## Migration Path

**Week 1:**
1. Create `config/system_config.py` in llm_portfolio_assistant
2. Extract all hardcoded values (thresholds, model names, etc.)
3. Update code to reference config

**Week 2:**
1. Write `scripts/sync_docs.py`
2. Test manual sync
3. Run against current docs to verify

**Week 3:**
1. Add `tests/test_docs_sync.py`
2. Set up GitHub Action for CI
3. Test with intentional mismatch

**Week 4:**
1. Create quarterly audit checklist
2. Document the process
3. Schedule recurring calendar reminder

---

## Quick Wins (Today)

If you don't want to build the full pipeline yet:

**Add to README.md:**

```markdown
## 📝 Keeping Docs in Sync

**Before updating documentation:**

1. Check current implementation:
   ```bash
   # Search type
   grep "W_KW" llm_portfolio_assistant/services/rag_service.py

   # Confidence thresholds
   grep "CONF_" llm_portfolio_assistant/utils/scoring.py

   # Embedding model
   grep "text-embedding" llm_portfolio_assistant/services/embedding_service.py
   ```

2. Update docs to match

3. Cross-reference these files:
   - `services/rag_service.py` → `docs/03-ux-design-process.md`
   - `utils/scoring.py` → `docs/03-ux-design-process.md`
   - `semantic_router.py` → **TODO: document this**
   - `ARCHITECTURE.md` → `docs/02-technical-architecture.md`
```

---

## Next Steps

Would you like me to:
1. ✅ Implement the config-driven sync script?
2. ✅ Add CI/CD validation tests?
3. ✅ Create the quarterly audit checklist?
4. ✅ Document the nonsense detection system?
5. ✅ All of the above?

Let me know which approach you prefer! 🐾
