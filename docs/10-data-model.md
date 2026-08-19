# Data Model

**JSONL Schema and Field Definitions**

> This document covers the JSONL schema for story records: required fields, STAR framework fields, taxonomy fields, entity detection fields, and validation rules at load time. Field definitions here are the source of truth for embedding construction, Pinecone metadata, and filter logic.

---

## Table of Contents

1. [JSONL Schema Overview](#jsonl-schema-overview)
2. [Required Fields](#required-fields)
3. [STAR Framework Fields](#star-framework-fields)
4. [5P Taxonomy Fields](#5p-taxonomy-fields)
5. [Metadata Fields](#metadata-fields)
6. [Filtering & Tagging Fields](#filtering--tagging-fields)
7. [Field Reference Table](#field-reference-table)
8. [Data Validation Rules](#data-validation-rules)
9. [Example Story](#example-story)

---

## JSONL Schema Overview

Each story is a JSON object with **Title-case field names** (matching source Excel structure). Fields can contain:
- Strings (e.g., `"Title": "Platform Modernization"`)
- Lists (e.g., `"Process": ["Migrated to AWS", "Implemented CI/CD"]`)
- Comma-separated strings auto-parsed to lists (e.g., `"public_tags": "aws,cloud,platform"`)

**Loading Behavior:**
- List fields are normalized at load time (accepts both `"string"` and `["array"]`)
- Missing fields default to `""` (strings) or `[]` (lists)
- Stories without an `id` field are skipped to maintain Pinecone mapping integrity

---

## Required Fields

These fields are **required** for every story:

| Field | Type | Purpose | Example |
|-------|------|---------|---------|
| `id` | str | Unique story identifier (must match Pinecone vector ID) | `"101"`, `"story_abc123"` |
| `Title` | str | Project title / headline | `"Platform Modernization at JP Morgan"` |
| `Client` | str | Client/employer name | `"JP Morgan"`, `"Johnson & Johnson"` |
| `Industry` | str | Industry category | `"Financial Services"`, `"Healthcare"` |

**Validation:**
- `id` must be non-empty and non-zero
- `id` is coerced to string at load time
- Stories without valid `id` are skipped with warning

---

## Entity Detection Fields

**Critical for semantic router and query routing (used by detect_entity())**

| Field | Type | Purpose | Example |
|-------|------|---------|---------|
| `Client` | str | Client organization | `"JP Morgan"`, `"RBC"` |
| `Employer` | str | Employment org (Matt's employer) | `"Accenture"`, `"Independent"` |
| `Division` | str | Organizational division/unit | `"Cloud Innovation Center"`, `"Technology"` |

**Entity Pinning Behavior:**
- Query mentions "JP Morgan" → filters by `Client = "JP Morgan"`, pins matching story to #1
- Query mentions "Accenture" → filters by `Employer = "Accenture"`, pins matching story to #1
- Query mentions "Cloud Innovation Center" → filters by `Division = "Cloud Innovation Center"`
- Multi-field search uses Pinecone `$or` operator across entity fields

**Context Exclusion:**
Prefixes like "after", "leaving", "before" prevent entity filtering:
- "Career transition after Accenture" → broad search, NOT filtered to Accenture stories

---

## STAR Framework Fields

**STAR = Situation, Task, Action, Result**

Stories follow the STAR methodology for behavioral interviewing:

| Field | Type | Purpose | Example |
|-------|------|---------|---------|
| `Situation` | list[str] | Context bullets (1-3 items) | `["Legacy monolith with 200+ dependencies"]` |
| `Task` | list[str] | Challenge/objective bullets (1-3 items) | `["Modernize architecture without downtime"]` |
| `Action` | list[str] | Actions taken bullets (2-5 items) | `["Migrated to microservices", "Implemented CI/CD"]` |
| `Result` | list[str] | Outcomes with metrics (2-4 items) | `["Reduced deployment time by 75%"]` |

**Alias Fields (Legacy):**
- `Process` → Maps to `Action` (how bullets)
- `Performance` → Maps to `Result` (what bullets)
- `Purpose` → Maps to `Task` (why/goal)

---

## 5P Taxonomy Fields

**5P Framework = Person, Place, Purpose, Process, Performance**

These fields enable semantic search and intelligent filtering:

| Field | Type | Purpose | Example |
|-------|------|---------|---------|
| `Person` | str | Key stakeholder or role | `"VP of Engineering"`, `"Product Owner"` |
| `Place` | str | Location or organizational context | `"Innovation Center"`, `"New York HQ"` |
| `Purpose` | str | Business goal or objective | `"Accelerate time to market"` |
| `Process` | list[str] | How actions / approach bullets | `["Implemented Scrum", "Automated testing"]` |
| `Performance` | list[str] | Outcomes / results bullets | `["Increased velocity by 3x"]` |
| `5PSummary` | str | Pre-written 1-2 sentence summary | `"Goal: Modernize platform. Outcome: 40% cost reduction."` |

**5P Summary Priority:**
1. If `5PSummary` field exists → use as-is
2. Else compose from: `Purpose` + top 2 `Process` + strongest metric from `Performance`
3. Fallback: Generic summary from available fields

---

## Metadata Fields

Additional classification and context fields:

| Field | Type | Purpose | Example |
|-------|------|---------|---------|
| `Category` | str | High-level project category | `"Digital Transformation"` |
| `Sub-category` | str | Domain/technical area | `"Platform Engineering"`, `"Agile Transformation"` |
| `Solution / Offering` | str | Capability or service offered | `"Cloud Modernization"`, `"Innovation Leadership"` |
| `Role` | str | Matt's role on the project | `"Director of Delivery"`, `"Platform Lead"` |
| `Era` | str | Career phase for Timeline view grouping | `"Integration & Platform Foundations (2005-2008)"` |
| `Theme` | str | Thematic classification | `"Professional Narrative"`, `"Technical Leadership"` |
| `Use Case(s)` | list[str] | Business use cases addressed | `["Legacy migration", "Team scaling"]` |
| `Competencies` | list[str] | Skills/competencies demonstrated | `["Platform Engineering", "Agile Coaching"]` |
| `Interview Questions` | list[str] | Sample interview questions this story answers | See Interview Questions section |
| `Project Scope / Complexity` | str | Scope classification | `"Enterprise-scale transformation"`, `"Career Overview"` |
| `Start_Date` | str | Project start date (YYYY-MM format) | `"2019-03"`, `"2015-06"` |
| `End_Date` | str | Project end date (YYYY-MM format) | `"2023-09"`, `"2017-12"` |

**Field Relationships:**
- `Category` → Broad classification (e.g., "Digital Transformation")
- `Sub-category` → Specific domain (e.g., "Platform Engineering")
- `Solution / Offering` → Capability filter (e.g., "Cloud Modernization")
- `Era` → Used by Timeline View for career phase grouping
- `Theme` → High-level thematic tag for grouping related stories

---

## Interview Questions Field

**Purpose:** Sample interview questions that this story can answer

**Type:** list[str]

**Example:**
```json
"Interview Questions": [
  "How do you approach solving ambiguous problems at Accenture?",
  "What strategies have you employed to build high-trust engineering cultures?",
  "Walk me through your leadership journey at Accenture, focusing on transformation and innovation?"
]
```

**Use Cases:**
- Semantic search: Helps stories match queries phrased as questions
- Eval framework: Golden queries for testing RAG quality
- Interview prep: Quick reference for common behavioral questions

**Format:**
- Can be a single long string with comma-separated questions
- Can be an array of individual question strings
- Questions often include specific context (company names, metrics, domains)

---

## Filtering & Tagging Fields

Fields used for search, filtering, and semantic routing:

| Field | Type | Purpose | Example |
|-------|------|---------|---------|
| `public_tags` | list[str] | Searchable tags (comma-separated or array) | `["aws", "microservices", "kubernetes"]` |
| `Industry` | str | Industry filter (single-select) | `"Financial Services"` |
| `Solution / Offering` | str | Capability filter (single-select) | `"Platform Engineering"` |
| `Client` | str | Client filter (multi-select in UI) | `"JP Morgan"` |
| `Sub-category` | str | Domain filter (multi-select in UI) | `"Cloud Modernization"` |
| `Role` | str | Role filter (multi-select in UI) | `"Platform Lead"` |

**Tag Handling:**
- `public_tags` can be:
  - Comma-separated string: `"aws, cloud, platform"` → parsed to `["aws", "cloud", "platform"]`
  - JSON array: `["aws", "cloud", "platform"]` → kept as-is
- Tag matching is **case-insensitive**
- Empty/whitespace-only tags are filtered out

---

## Field Reference Table

Complete alphabetical field reference:

| Field | Type | Required | Searchable | Filterable | Entity Detection | Used in 5P Summary |
|-------|------|----------|------------|------------|-----------------|-------------------|
| `5PSummary` | str | No | Yes | No | No | Primary |
| `Action` | list[str] | No | Yes | No | No | Via `Process` |
| `Category` | str | No | Yes | No | No | No |
| `Client` | str | Yes | Yes | Yes | Yes | No |
| `Competencies` | list[str] | No | Yes | No | No | No |
| `Division` | str | No | Yes | No | Yes | No |
| `Employer` | str | No | Yes | No | Yes | No |
| `End_Date` | str | No | No | No | No | No |
| `Era` | str | No | Yes | Yes | No | No |
| `id` | str | **Yes** | No | No | No | No |
| `Industry` | str | Yes | Yes | Yes | No | No |
| `Interview Questions` | list[str] | No | Yes | No | No | No |
| `Performance` | list[str] | No | Yes | No | No | Outcome |
| `Person` | str | No | Yes | No | No | No |
| `Place` | str | No | Yes | No | No | No |
| `Process` | list[str] | No | Yes | No | No | Approach |
| `Project` | str | No | Yes | No | No | No |
| `Project Scope / Complexity` | str | No | No | No | No | No |
| `public_tags` | list[str] | No | Yes | Yes | No | No |
| `Purpose` | str | No | Yes | No | No | Goal |
| `Result` | list[str] | No | Yes | No | No | Via `Performance` |
| `Role` | str | No | Yes | Yes | No | No |
| `Situation` | list[str] | No | Yes | No | No | No |
| `Solution / Offering` | str | No | Yes | Yes | No | No |
| `Start_Date` | str | No | No | No | No | No |
| `Sub-category` | str | No | Yes | Yes | No | No |
| `Task` | list[str] | No | Yes | No | No | No |
| `Theme` | str | No | Yes | No | No | No |
| `Title` | str | Yes | Yes | No | No | Via summary |
| `Use Case(s)` | list[str] | No | Yes | No | No | No |

**Notes:**
- **Entity Detection** column shows fields used by `detect_entity()` in the semantic router. The detection set is intentionally narrower than the search set to avoid false positives on generic values.
- Once an entity is confirmed, search widens across more fields (see `ENTITY_SEARCH_FIELDS` in `config/constants.py`), including Title.
- Date fields (`Start_Date`, `End_Date`) are primarily for Timeline View and chronological sorting.

---

## Data Validation Rules

### At Load Time

**ID Validation:**
```python
story_id = story.get("id")
if story_id in (None, "", 0):
    # Skip story - Pinecone mapping requires stable IDs
    skipped_no_id += 1
    continue

story["id"] = str(story_id).strip()  # Coerce to string
```

**List Field Normalization:**

List-typed STAR and 5P fields (`Situation`, `Task`, `Action`, `Result`, `Process`, `Performance`, `Competencies`, `Use Case(s)`) accept either a string or an array in the source JSONL and are coerced to a list at load time by `normalize_story()` in `utils/corpus_loader.py`. Empty and whitespace-only values are dropped.

**Tag Parsing:**
```python
# Parse comma-separated tags to list
if "public_tags" in story and isinstance(story["public_tags"], str):
    story["public_tags"] = _split_tags(story["public_tags"])
    # "aws, cloud, platform" → ["aws", "cloud", "platform"]
```

### Data Quality Checks

**Metric Detection:**
- Stories flagged with metrics if they contain patterns like:
  - Percentages: `50%`, `3.5%`
  - Dollar amounts: `$1M`, `$250K`
  - Multipliers: `3x`, `10x`
  - Basis points: `10bps`, `25pp`

**Required Content:**
- Stories should have at least one of: `Process`, `Performance`, `5PSummary`
- Empty stories are loaded but won't rank well in search

---

## Example Story

```json
{
  "id": "101",
  "Title": "Platform Modernization at JP Morgan",
  "Client": "JP Morgan",
  "Employer": "Accenture",
  "Division": "Cloud Innovation Center",
  "Project": "Platform Modernization",
  "Industry": "Financial Services",
  "Category": "Digital Transformation",
  "Sub-category": "Platform Engineering",
  "Solution / Offering": "Cloud Modernization",
  "Role": "Director of Platform Engineering",
  "Theme": "Technical Leadership",
  "Era": "Innovation Center (2019-2023)",
  "Start_Date": "2019-03",
  "End_Date": "2020-12",
  "Project Scope / Complexity": "Enterprise-scale transformation",

  "Situation": [
    "Legacy monolithic architecture with 200+ microservice dependencies",
    "Manual deployment processes taking 2-3 weeks per release",
    "Team of 15 engineers struggling with technical debt"
  ],

  "Task": [
    "Modernize platform architecture without disrupting production",
    "Reduce deployment time and increase release frequency",
    "Build scalable CI/CD pipeline for 50+ services"
  ],

  "Action": [
    "Led migration from monolith to containerized microservices",
    "Implemented Kubernetes orchestration with auto-scaling",
    "Built Jenkins CI/CD pipeline with automated testing",
    "Established GitOps workflows and infrastructure as code"
  ],

  "Result": [
    "Reduced deployment time by 75% (2 weeks → 2 days)",
    "Increased release frequency from monthly to daily",
    "Achieved 99.9% uptime across all services",
    "Scaled team capacity to support 150+ developers"
  ],

  "Person": "VP of Engineering",
  "Place": "New York",
  "Purpose": "Accelerate time to market and reduce operational costs",

  "Process": [
    "Led migration from monolith to containerized microservices",
    "Implemented Kubernetes orchestration with auto-scaling",
    "Built Jenkins CI/CD pipeline with automated testing"
  ],

  "Performance": [
    "Reduced deployment time by 75%",
    "Increased release frequency from monthly to daily",
    "Achieved 99.9% uptime"
  ],

  "5PSummary": "Goal: Modernize platform architecture to accelerate delivery. Approach: Migrated to containerized microservices with Kubernetes and CI/CD automation. Outcome: Reduced deployment time by 75% and enabled daily releases.",

  "Competencies": [
    "Platform Engineering",
    "Cloud Architecture",
    "DevOps",
    "Team Leadership"
  ],

  "Use Case(s)": [
    "Legacy modernization",
    "Cloud migration",
    "DevOps transformation"
  ],

  "Interview Questions": [
    "Tell me about a time you modernized a legacy platform",
    "How did you reduce deployment time at JP Morgan?",
    "Describe your experience with cloud migration and Kubernetes"
  ],

  "public_tags": ["kubernetes", "aws", "microservices", "cicd", "platform-engineering"],
  "content": ""
}
```

---

## Field Evolution History

### January 2026 - Entity Detection & Evaluation
- Added entity fields: `Employer`, `Division`, `Project` (for semantic router)
- Added `Interview Questions` field (for eval framework and semantic search)
- Added date fields: `Start_Date`, `End_Date` (for Timeline View sorting)
- Added classification: `Theme`, `Project Scope / Complexity`
- Documented `content` field (reserved, currently unused)

### October 2025 - Filter Redesign
- Added primary filters: `Industry`, `Solution / Offering`
- Renamed domains → `Sub-category` for clarity
- Added `Category` for high-level grouping

### April 2025 - Initial MVP
- STAR fields: `Situation`, `Task`, `Action`, `Result`
- 5P fields: `Person`, `Place`, `Purpose`, `Process`, `Performance`
- Metadata: `Title`, `Client`, `Role`

### Legacy Field Mappings
Some fields have aliases for backward compatibility:
- `how` → `Process` (Action bullets)
- `what` → `Performance` (Result bullets)
- `why` → `Purpose` (Goal/objective)

---

## Data Sources

**Production data file:** `echo_star_stories_nlp.jsonl` ({{ site.data.facts.story_count_label }} stories)

Two-stage pipeline: Excel workbook → `generate_jsonl_from_excel.py` → `echo_star_stories.jsonl` (Stage 1 raw output) → semantic enrichment scripts → `echo_star_stories_nlp.jsonl` (Stage 2, production). See [Data Pipeline](12-data-pipeline) for the full ingestion workflow.

**Quality Assurance:**
- Manual review of STAR completeness
- Automated metric extraction validation
- Tag normalization and deduplication

---

## Semantic Search Integration

### Pinecone Vector Index

Stories are embedded and indexed in Pinecone for semantic search:

**Vector ID:** Must match story `id` field (required for result mapping)

**Metadata Indexed:**

`build_custom_embeddings.py` indexes all story fields as metadata via `build_metadata()`. Two layers: canonical PascalCase fields (Title, Client, Employer, Division, Role, Industry, Era, Purpose, Process, Performance, 5PSummary, public_tags, and full STAR content) plus lowercase UI-friendly duplicates for Pinecone filter queries (client, employer, division, role, project, industry, complexity, domain, tags, summary).

Simplified example:
```python
{
    "id": story["id"],
    "Title": story["Title"],       # PascalCase canonical
    "Client": story["Client"],
    "employer": employer.lower(),  # lowercase duplicate for $eq filters
    "division": division.lower(),
    # ... see build_metadata() in build_custom_embeddings.py
}
```

**Embedding Source:**
- Model: OpenAI `text-embedding-3-small`
- Input: a labeled composition assembled by `build_embedding_text()` in `build_custom_embeddings.py`. It front-loads `Use Case(s)`, then includes `Title`, `Theme`, `Industry`, `Sub-category`, `5PSummary`, the full STAR block (`Situation`, `Task`, `Action`, `Result`), `Process`, `Competencies`, `public_tags`, and `Interview Questions`. Each field is truncated by character limit, not by item count.
- Dimension: 1536

### Vocabulary Building

At app startup, `initialize_vocab()` extracts tokens from:
- `Title`
- `Client`
- `Sub-category` (domain)
- `5PSummary`
- `public_tags`

Used for token overlap validation in search queries.

---

## Filter Logic

### Primary Filters (Single-Select)
```python
# Industry filter - exact match
if filter["industry"] and story["Industry"] != filter["industry"]:
    return False

# Capability filter - exact match
if filter["capability"] and story["Solution / Offering"] != filter["capability"]:
    return False
```

### Advanced Filters (Multi-Select, OR Logic)
```python
# Client filter - story must be in selected clients list
if filter["clients"] and story["Client"] not in filter["clients"]:
    return False

# Domain filter - story Sub-category must be in selected domains
if filter["domains"] and story["Sub-category"] not in filter["domains"]:
    return False

# Tags filter - case-insensitive, any match
if filter["tags"]:
    story_tags = {t.lower() for t in story["public_tags"]}
    filter_tags = {t.lower() for t in filter["tags"]}
    if not (story_tags & filter_tags):  # Intersection must be non-empty
        return False
```

### Keyword Search (Token-Based ALL Match)
```python
# Query: "platform kubernetes aws"
# Story must contain ALL tokens in searchable fields:
searchable_fields = [
    "Title", "Client", "Role", "Sub-category",
    "Person", "Place", "Purpose",
    " ".join(Process), " ".join(Performance),
    " ".join(public_tags)
]

# Tokenize and check: all query tokens must appear in haystack
if not all(query_token in story_tokens for query_token in query_tokens):
    return False
```

---

## Related Documentation

- [API Reference](09-api-reference) - Function signatures and module overview
- [Technical Architecture](02-technical-architecture) - RAG pipeline and search architecture
- [Product Vision](01-product-vision) - STAR methodology and governance model

---

*Last updated: {{ site.data.page_dates['10-data-model'] }}*
