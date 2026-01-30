# Current Project Context

**Last Updated:** January 30, 2026 (RAG Pipeline Documentation Update)
**Session Status:** 🟢 Documentation Complete - Aligned with Jan 2026 Implementation

---

## Latest Update: January 30, 2026 - RAG Pipeline Documentation

**What Changed:**
Updated design specification to reflect current RAG pipeline implementation from llm_portfolio_assistant codebase.

### Documentation Updates

**Files Updated:**
- ✅ `docs/02-technical-architecture.md` - Added 5-stage RAG pipeline section
- ✅ `docs/09-api-reference.md` - Updated threshold values
- ✅ `docs/03-ux-design-process.md` - Updated query validation info
- ✅ `README.md` - Added Jan 2026 achievements and updated status
- ✅ `images/architecture/ask-mattgpt-flow.mermaid` - Complete flow rewrite showing all 5 stages

### Key Changes Documented

**1. 5-Stage RAG Pipeline (98.1% eval pass rate):**
- Stage 1: Rules-based nonsense detection (regex patterns)
- Stage 2: Semantic router (intent classification + out-of-scope detection)
- Stage 3: Confidence gating (Pinecone results scoring)
- Stage 4: Entity detection & story pinning
- Stage 5: Intent-aware ranking with context isolation

**2. Components Removed (Jan 2026):**
- ❌ Entity Gate threshold bouncer (was blocking legitimate narrative queries)
- ❌ `classify_query_intent()` LLM call (GPT-4o-mini redundant with semantic router)

**3. Semantic Router Enhancements:**
- Now handles: intent classification, synthesis detection, narrative detection, out-of-scope detection
- Expanded: 10 intent families → 14 intent families
- New families: `narrative`, `synthesis`, `out_of_scope`, `agile_transformation`

**4. Updated Thresholds:**
```python
# Semantic Router
HARD_ACCEPT = 0.80   # Clearly on-topic (unchanged)
SOFT_ACCEPT = 0.40   # Lowered from 0.72 (less aggressive filtering)

# RAG Confidence
CONFIDENCE_HIGH = 0.25  # Strong match
CONFIDENCE_LOW = 0.20   # Raised from 0.15

# Pinecone
PINECONE_MIN_SIM = 0.15
SEARCH_TOP_K = 10       # Unified (was 100/7 conflict)
```

**5. Model Upgrades:**
- Primary LLM: GPT-4o (upgraded from GPT-4o-mini)
- Embedding model: text-embedding-3-small (1536 dims) - unchanged

**6. Architecture Flow Diagram:**
- Updated mermaid diagram to show all 5 stages explicitly
- Shows synthesis bypass flow (no Pinecone search)
- Updated threshold values throughout

**New Documentation:**
- Created `docs/11-rag-quality-evaluation.md`
  - Eval framework overview (98.1% pass rate, 60+ golden queries)
  - Query categories (6): entity, behavioral, technical, marketing, synthesis, edge cases
  - Evaluation criteria: source relevance, entity pinning, meta-commentary, confidence thresholds
  - Eval-driven development: How evals guided Jan 2026 cleanup
  - Running evals, adding test cases, metrics & reporting

**Version Updates:**
- README.md: v2.0 → v2.1 (added 6th doc)
- 02-technical-architecture.md: v1.1 → v1.2
- 04-building-mattgpt.md: v1.1 → v1.2

---

## What We're Building

A **multi-purpose design specification** for MattGPT that serves as:
1. **Portfolio showcase** for recruiters (high-level credibility)
2. **Technical deep-dive** for hiring managers (architecture, decisions)
3. **Actionable design spec** for development (wireframes, flows, components)
4. **Learning artifact** for skill development (AI/ML, RAG, product thinking)

**Live Site:** https://mcpugmire1.github.io/mattgpt-design-spec/

---

## Why This Matters (Project Purpose & Context)

**IMPORTANT:** This design spec serves multiple critical purposes. Understanding this context explains why precision, completeness, and detail matter so much:

### 1. **Prove Architecture & Design Ability**
Demonstrates to employers, colleagues, and students that I can architect and design complex AI-powered systems from vision through technical implementation.

### 2. **Shareable Reference**
A polished artifact to share with:
- Future employers (portfolio credibility)
- Colleagues (collaboration reference)
- Students (teaching artifact)
- Development teams (handoff documentation)

### 3. **Build Personal Credibility**
This project helps rebuild my professional portfolio and validates my technical depth. It's a tangible representation of my capabilities in product thinking, AI/ML, and modern architecture.

### 4. **System of Record for Development**
This is THE authoritative reference for MattGPT development work. It captures:
- **Phase 1 Context:** When refactoring the monolithic Python Streamlit app
- **Phase 2 Vision:** When reimagining the app in React modern architecture
- **Component Specs:** Exact copy, UI patterns, filters, navigation, brand voice
- **Architecture Decisions:** RAG pipeline, data governance, technical evolution

### 5. **Productivity Insurance Against Context Loss**
**Why is this so detailed?** Because when Claude Code crashes or session tokens expire, we lose all context. If this isn't comprehensive, we waste time re-discovering:
- Exact button labels and navigation structure
- Filter dropdown options and view switcher logic
- Brand voice guidelines (Agy's personality, emoji usage)
- Architecture decisions and technical rationale
- What's done vs. what's placeholder

**Result:** Exhaustive documentation prevents productivity loss and exhaustion from repeated context rebuilding.

### 6. **Digital Representation of Matt & Agy**
This spec represents:
- **Matt's approach:** Detail-oriented, systematic, user-focused product thinking
- **Agy's personality:** Determined Plott Hound, loyal tracking companion, goofish charm

Expect hyper-focus on:
- UI accuracy (wireframes as golden source)
- Documentation correctness (no emoji inconsistencies)
- Functionality precision (filters, search, navigation)
- Brand consistency (🐾 for Agy, not 🐝)

**This is intentional, not perfectionism.** Accuracy matters because this spec is our shared source of truth.

---

## Current State (What's Done)

### ✅ Documentation (Complete)
- `/docs/01-product-vision.md` - WHY/HOW/WHAT, personas, data governance, AI system prompt
- `/docs/02-technical-architecture.md` - 3-phase evolution (MVP → Production → Enterprise)
- `/docs/03-ux-design-process.md` - Site architecture, user flows, wireframe specs
- `/docs/04-building-mattgpt.md` - Technical implementation, roadmap (has PLACEHOLDERS for personal story)
- `/docs/05-agy-voice-guide.md` - Complete brand voice documentation for Agy (Plott Hound AI assistant)

### ✅ Design Artifacts (Complete)
- 9 interactive HTML wireframes in `/wireframes/`
- Component inventory and sitemap in `/components/`
- Brand kit (logos, colors, typography) in `/brand-kit/`
- Architecture diagrams in `/images/architecture/`

### ✅ GitHub Pages (Deployed)
- Theme: `jekyll-theme-minimal`
- Logo: Agy hero image in sidebar
- All documentation auto-converted to styled HTML
- All wireframes accessible as clickable prototypes

### ✅ Wireframe vs Documentation Audit (COMPLETED - October 18)
**Status:** Complete! Systematic cross-reference of all wireframes against all documentation

**Work Completed:**
- Audited all 9 HTML wireframes (golden source) against MD documentation
- Fixed 4 critical discrepancies:
  1. Global navigation structure in `sitemap_navigation.md`
  2. Agy emoji inconsistencies (🐝 → 🐾) in 4 locations
  3. "See It In Action" heading emoji (🐝 → 🎯)
  4. AI loading state text
- Created two comprehensive audit reports:
  - `WIREFRAME-DOCUMENTATION-AUDIT.md` - Initial audit documenting fixes
  - `COMPREHENSIVE-WIREFRAME-AUDIT.md` - Exhaustive verification report

**Audit Coverage:**
- ✅ All 9 wireframes verified for consistency
- ✅ All 7 homepage cards cross-referenced
- ✅ Filter UI (4 dropdowns, view switcher, search bar)
- ✅ Footer consistency across all pages
- ✅ All button labels and CTAs
- ✅ Empty states and helper text
- ✅ Brand emoji usage (🐾 for Agy)

**Final Result:** Documentation is **100% accurate** after fixes, wireframes confirmed as golden source

**Final Update (October 18):** Closed all remaining gaps - added filter dropdown options and page subtitle

**Commits:** 2434ffd (initial fixes), 9605d72 (100% completion)

---

## Current Work (What's In Progress)

### ✅ User Journey Diagrams (COMPLETED - October 18)
**Status:** Complete! All 6 user journey diagrams now render as SVG images

**Solution Implemented:**
- Pre-rendered Mermaid diagrams to SVG using mermaid-cli
- Replaced code blocks with SVG image references
- Preserved source `.mmd` files for future edits

**What was created:**
- Flow 1: Banking Browse-First User (14KB SVG)
- Flow 2: Search-First User (19KB SVG)
- Flow 3: Capability-First User (18KB SVG)
- Flow 4: Conversational User (19KB SVG)
- Flow 5: Cross-Industry User (18KB SVG)
- Flow 6: Direct Search User (22KB SVG)

**Location:** `/images/wireflows/` (source + rendered)
**Format:** SVG images (reliable, fast, accessible)
**Colors:** Branded with MattGPT palette (purple → indigo → blue → green)
**Note:** These are high-level navigation flows, not true wireflows with screen-to-screen UI component mapping

**Status:** ✅ **COMPLETE** - Diagrams render correctly on GitHub Pages

---

### ✅ Custom Brand Styling (COMPLETED - October 18)
**Status:** Complete! MattGPT brand colors applied throughout the site

**What was done:**
- Created `/assets/css/style.scss` with CSS custom properties
- Applied brand colors to links, headers, tables, code blocks, blockquotes
- Fixed SVG arrow visibility in architecture diagrams
- Improved responsive layout for diagrams

**Status:** ✅ **COMPLETE** - Site has consistent branding

---

### ✅ Architecture Diagrams Replaced with Original PDF Images (COMPLETED - October 19)
**Status:** Complete! Replaced Mermaid-generated diagrams with original PDF screenshots

**Problem Identified:**
- Mermaid diagrams were recreations that didn't match the quality and completeness of the original PDF
- Lost the cohesive unit of flow diagram + table in Slide 8
- Site architecture hierarchy was harder to read in Mermaid format

**Solution Implemented:**
1. **Replaced Mermaid with Original PDFs:**
   - `tech_rag_architecture.png` - Direct screenshot from PDF Slide 8 (flow diagrams + 4-phase table)
   - `site_architecture.png` - Direct screenshot from PDF Slide 9 (page hierarchy)
2. **Removed Old Files:**
   - Deleted `rag_build_run.svg`, `rag_build_run.mmd`
   - Deleted `ask_mattgpt_pipeline.svg`, `ask_mattgpt_pipeline.mmd`
   - Deleted `site_architecture.svg`, `site_architecture.mmd`
3. **Updated Documentation:**
   - `index.md` - Updated diagram links to reference PNG images

**Current Architecture Diagrams:**
- **Slide 8:** RAG Build + Run Architecture (`tech_rag_architecture.png`) - Flow diagrams + 4-phase implementation table
- **Slide 9:** Site Architecture (`site_architecture.png`) - Page hierarchy & navigation structure

**Status:** ✅ **COMPLETE** - Using original PDF diagrams that represent hours of design work

---

### 🔄 True Wireflows (OUTSTANDING - In Progress)
**Status:** Matt creating in Miro, will integrate later

**What's Needed:**
True wireflows showing screen-to-screen UI interactions with wireframe mockups and tap/click annotations. Example:
```
[Wireframe: Home Page]
  → [tap on "See Banking Projects" button]
  → [Wireframe: Banking Landing Page showing 16 cards]
  → [tap on "Agile Transformation" card]
  → [Wireframe: Category page]
```

**Current State:**
- ✅ User journey diagrams exist (high-level navigation paths)
- 🔄 True wireflows in progress in Miro

**Integration Plan (TBD):**
- Option 1: Export Miro boards as SVG/PNG → Add to `/images/wireflows/`
- Option 2: Embed Miro board links in documentation
- Option 3: Screenshot key flows and annotate in markdown

**File Location (when ready):** `/images/wireflows/true-wireflows/` or `/images/miro-exports/`

---

### 🔄 Personal Narrative (OUTSTANDING)
**File:** `/docs/04-building-mattgpt.md`

**Placeholder sections needing Matt's input:**
1. **Why I Built This** - Personal motivation, triggering scenarios, timeline
2. **Key Challenges & Solutions** - Technical problems faced, how solved
3. **Lessons Learned** - Insights, surprises, advice for others
4. **Build Timeline** - Week-by-week breakdown of development
5. **Future Vision** - 3-5 year outlook, adjacent problems, career alignment

**Status:** Technical content written, personal story sections marked with `[PLACEHOLDER]`

---

## Known Issues

### Issue 1: Wireframe 404s (RESOLVED)
- **Problem:** HTML wireframes returning 404 errors
- **Root cause:** Jekyll trying to process HTML files instead of serving as static assets
- **Solution:** Added `keep_files` and `include` rules to `_config.yml`
- **Status:** Fixed, pushed to GitHub

### Issue 2: Documentation Link Errors (RESOLVED)
- **Problem:** Links to docs returning 404s
- **Root cause:** Missing baseurl prefix, incorrect `.html` extensions
- **Solution:** Updated all links in `index.md` to include `/mattgpt-design-spec/` prefix
- **Status:** Fixed, pushed to GitHub

### Issue 3: Brand Guide Not Visible (RESOLVED)
- **Problem:** Brand kit assets in folder but not accessible on site
- **Root cause:** No link from landing page
- **Solution:** Added "Brand Guidelines" link to landing page
- **Status:** Fixed, pushed to GitHub

### Issue 4: Cross-Industry Landing Page 404 (RESOLVED - October 18)
- **Problem:** Cross-industry landing page link returning 404 error
- **Root cause:** File doesn't exist yet
- **Solution:** Changed to disabled "Coming Soon" card with visual styling
- **Status:** Fixed, pushed to GitHub
- **Commit:** 5ba5b54

### Issue 5: Footer Width Inconsistency in Wireframes (RESOLVED - October 18)
- **Problem:** Footer in Explore Stories wireframes stretching full browser width, inconsistent with other pages
- **Root cause:** Footer div missing width constraints
- **Solution:** Added `max-width: 1400px; margin: 0 auto;` to all 4 Explore Stories wireframes
- **Files fixed:**
  - `explore_stories_table_wireframe.html`
  - `explore_stories_cards_wireframe.html`
  - `explore_stories_timeline_wireframe.html`
  - `explore_stories_mobile_wireframe.html`
- **Status:** Fixed, pushed to GitHub
- **Commit:** 5ba5b54

---

## ✅ Streamlit App Styling Complete (COMPLETED - October 19)
**Status:** Complete! Streamlit app fully restyled to match wireframe design

**What was done:**
1. **Removed sidebar navigation** - Changed to top button navigation for better mobile UX
   - Set `USE_SIDEBAR_NAV = False` in `app.py:433`
   - Set `initial_sidebar_state="collapsed"` in `app.py:311`

2. **Applied purple gradient brand identity** throughout app
   - Hero section: Full gradient background (`#667eea` → `#764ba2`) with white text
   - Stats bar: Changed from cards to grid-with-borders layout (wireframe style)
   - Category cards: All 6 cards now have gradient backgrounds with white text
   - Buttons: White buttons with purple text, gradient hover effects

3. **CSS overhaul** in `/ui/components.py`
   - Expanded from 725 to 1219 lines (494 new lines of CSS)
   - Replaced all blue colors with purple gradient scheme
   - Added comprehensive component styling (badges, pills, alerts, forms)
   - Improved typography and spacing throughout

4. **Fixed environment issues**
   - Recreated virtual environment after folder move from /Downloads
   - Reinstalled all dependencies
   - Verified streamlit working: `venv/bin/streamlit run app.py`

**Files modified:**
- `/Users/matthewpugmire/Projects/portfolio/llm_portfolio_assistant/app.py` (lines 311, 433)
- `/Users/matthewpugmire/Projects/portfolio/llm_portfolio_assistant/ui/components.py` (725 → 1219 lines)

**Status:** ✅ **COMPLETE** - Ready for user to refresh browser and test

---

## Next Steps (Immediate Priorities)

1. **Test Streamlit styling changes** - Refresh browser and verify all visual updates
2. **Deploy updated Streamlit app** to Streamlit Cloud for job search
3. **Fill in placeholder sections** in `04-building-mattgpt.md` with personal narrative (lower priority)
   - Why I Built This
   - Key Challenges & Solutions
   - Lessons Learned
   - Future Vision
   - Build Timeline

---

## Next Steps (Nice-to-Have / Later)

1. **PDF exports** of documentation for offline sharing
2. **Analytics setup** to track which pages get the most attention
3. **Custom domain** (e.g., mattgpt-design.com) instead of GitHub Pages URL
4. **Additional CSS refinements** for mobile optimization

---

## Key Decisions Made

### Design Decisions
- **Theme:** Chose `jekyll-theme-minimal` over Cayman/Slate for clean, technical aesthetic
- **Logo:** Using Agy hero image in sidebar (Plott Hound mascot)
- **Structure:** Markdown as source of truth → multiple outputs (GitHub Pages, PDF, etc.)

### Technical Decisions
- **GitHub Pages:** Free hosting, auto-builds on push, Jekyll for markdown → HTML
- **File organization:** Logical folders (docs, wireframes, components, brand-kit, images)
- **Link strategy:** Absolute paths with baseurl to avoid 404s

### Content Decisions
- **4 core docs:** Split into logical sections (vision, architecture, UX, building)
- **Placeholder approach:** Technical content complete, personal narrative sections marked for later
- **Multi-audience:** Documentation serves recruiters, hiring managers, AND developers

---

## Repository Structure

```
mattgpt-design-spec/
├── _config.yml                          # Jekyll configuration
├── index.md                             # Landing page (auto-converted to HTML)
├── README.md                            # GitHub repo description
├── CONTEXT.md                           # This file (session state, project status)
│
├── docs/                                # Core documentation
│   ├── 01-product-vision.md
│   ├── 02-technical-architecture.md
│   ├── 03-ux-design-process.md
│   └── 04-building-mattgpt.md          # Has PLACEHOLDER sections
│
├── wireframes/                          # Interactive HTML prototypes
│   ├── homepage_wireframe.html
│   ├── banking_landing_page_wireframe.html
│   ├── explore_stories_table_view_wireframe.html
│   └── [6 more wireframes]
│
├── components/                          # Component library
│   ├── component_inventory.md
│   └── sitemap_navigation.md           # App navigation flows (OLD app)
│
├── images/                              # Organized assets
│   ├── logos/                          # Agy variations
│   ├── architecture/                   # RAG diagrams, site architecture
│   ├── screenshots/                    # App screenshots
│   └── wireframes/                     # Wireframe mockup images
│
├── brand-kit/                           # Complete brand assets
│   ├── brand_kit_preview.html          # Visual brand guide (9MB)
│   ├── logos/
│   ├── chat_avatars/
│   ├── favicons/
│   └── [more brand assets]
│
└── data/
    └── wireframe_annotations_all.csv   # Wireframe component data
```

---

## How to Resume After Context Loss

**If Claude Code crashes or you start a new session:**

1. **Read this file first** (`CONTEXT.md`) - it captures the current state, project purpose, and completed work
2. **Review recent git commits** for latest changes:
   ```bash
   git log --oneline -10
   ```
3. **Ask Matt:** "What are you working on right now?" to confirm current focus
4. **Reference archived planning:** `PROGRESS-ARCHIVE.md` contains initial planning phases (historical reference only)

**Update this file** at the end of each major session or milestone.

---

## Contact / Key Info

- **GitHub Repo:** https://github.com/mcpugmire1/mattgpt-design-spec
- **Live Site:** https://mcpugmire1.github.io/mattgpt-design-spec/
- **Live App (MVP):** https://askmattgpt.streamlit.app
- **Email:** mpugmire@gmail.com
- **LinkedIn:** https://www.linkedin.com/in/mattpugmire/

---

*This file is the single source of truth for "where are we right now?" Keep it updated!*
