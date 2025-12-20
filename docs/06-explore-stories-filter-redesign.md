# Explore Stories Filter Redesign (Phase 4)

**Status:** ✅ Implemented (December 2025)
**Design Date:** October 28, 2024
**Implementation Completed:** December 2025
**Related Implementation Doc:** [llm_portfolio_assistant/EXPLORE_STORIES_UX_REDESIGN.md](../../llm_portfolio_assistant/EXPLORE_STORIES_UX_REDESIGN.md)

---

## Overview

This document provides a high-level summary of the Explore Stories filter redesign. For detailed implementation specifications, wireframes, testing plans, and technical details, see the **[full implementation spec](../../llm_portfolio_assistant/EXPLORE_STORIES_UX_REDESIGN.md)** in the implementation repository.

---

## Context

### Problem Statement
Current Explore Stories filters are misaligned with the data model:
- Some filters are empty (Audience, Tags) due to data structure changes
- "Domain" uses synthetic field combining Category + Sub-category
- "Solution / Offering" (capability) is not filterable, blocking landing page integration
- Filter UI is cluttered with non-functional dropdowns

### User Journey Alignment
```
Home → Landing Page (Banking/Cross-Industry) → Explore Stories → Ask MattGPT
         ↓                                           ↓               ↓
      Overview/Viz                              Faceted Browse    Conversational
      "47 projects"                             Filter & scan     Deep questions
```

Landing pages provide **data visualization** to show the big picture. Explore Stories provides **faceted browsing** to drill into specific project subsets. The redesign enables seamless navigation between these stages.

---

## Design Solution

### New Filter Layout

**Primary Filters (Always Visible):**
- Search keywords (semantic + keyword search)
- Industry (Financial Services / Banking, Cross Industry, etc.)
- Capability (Agile Transformation, Modern Engineering, etc.)

**Advanced Filters (Collapsed by Default):**
- Client (JP Morgan Chase, RBC, etc.)
- Role (Director, Architect, etc.)
- Domain (Application Modernization, Security & Compliance, etc.)

### Visual Design
```
┌────────────────────────────────────────────────────────────────────┐
│  [Search keywords...                ] [Industry ▼] [Capability ▼] │
│                                                                     │
│  ▸ Advanced Filters                                      [Reset]   │
└────────────────────────────────────────────────────────────────────┘
```

**Expanded State:**
```
┌────────────────────────────────────────────────────────────────────┐
│  [Search keywords...                ] [Industry ▼] [Capability ▼] │
│                                                                     │
│  ▾ Advanced Filters                                      [Reset]   │
│    [Client ▼] [Role ▼] [Domain ▼]                                 │
└────────────────────────────────────────────────────────────────────┘
```

---

## Key Changes

### Data Model
**From:** Synthetic fields created in `load_star_stories()` with business logic
**To:** "Dumb loader" that preserves all JSONL fields as-is

**Impact:**
- No data loss - all JSONL fields available to UI
- `Solution / Offering` field now accessible for filtering
- Loader becomes pure data loading (no transformation logic)

### Filter Mappings
| UI Label | JSONL Field | Notes |
|----------|-------------|-------|
| Industry | `Industry` | NEW - not currently a filter |
| Capability | `Solution / Offering` | NEW - critical for landing page integration |
| Client | `Client` | KEEP - works correctly |
| Role | `Role` | KEEP - works correctly |
| Domain | `Sub-category` | UPDATE - use Sub-category directly |
| ~~Audience~~ | ~~personas~~ | REMOVE - killed in previous iteration |
| ~~Tags~~ | ~~tags~~ | REMOVE - empty in data |
| ~~Domain Category~~ | ~~Category~~ | REMOVE - redundant with Capability |

---

## Landing Page Integration

### Current Behavior
- Click "View Projects →" under "Agile Transformation & Delivery (8 projects)"
- Navigates to Explore Stories with NO filters applied

### New Behavior
```python
# Banking landing page button handler
if st.button("View Projects →"):
    st.session_state["prefilter_industry"] = "Financial Services / Banking"
    st.session_state["prefilter_capability"] = "Agile Transformation & Delivery"
    st.session_state["active_tab"] = "Explore Stories"
    st.rerun()
```

**Result:** Shows exactly 8 Banking + Agile Transformation projects with visible filter chips

---

## UX Rationale

### Progressive Disclosure
**Primary Filters:** Answer the recruiter question "What type of work in what industry?"
- Industry = "I need someone with Banking experience"
- Capability = "Who can do Agile Transformation"

**Advanced Filters:** Power user features for deeper slicing
- Client = "Bonus if they've worked at JP Morgan"
- Role = "At Director+ level"
- Domain = "Specifically in Application Modernization"

### Information Hierarchy
```
Strategic/High-level
↓
[Search] [Industry] [Capability]
    ↓
    Tactical/Specific (collapsed)
    ↓
    ▸ Advanced: [Client] [Role] [Domain]
```

---

## Implementation Status

### ✅ Phase 1: Data Layer (Completed)
- Refactored `load_star_stories()` to preserve all JSONL fields
- Removed synthetic field creation
- All consumers use raw field names

### ✅ Phase 2: Filter Logic (Completed)
- Updated `utils/filters.py` to support Industry, Capability, Sub-category
- Removed synthetic "domain" field references

### ✅ Phase 3: Explore Stories UI (Completed)
- Implemented Primary + Advanced collapsed layout
- Industry and Capability dropdowns functional
- Client, Role, Domain in Advanced section
- Session state pre-filters working

### ✅ Phase 4: Landing Page Integration (Completed)
- Banking/Cross-Industry landing page button handlers implemented
- Pre-filter state management works across page transitions

### ✅ Phase 5: Other Pages (Completed)
- ask_mattgpt.py, semantic_router.py, scoring.py updated
- Semantic search and context building verified working

---

## Success Criteria

### User Experience
- ✅ Landing page → Explore Stories navigation shows correct filtered count
- ✅ Filter chips clearly show what's applied
- ✅ Advanced filters accessible but not cluttering primary UI
- ✅ Search continues to work with semantic + keyword fallback

### Technical
- ✅ No data loss from JSONL to story objects
- ✅ All 119 stories load correctly with new field structure
- ✅ Filter matching logic works with raw JSONL field names
- ✅ Pre-filter state management works across page transitions

---

## Testing Strategy

**Test Cases:**
1. Direct navigation to Explore Stories (no pre-filters)
2. Banking Landing → Agile Transformation (pre-filtered to 8 projects)
3. Cross-Industry Landing → Modern Engineering (pre-filtered to 26 projects)
4. Search + Filters combined (semantic search with facets)
5. Advanced Filters (further narrow pre-filtered results)
6. Filter chips (remove filters one by one)

---

## Related Documentation

### Design Spec Repo (This Repo)
- [03-ux-design-process.md](03-ux-design-process.md) - Overall UX design and site architecture
- [Interactive Wireframes](../wireframes/) - Explore Stories wireframes (table/card/timeline views)

### Implementation Repo
- **[EXPLORE_STORIES_UX_REDESIGN.md](../../llm_portfolio_assistant/EXPLORE_STORIES_UX_REDESIGN.md)** - Detailed implementation spec with code examples and testing plan
- [ARCHITECTURE.md](../../llm_portfolio_assistant/ARCHITECTURE.md) - Phase 4 refactoring context
- [SESSION_HANDOFF.md](../../llm_portfolio_assistant/SESSION_HANDOFF.md) - Current session context

---

## Implementation Decisions (Resolved)

- ✅ Industry filter: **Single-select** (st.selectbox)
- ✅ Capability filter: **Single-select** (st.selectbox)
- ✅ Advanced filters (Client, Role, Domain): **Multi-select** (st.multiselect)
- ✅ Advanced section: **State preserved** via session_state["show_advanced_filters"]
- ✅ Wireframes: **No update needed** - design intent captured

### Implementation Details Not in Original Design

#### 1. Versioned Widget State Management
To prevent Streamlit widget key collisions and ensure clean filter resets:
- Each filter type has version counter (`_widget_version_*`)
- Incremented on filter removal to force widget recreation
- Prevents stale state bugs

See: explore_stories.py:1490-1554

#### 2. Search Guarding
Search only executes when user explicitly submits form:
- `__search_triggered__` flag distinguishes intentional searches
- Prevents accidental Pinecone API calls on every keystroke
- Improves performance and reduces costs

See: explore_stories.py:1621-1642

#### 3. Confidence Banners
Three-tier confidence system for search results:
- **High:** "Found N matching stories for 'query'"
- **Low:** "Showing closest matches. Relevance may be low."
- **None:** "No strong matches. Matt may not have worked with this."

See: explore_stories.py:241-270

#### 4. Active Filter Chips
Visual chips showing applied filters with individual removal:
- Hash-based stable keys prevent button ID collisions
- "Clear all" resets entire filter state
- Syncs with widget versions for clean updates

See: explore_stories.py:273-354

---

## Next Steps

1. Review and approve this design doc
2. Begin Phase 1 implementation (dumb loader refactor)
3. Update interactive wireframes once UI is implemented
4. Document final implementation in this design spec

---

**Last Updated:** December 20, 2025 (Post-Implementation)
**Version:** 1.1 (Implementation Complete)
**Implementation Status:** ✅ Complete
