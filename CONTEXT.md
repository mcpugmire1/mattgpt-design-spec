# Current Project Context

**Last Updated:** October 2026 (MATTGPT-322, -320, -294 and -323 pass against app 58fe71f)
**Session Status:** 🟢 Aligned with `origin/main` of the app repo as of October 2026; open items wait on app work (see Next Steps)

---

## Latest Update: October 2026 - MATTGPT-322 Voiced Decline and Landed Follow-ups

**What Changed:**
MATTGPT-322 (app 58fe71f) replaced rule 0b's fixed "Nothing I know about Matt covers that" with a voiced decline: first person, one or two sentences, the topic named, opening with a line `_choose_decline_line()` picks from `DECLINE_LINES` per answer.

- **05:** "When Agy doesn't know" and "Not found" describe the voiced decline for profile categories; other questions still have no decline path; the "Admit gaps honestly" example ("That's outside Matt's experience.") stays as the router banner's
- **04:** RAG bullet drops the old 0b sentence; whether Agy declines is still left to the model
- **11:** reports MATTGPT-320's golden run (October 16, 2026, app 3d1f63d) with its evidence path; Q59's xfail note dropped (MATTGPT-319); entity detection queries skip only the entity check and still run through the structural checks
- **02, 04, index.md:** point to doc 11 for the run result instead of restating it; index.md's "What holds" says the eval runs after major changes to the answer pipeline
- **12:** the export rebuilds the JSONL from Excel on every run (MATTGPT-323); the Career Arc is read from `career_arc` in the profile (MATTGPT-294)

**Found along the way (app side, handed to the backlog session):**
- `eval_summary` is built from counting query ids, not a run; the last golden run's result is typed in doc 11 until `extract_facts.py` syncs it (ticket logged)

## Previous Update: October 2026 - MATTGPT-275 Re-pass

**What Changed:**
MATTGPT-275 (app 1095d2c) retired `rag_answer()`: Ask Agy now runs the nonsense rules and router redirects, then a tool loop in which the model writes its own searches and streams its answer. The confidence gate, entity detection and pinning, synthesis retrieval and temperatures went with it. Nearly every doc still described the old pipeline; every synced number stayed correct. Checked against the app's `origin/main`, one doc per commit.

- **02, 09:** answer pipeline and diagram, Ask Agy backend reference, session keys, error handling; no confidence gate
- **05:** no decline path; openers model-written from the lists; error copy in `agy_answer()`; dash and comparison rules; closers still PAUSED
- **11:** the eval calls `agy_answer()` and checks answers and sources; no pass rate claimed; the 95% and 98% gates dropped
- **04, 01, 03, index.md:** model, pipeline and grounding claims match the prompt instruction; no eval number until a run on the new pipeline
- **10, 12:** entity fields no longer detected or pinned; the `$or` entity filter has no caller; dates reach the answer model
- **keeping-docs-in-sync.md:** the decline example became the case study for prose drift
- **Nav and wireframes:** nav em dashes and doc 08 description fixed; legacy prototypes moved to `archive/wireframes-legacy/` (they were unlinked but still published); v4 marked superseded where the app moved on

**Found along the way (app side, handed to the backlog session):**
- `facts.chat_model` comes from `DEFAULT_CHAT_MODEL`, which nothing in production reads; the answer model (`ASK_ANSWER_MODEL`) is not synced, so the docs cite the constant by name
- No golden eval has run on the shipped pipeline; Q59's `xfail_reason` still cites the retired gate
- When Pinecone returns no hits, `semantic_search()` falls back to the corpus in file order (MATTGPT-292)
- The How Agy Searches dialog says Agy "says so plainly" when no story covers a question; the prompt instructs that only for profile categories (rule 0b)

## Previous Update: October 2026 - Drift Pass Against Arch Sync 850071c

**What Changed:**
A drift audit compared the spec with the app repo's `origin/main` and found the voice guide, architecture, API, data pipeline and testing docs describing behavior the app no longer has. Each fix was checked against `origin/main`, never the app repo's working tree, and committed one doc at a time.

- **05 Agy Voice Guide:** openers match `STANDARD_OPENERS` / `SYNTHESIS_OPENERS`; "Great question!" removed; decline banners replace pivot and clarifying-question copy; closing offers marked PAUSED until MATTGPT-275 (ADR 035); loading-message dog humor allowed, answers excluded; thank-you and frustrated-user sections marked NOT IMPLEMENTED
- **11 Testing and Quality:** unit tests are enforced by CI (`test.yml`) and the pre-push hook; eval and BDD stay manual; Q59 is a known expected failure; Ask Agy declines low confidence, My Work warns
- **12 Data Pipeline:** Excel owns `public_tags` and `Interview Questions`; ids are `Title|Client` slugs; tags are LLM-generated; Pinecone metadata casing; MATT_DNA's derived, profile and hardcoded sections
- **02 Technical Architecture:** pipeline diagram reordered (entity detection before search and the gate); router rejects only out_of_scope and personal at HARD_ACCEPT; synthesis retrieval; banner table
- **09 API Reference:** module tables, data flow, `semantic_search()` return shape, `W_KW`, filter keys, environment variables
- **07 and 08 CSS and Mobile:** desktop-first with `max-width` overrides; mobile range includes 767px
- **04 Building MattGPT:** stack, topic-only intent routing, ranking, pipeline order; logging, tagging and export claims match the code
- **10 Data Model:** loader behavior, embedding text order, unused vocabulary path
- **index.md:** eval result reported from the latest recorded runs; last-updated footer added
- **keeping-docs-in-sync.md:** synced fact list by category; decline example
- **Site architecture page:** marked a dated historical snapshot
- **CLAUDE.md:** breakpoint rule points to doc 08 instead of restating values
- **Process:** the pass is captured as the `mattgpt-design-spec-sync` skill

**Found along the way:**
- `facts.eval_summary` is not a measured pass rate: `extract_facts.py` writes the query count over itself. The spec no longer presents it as one.
- App code defects (vocabulary key case, `place` metadata, `ERA_ORDER`, salary banner) are tracked in the app repo's backlog, not here.

## Previous Update: May 2026 - Feature Catch-Up + Design Spec Alignment

**What Changed (May 11):**
Brought design spec current with shipped codebase work: Role Match Phase 4 slice 1 (lock icon + password gate), triage agent surface (`scripts/assess_jd.py` wrapping `jd_assessor.py` for external orchestration), Role Match wireframes added to `/wireframes/`, BDD scenario count updated to 246 across 13 feature files, query logger column count corrected to 32, historical narrative extracted from 02-technical-architecture to codebase HISTORY.md, stale "LATER" roadmap entry replaced (Role Match shipped, not aspirational), index.md landing page updated with Role Match links.

**What Changed (May 1):**
Resolved 9 drift items between design spec and codebase. React migration reframe applied across 6 spec docs, voice guide implementation status corrected, public tags moved from planned to shipped, 3 Phase 2 enhancements marked Decided Against, intent family count corrected to 15, personal narrative placeholders marked complete.

**January-May 2026 changes reflected:**
- Role Match Phases 1-3 shipped April (JD assessment, story chips, action buttons)
- Role Match Phase 4 slice 1 shipped May (lock icon + password gate UI shell)
- Triage agent scaffold shipped May (`scripts/assess_jd.py` + `agent/` tree)
- `personal` intent family added to semantic router (warm redirect for age/salary/identity questions)
- Query logger expanded to 32-column schema (Google Sheets)
- Mobile responsive design shipped (`mobile_overrides.py`, 1,520 lines)
- BDD test suite expanded to 219 scenarios across 30 feature files
- Eval suite expanded from 31 to 64 golden queries (100% pass rate)

## Previous Update: January 30, 2026 - RAG Pipeline Documentation

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
- Expanded: 10 intent families → 15 intent families
- New families: `narrative`, `synthesis`, `out_of_scope`, `agile_transformation`, `personal`

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
- Created `docs/11-testing-and-quality.md`
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

### ✅ Personal Narrative (COMPLETE)
**File:** `/docs/04-building-mattgpt.md`

All 5 sections fully written with substantive personal content: Why I Built This, Key Challenges & Solutions, Lessons Learned, Build Timeline, Future Vision. No `[PLACEHOLDER]` markers remain.

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

1. **Swap hand-typed values for facts** once `extract_facts.py` emits them: `ask_answer_model` (docs 02, 04 and 09 cite `ASK_ANSWER_MODEL` by name until then), `role_match_model` (ticket text given to Matt Oct 28, 2026: rename `chat_model`, which since app 466b55a holds Role Match's model; doc 02 cites `DEFAULT_CHAT_MODEL` by name until then), `standard_openers`, `synthesis_openers`, `banner_copy`, `w_pc`, `w_kw`; the last golden run (doc 11's "Last run" line is typed with its date and evidence path until then); and stop using `eval_summary` until it reports a real run
2. **MATTGPT-275 closer work:** when the code-rendered closer and profile contact links land, remove doc 05's PAUSED callouts and revisit their stated reason
3. **MATTGPT-298:** update doc 05's salary line (salary questions get a polite reply instead of the nonsense banner)
4. **MATTGPT-292:** when the file-order fallback is fixed, drop the known-issue notes in docs 02 and 09 and describe the logged fallback, which is not shown as search results
5. **MATTGPT-236:** when the router leaves the answer path, update the docs that describe Ask Agy's router redirects (02, 05, 09), and revisit doc 05's "Admit gaps honestly" example ("That's outside Matt's experience.", the router banner) and the banner list under "Production Implementation". Doc 05's personal banner ("age, identity") becomes rule 0b's decline for the questions ADR 054 moves; check identity questions against the code, since ADR 054 doesn't name them. Breadth on big-picture and named division or client questions, with more than 3 cards, is described once the replacement signal is decided and lands (with MATTGPT-338)
6. **MATTGPT-128:** source cards are the stories the answer cited, not every story retrieved: doc 02's Step 5 and doc 01 say the opposite today. Fact answers show no story cards beside the fact cards
7. **MATTGPT-340:** the opener is code-picked and used word for word; doc 05's Quick Reference and Templates say the model writes its own first line
8. **MATTGPT-338:** doc 11's entity detection line says `evaluate_query()` marks those queries skipped and passed; once -338 lands they are skipped and not passed
9. **Wording variants:** when a variants measure runs on `agy_answer()`, describe coverage beyond synonyms (misspellings, punctuation, natural phrasing)
10. **After the MATTGPT-128 governor trial:** draft a change-governance page (removal list, Decision trailers, governor and checker, decision packet, guard hooks; the 1095d2c walkthrough as the case), shown in chat first; revisit `index.md`'s "What holds" and "With a team" lines
11. **Role Match intent lines** (Arch Governor's checker-passed draft, Oct 20, 2026, against app 4753532, which was not yet on `origin/main` when queued; re-check each against `origin/main` before writing). Form: the intent in one sentence plus links to the records behind it; link app-side ADRs and tickets rather than restating them. Do these after the out-of-date fixes.
    - R1 honest rating: add the under-call half (MATTGPT-244 acceptance); the rest is in `index.md` "The decisions" and doc 03 J1
    - R2 verbatim extraction, steady requirement count: ADR 052, MATTGPT-160, MATTGPT-302 (G1: doc 03 J1, doc 09 Services Layer)
    - R3 evidence-backed, nothing invented, a strong rating cites at least one item: the assessor prompt in `services/jd_assessor.py`, MATTGPT-244 (G2: docs 03 J1 and 01)
    - R4 under 90 seconds: already stated; no measurement defined (scorecard section 4, MATTGPT-160)
    - R5 agrees with Ask Agy: MATTGPT-268, MATTGPT-309 (G3: doc 03 J3 has it only as a failure mode)
    - R6 recommendation and fit score in the private view only: MATTGPT-012 (G4; waits on -012). The "private view" line in doc 02 What's Live goes with the out-of-date fixes
    - R7 no dashes in model-written text, except the JD's own wording or a cited story title: MATTGPT-302, ADR 048 (G5: doc 05 Implementation Notes covers Ask Agy only; wording waits on -302's D2)
    - R8 failures visible, a failed row marked unassessed, not dropped: scorecard driver 7 (G6: doc 09 Error Handling; retry and timeout wait on D10 at -302's go)
    - R9 Role Match's model name from `config/constants.py`: MATTGPT-302, MATTGPT-140 (G7: doc 02 Model Configuration; changes when -302 or -140 lands)
    - R10 desktop only, a gate below 1024px: ADR 062 (app dd73b45, not yet on `origin/main` when queued), `role_match.feature` "DESKTOP ONLY" scenarios (G8)
12. **Revisit the sync skill and the process** once the coverage gaps are closed: the spec holds intent, decisions and the visitor experience; code detail lives in `ARCHITECTURE.md` and the ADRs; Arch sync tags spec-visible ADRs onto this queue
13. **Agy's voice** (Matt's voice ticket; key to come from Matt): hold doc 05's "Who is Agy?", Voice DNA and Voice Principles until the ticket settles which description is the real one. Doc 05 describes a persona (traits, the 60/30/10 blend, tracking metaphors, dry humor) that `BASE_PROMPT` in `prompts.py` doesn't carry; its "## VOICE" block is warm, steady, grounded, confident and calm. Doc 05 calls Agy "he", `BASE_PROMPT` "she". Check visitor-facing copy with the `agy-voice-review` skill

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
