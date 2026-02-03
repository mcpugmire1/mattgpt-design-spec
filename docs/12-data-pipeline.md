# Data Pipeline & Ingestion

**From Excel to Production RAG: Complete Data Flow**

> This document describes the complete data pipeline from source Excel spreadsheet through embedding generation to production semantic search.

---

## Table of Contents

1. [Pipeline Overview](#pipeline-overview)
2. [Stage 1: Excel to JSONL](#stage-1-excel-to-jsonl)
3. [Stage 2: Manual Enrichment](#stage-2-manual-enrichment)
4. [Stage 3: Embedding Generation](#stage-3-embedding-generation)
5. [Data Governance Principles](#data-governance-principles)
6. [Ingestion Workflow](#ingestion-workflow)
7. [Cost & Performance](#cost--performance)
8. [Migration History](#migration-history)

---

## Pipeline Overview

**3-Stage Data Pipeline:**

```
Excel Master Sheet → JSONL Conversion → Semantic Enrichment → Embeddings → Pinecone Index
```

**Purpose:** Transform structured STAR stories from Excel into semantically searchable vectors while preserving data integrity and governance.

**Guiding Principle:** **Excel is the single source of truth.** All content originates there. Scripts preserve, never overwrite.

---

## Stage 1: Excel to JSONL

### Purpose

Convert Excel master sheet to structured JSONL format while preserving existing data.

### Script Details

**Script:** `generate_jsonl_from_excel.py` (259 lines, root-level)

**Input:**
- Excel file: `MPugmire - STAR Stories - [DATE].xlsx`
- Sheet: `"STAR Stories - Interview Ready"`

**Output:**
- `echo_star_stories.jsonl` (130+ records)

### Key Features

**Merge Strategy:**
- Preserves existing `public_tags`, `content`, `id` fields from previous JSONL
- Only updates fields present in Excel (won't delete existing data)
- Slug-based key matching: `Title|Client` → `"title|client"`

**Backup Protection:**
- Auto-creates `.bak` file before overwriting
- Prevents accidental data loss
- Preserves previous state for rollback

**Dry-Run Mode:**
- Preview changes before committing
- Shows what will be added/updated
- Safety check before overwriting production data

### Environment Configuration

```bash
INPUT_EXCEL_FILE="MPugmire - STAR Stories - 01DEC25.xlsx"
SHEET_NAME="STAR Stories - Interview Ready"
DRY_RUN=False  # Set to True for preview
```

### Fields Extracted

All Title-case field names from Excel (matches source structure):

```python
{
    "id": "story_123",
    "Title": "Scaled Engineering Team from 4 to 150+",
    "Client": "Fortune 500 Bank",
    "Employer": "Accenture",
    "Division": "Cloud Innovation Center",
    "Industry": "Financial Services",
    "Sub-category": "Platform Engineering",
    "Role": "Head of Engineering",
    "Era": "Innovation Center (2019-2023)",
    "Start_Date": "2019-03",
    "End_Date": "2023-09",
    "Situation": ["Rapid growth, technical debt..."],
    "Task": ["Scale team while maintaining quality..."],
    "Action": ["Implemented hiring pipeline, mentorship..."],
    "Result": ["150+ engineers, 40% faster delivery..."],
    "Theme": "Team Scaling & Leadership",
    "Person": "VP of Engineering",
    "Place": "Innovation Center",
    "Purpose": "Scale engineering capacity while maintaining culture",
    "Process": ["Hiring pipeline", "Mentorship program"],
    "Performance": ["150+ engineers", "40% faster delivery"],
    "5PSummary": "Goal: Scale team to support rapid growth. Approach: Structured hiring + mentorship. Outcome: 150+ engineers, 40% faster delivery.",
    "Tags": "scaling, leadership, hiring",
    "public_tags": ["Team Scaling", "Engineering Leadership"]
}
```

### Normalization Rules

**List Fields:**
Auto-converted to arrays if stored as strings:
- `Situation`, `Task`, `Action`, `Result`
- `Process`, `Performance`, `Competencies`, `Use Case(s)`
- `Interview Questions`

**Tag Parsing:**
Comma-separated tags automatically split:
- `"aws, cloud, platform"` → `["aws", "cloud", "platform"]`

**ID Coercion:**
- All IDs coerced to string: `123` → `"123"`
- Empty/missing IDs → story skipped with warning

---

## Stage 2: Manual Enrichment

### Purpose

Add semantic metadata and public-facing tags to enhance search quality.

### Script Details

**Script:** `generate_public_tags.py` (171 lines, root-level)

**Input:** `echo_star_stories.jsonl` (from Stage 1)

**Output:** `echo_star_stories_nlp.jsonl` (enriched with semantic metadata)

### Enrichment Process

**1. Persona Tagging**

Map stories to interview personas for targeted retrieval:
- "Product Leader"
- "Technical Architect"
- "Delivery Manager"
- "Engineering Leader"

**2. 5P Summaries**

Generate concise summaries for quick scanning:
```
Goal: [Purpose]
Approach: [Process highlights]
Outcome: [Performance metrics]
```

**3. Public Tags**

Create user-friendly tags from technical metadata:
- Extract from: Title, Theme, Sub-category, Competencies
- Normalize: lowercase, deduplicate, semantic grouping
- Examples: "cloud transformation", "agile delivery", "platform modernization"

**4. Theme Assignment**

Categorize stories by transformation themes:
- Technical Leadership
- Team Scaling & Leadership
- Platform Modernization
- Agile Transformation
- Innovation Leadership
- Professional Narrative

### Why This Stage Matters

**Semantic Richness:**
- Excel captures facts (STAR stories)
- Enrichment adds semantic metadata for better search

**User-Facing Language:**
- Technical tags → accessible terminology
- Industry jargon → plain language

**Interview Prep:**
- Persona mapping helps filter stories for specific interview types
- 5P summaries provide quick story selection

---

## Stage 3: Embedding Generation

### Purpose

Generate vector embeddings and upsert to Pinecone for semantic search.

### Script Details

**Script:** `build_custom_embeddings.py` (291 lines, root-level)

**Input:** `echo_star_stories_nlp.jsonl` (enriched stories)

**Output:**
- Pinecone index: `matt-portfolio-v2`
- Namespace: `default`
- Dimensions: 1536 (OpenAI text-embedding-3-small)

### Embedding Strategy

**Text Composition for Embedding:**

```python
def build_embedding_text(story):
    """
    Combines multiple fields into rich semantic representation:

    1. Title (story title - improves keyword matching)
    2. Theme + Industry + Sub-category (behavioral context)
    3. 5P Summary (concise overview)
    4. STAR fields: Situation, Task, Action, Result (2-3 items each)
    5. Process details (max 3 items)
    6. Public tags (comma-separated)

    Result: ~200-400 token text optimized for behavioral queries
    """
    parts = [
        story.get("Title", ""),
        f"{story.get('Theme', '')} | {story.get('Industry', '')} | {story.get('Sub-category', '')}",
        story.get("5PSummary", ""),
        # STAR fields (truncated to 2-3 items each)
        " ".join(story.get("Situation", [])[:3]),
        " ".join(story.get("Task", [])[:3]),
        " ".join(story.get("Action", [])[:3]),
        " ".join(story.get("Result", [])[:3]),
        # Process and tags
        " ".join(story.get("Process", [])[:3]),
        ", ".join(story.get("public_tags", []))
    ]
    return " ".join(p for p in parts if p).strip()
```

**Why This Approach:**

- **Title inclusion:** Story titles contain key terminology users search for directly (e.g., "Platform Modernization", "Cloud Migration")
- **Behavioral focus:** Theme/Industry/Sub-category surface in behavioral interviews
- **Balanced detail:** Full STAR fields would dilute semantic signal (too verbose)
- **Tag inclusion:** Public tags capture essence without verbosity
- **Process field:** Critical for "how did you..." questions

**Key Decision (January 2026):**
Added Title to embedding text after observing that users often search for story titles verbatim (e.g., "tell me more about TICARA"). Previously, title-only queries had low similarity scores.

### Metadata Stored in Pinecone

**Pinecone stores both vector + metadata for filtering:**

```python
{
    "id": "story_123",  # Must match JSONL id for result mapping
    "title": "Scaled Engineering Team...",
    "client": "Fortune 500 Bank",
    "employer": "Accenture",
    "division": "Cloud Innovation Center",
    "project": "Team Scaling Initiative",
    "industry": "Financial Services",
    "domain": "Platform Engineering",  # Maps to Sub-category
    "role": "Head of Engineering",
    "theme": "Team Scaling & Leadership",
    "tags": ["Team Scaling", "Engineering Leadership"],
    "embedding": [0.023, -0.045, ...],  # 1536-dimensional vector
}
```

**Field Mapping (JSONL → Pinecone):**
- `Sub-category` → `domain` (for backward compatibility)
- All entity fields stored lowercase for consistent filtering
- Tags stored as array for multi-tag filtering

### Processing Stats

- **Stories:** ~130 stories
- **Time:** ~30 seconds for full re-index
- **Cost:** $0.0008 per full re-index (130 stories × ~300 tokens avg)
- **API:** OpenAI text-embedding-3-small @ $0.02 per 1M tokens

### Environment Configuration

```bash
STORIES_JSONL=echo_star_stories_nlp.jsonl
OPENAI_API_KEY=sk-...
PINECONE_API_KEY=...
PINECONE_INDEX_NAME=matt-portfolio-v2
PINECONE_NAMESPACE=default
```

---

## Data Governance Principles

### Single Source of Truth

**Excel is the master data source.** All story content, STAR fields, and metadata originate there.

**Why Excel:**
- Familiar editing environment (no JSON syntax errors)
- Version control via OneDrive/Dropbox/Git
- Easy bulk edits and data validation
- Copy/paste from interview prep notes

### Hybrid Sovereignty Model

**Excel Owns:**
- All story content (STAR fields, titles, clients, metadata)
- Structural changes (add/remove stories, field updates)
- Business logic (what stories exist, their content)

**JSONL Owns:**
- Derived fields: `public_tags`, `Interview Questions`, `content`
- Runtime state (enrichment artifacts)
- Semantic metadata

**Scripts Preserve:**
- Merge strategy: Excel updates flow in, JSONL-only fields preserved
- Backup on write: `.bak` files prevent data loss
- No destructive overwrites

### January 2026 Sovereignty Patterns

**1. Dynamic Identity (MATT_DNA)**

Sacred vocabulary and professional narrative derived from JSONL at startup:
```python
# Extracted from "About Matt" story in JSONL
MATT_DNA = {
    "identity": ["builder", "modernizer"],
    "philosophy": ["complexity to clarity", "proof over promises"],
    "not_looking_for": ["maintenance role", "status quo preservation"]
}
```

**Why Dynamic:**
- Single source of truth (JSONL "About Matt" story)
- No hardcoded narrative in code
- Update story → updates identity everywhere

**2. Multi-Field Entity Search**

Entity detection searches across 6 fields via Pinecone `$or`:
```python
# Query: "Show me Accenture projects"
# Searches: Client OR Employer OR Division OR Project OR Place OR Title
pinecone_filter = {
    "$or": [
        {"client": {"$eq": "Accenture"}},
        {"employer": {"$eq": "Accenture"}},
        # ... (5 more fields)
    ]
}
```

**Why Multi-Field:**
- "Accenture" might appear as Client, Employer, or Division
- Maximizes recall while maintaining precision
- Avoids false negatives from single-field matching

**3. UI Hydration**

All landing page counts derived dynamically from story data:
```python
# No hardcoded counts - derive from loaded stories
banking_count = len([s for s in stories if s.get("Industry") == "Financial Services"])
healthcare_count = len([s for s in stories if s.get("Industry") == "Healthcare"])
```

**Why Dynamic:**
- Add story to Excel → counts update automatically
- No stale "130 projects" text to maintain
- Data-driven, self-documenting

### Anti-Patterns (Don't Do This)

**❌ Hardcoded Lists:**
```python
# BAD: Will drift out of sync with data
CLIENTS = ["JP Morgan", "Capital One", "RBC", "Johnson & Johnson"]
```

**✅ Derived from Data:**
```python
# GOOD: Always accurate
clients = {s.get("Client") for s in stories if not is_generic_client(s.get("Client"))}
```

**❌ Manual JSONL Editing:**
- Don't hand-edit JSONL files for content changes
- Content changes belong in Excel (master source)
- JSONL editing allowed ONLY for: `public_tags`, `Interview Questions`, `content` (derived fields)

**✅ Excel-First Workflow:**
1. Edit story in Excel
2. Run `generate_jsonl_from_excel.py`
3. Review diff, commit if correct

---

## Ingestion Workflow

### Full Data Refresh (Content Changes)

**When:** Story content updated (STAR fields, titles, clients, metadata)

**Steps:**
1. Edit Excel master sheet
2. Run Stage 1: `python generate_jsonl_from_excel.py`
3. Run Stage 2: `python generate_public_tags.py`
4. Run Stage 3: `python build_custom_embeddings.py`
5. Restart app (reload JSONL)

**Time:** ~2 minutes end-to-end

### Partial Refresh (Enrichment Only)

**When:** Only semantic tags or personas updated (no content changes)

**Steps:**
1. Run Stage 2: `python generate_public_tags.py`
2. Run Stage 3: `python build_custom_embeddings.py`
3. Restart app

**Time:** ~1 minute

### Embedding-Only Refresh (Pinecone Updates)

**When:** Pinecone index needs rebuilding (no JSONL changes)

**Steps:**
1. Run Stage 3: `python build_custom_embeddings.py`

**Time:** ~30 seconds

**Cost:** $0.0008

### Hot Reload (Development)

**Streamlit auto-reload:** Changes to `.py` files trigger automatic rerun

**Data changes require manual restart:** JSONL loaded at startup, not watched

---

## Cost & Performance

### Embedding Generation

**OpenAI text-embedding-3-small:**
- **Rate:** $0.02 per 1M tokens
- **Story Size:** ~300 tokens average (after text composition)
- **130 Stories:** ~39,000 tokens = $0.0008 per full re-index
- **Time:** ~30 seconds

**Annual Cost (4 full refreshes/month):**
- 4 refreshes × 12 months × $0.0008 = **$0.038/year**
- Effectively free

### Pinecone Vector Database

**matt-portfolio-v2 Index:**
- **Tier:** Starter (free tier, 100K vectors)
- **Usage:** 130 vectors (0.13% of quota)
- **Dimensions:** 1536
- **Cost:** $0/month

### LLM Generation (GPT-4o)

**Per Query:**
- **Input tokens:** ~2,000-4,000 (context + prompt)
- **Output tokens:** ~200-600 (response)
- **Cost per query:** ~$0.01-0.03

**Monthly Cost (100 queries/month):**
- 100 queries × $0.02 avg = **$2/month**

**Production Scale (1000 queries/month):**
- 1000 queries × $0.02 avg = **$20/month**

### Processing Performance

**Full Pipeline (Excel → Production):**
- Stage 1 (Excel → JSONL): ~5 seconds
- Stage 2 (Enrichment): ~10 seconds
- Stage 3 (Embeddings): ~30 seconds
- **Total:** ~45 seconds

**Semantic Search (Runtime):**
- Query embedding: ~200ms
- Pinecone vector search: ~300ms
- LLM generation: ~3-5 seconds (GPT-4o)
- **Total query latency:** ~4-6 seconds

---

## Migration History

### v1: MiniLM-L6-v2 (April 2025)

**Embedding Model:** Sentence-BERT `all-MiniLM-L6-v2`
- **Dimensions:** 384
- **Cost:** Free (local model)
- **Performance:** Fast (~10ms per embedding)

**Limitations:**
- Poor behavioral query matching
- Limited semantic understanding
- Struggled with synonyms and related concepts

**Example Failure:**
- Query: "How did Matt handle difficult stakeholders?"
- Result: Low similarity to "Executive alignment" stories (missed semantic connection)

### v2: OpenAI text-embedding-3-small (October 2025)

**Embedding Model:** OpenAI `text-embedding-3-small`
- **Dimensions:** 1536
- **Cost:** $0.02 per 1M tokens
- **Performance:** ~200ms per embedding (API latency)

**Improvements:**
- Better behavioral query understanding
- Stronger semantic similarity on related concepts
- Improved handling of interview-style questions

**Example Success:**
- Query: "How did Matt handle difficult stakeholders?"
- Result: High similarity to "Executive alignment", "Stakeholder management" stories

**Why the Upgrade:**
Eval framework showed 15% improvement in query relevance (from 83% → 98% pass rate) after migration. The cost increase ($0.0008 per re-index) was negligible compared to quality gains.

### v3: Title Inclusion (January 2026)

**Change:** Added story Title to embedding text composition

**Why:**
- Users often reference stories by title: "Tell me more about TICARA"
- Previous approach: low similarity for title-only queries
- Solution: Include title in embedded text

**Impact:**
- Improved title-based query matching
- No performance degradation
- Minimal cost increase (~20 additional tokens per story)

---

## Key Takeaways

1. **Excel is the single source of truth** - All content originates there
2. **3-stage pipeline** - Conversion → Enrichment → Embedding (45 seconds end-to-end)
3. **Merge strategy** - Scripts preserve existing data, never destructive
4. **Cost-effective** - $0.0008 per re-index, effectively free at current scale
5. **Dynamic governance** - MATT_DNA, UI counts, entity lists all derived from data
6. **Migration validated** - Eval framework confirmed v2 embeddings improved quality by 15%

---

**Related Documentation:**
- [Technical Architecture](/mattgpt-design-spec/docs/02-technical-architecture) - RAG pipeline and system design
- [Data Model](/mattgpt-design-spec/docs/10-data-model) - JSONL schema and field definitions
- [RAG Quality Evaluation](/mattgpt-design-spec/docs/11-testing-and-quality) - How evals validate pipeline changes

---

*Last Updated: January 30, 2026*
*Version: 1.0 (Initial Documentation)*
