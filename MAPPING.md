# Project Mapping & Detailed Inventory

**Purpose:** Detailed section-by-section inventory of all files, their status, and cross-references. This is the "blueprint" for seamless context restoration.

**Last Updated:** October 17, 2024

---

## Documentation Files (Core Content)

### `/docs/01-product-vision.md` ✅ COMPLETE

**Status:** Complete - extracted from PDF slides 3-7

**Sections:**
1. ✅ **The Credibility Engine** - WHY/HOW/WHAT framework
   - Line 19-52: Core value proposition and positioning
   - Tagline: "Matt doesn't claim credibility — he proves it in real time"

2. ✅ **Target User Personas** - 3 distinct personas
   - Line 76-149: Recruiter, Hiring Manager, Content User (Matt)
   - Each with: Profile, Primary Need, Feature Drivers, Focus, Typical Journey

3. ✅ **Data Model & Two-Layer Governance**
   - Line 152-211: Layer 1 (STAR integrity) + Layer 2 (tagging intelligence)
   - Mandatory STAR fields documented
   - 5P + Semantic tagging explained

4. ✅ **MattGPT AI Core** - System prompt and guardrails
   - Line 213-286: Operational mandate, archetype, non-negotiable guardrails
   - Integrity mandate: All answers must be auditable

5. ✅ **Project Scope & Boundaries**
   - Line 289-350: MVP definition with IN SCOPE vs OUT OF SCOPE
   - Clear guardrails documented

**Cross-references:**
- Referenced by: `index.md` (landing page link)
- References: Technical architecture doc, UX design doc

---

### `/docs/02-technical-architecture.md` ✅ COMPLETE

**Status:** Complete - extracted from PDF slide 8 + architecture evolution HTML

**Sections:**
1. ✅ **Architecture Roadmap Overview**
   - Line 10-16: Three-phase evolution strategy

2. ✅ **Phase 1: MVP - Rapid Validation**
   - Line 19-58: Current state (Streamlit monolith)
   - Tech stack: Streamlit, OpenAI GPT-4, Pinecone, Docker
   - Accepted trade-offs documented
   - Status: Complete (January 2025)

3. ✅ **Phase 2: Production Scale**
   - Line 61-101: Target Q2 2025
   - Tech stack: React + Next.js, FastAPI, LangChain
   - Scalability improvements: Micro-frontends, API gateway, mobile-first

4. ✅ **Phase 3: Enterprise-Grade**
   - Line 104-143: Future vision (2026+)
   - Tech stack: CloudFlare, Redis, DataDog, Airflow
   - Enterprise capabilities: HA, observability, optimization

5. ✅ **Strategic Rationale**
   - Line 146-218: MVP-first thinking, core IP preservation, trade-off awareness
   - Business alignment for each phase

**Cross-references:**
- Referenced by: `index.md`, `04-building-mattgpt.md` (roadmap section)
- References: Product vision doc
- Related: `/wireframes/architecture_evolution_slide_wireframe.html`

---

### `/docs/03-ux-design-process.md` ✅ COMPLETE

**Status:** Complete - extracted from PDF slides 9-26

**Sections:**
1. ✅ **Site Architecture & Information Hierarchy**
   - Line 13-94: Full site structure tree
   - Homepage → Industry landings → Explore Stories → Detail view

2. ✅ **User Navigation Flows** - 6 distinct flows
   - Line 96-187:
     - Flow 1: Banking Browse-First User
     - Flow 2: Search-First User
     - Flow 3: Capability-First User
     - Flow 4: Conversational User
     - Flow 5: Cross-Industry User
     - Flow 6: Direct Search
   - ❌ **MISSING:** Visual diagrams for these flows (text-only currently)

3. ✅ **Homepage Starter Cards**
   - Line 189-225: Mapping of each card to destination

4. ✅ **Search Pipeline Architecture**
   - Line 227-307: Query flow, architecture components, retrieval details
   - Hybrid search: 80% semantic + 20% keyword

5. ✅ **View Specifications** - Complete wireframe annotations
   - Line 309-655: Homepage, Banking Landing, Cross-Industry Landing
   - Line 657-794: Explore Stories (Table/Card/Timeline views)
   - Line 796-881: Detail View with STAR display
   - Line 883-1017: Ask MattGPT (Landing, Expanded, Conversation)
   - Line 1019-1168: About Matt page

6. ✅ **Design System Guidelines**
   - Line 1170-1263: Colors, typography, spacing, components, accessibility

**Outstanding Work:**
- ❌ **Visual wireflow diagrams** - Currently text-only, need visual Mermaid/Figma diagrams
- ❌ **Wireflow mapping section** - Should add visual diagrams for the 6 user flows

**Cross-references:**
- Referenced by: `index.md`
- References: All wireframe HTML files
- Related: `/components/sitemap_navigation.md` (overlaps with flows)

---

### `/docs/04-building-mattgpt.md` ⚠️ PARTIAL

**Status:** Technical content complete, personal narrative sections have PLACEHOLDERS

**Complete Sections:**
1. ✅ **The Problem** - Line 13-59
2. ✅ **Technical Implementation** - Line 87-246
   - Tech stack table
   - System architecture diagrams
   - Hybrid search algorithm (Python code example)
   - Data governance explanation
3. ✅ **What MattGPT Can Do** - Line 248-389
   - LLM-powered capabilities
   - Hybrid retrieval strategy
   - Frontend/backend features
4. ✅ **Product Roadmap** - Line 438-568
   - NOW/NEXT/LATER framework
   - Detailed feature lists for each phase

**Incomplete Sections (PLACEHOLDERS):**
1. ❌ **Why I Built This** - Line 61-85
   - `[PLACEHOLDER: Personal Narrative Section]`
   - Needs: What triggered the decision, previous attempts, specific scenarios, personal motivation, timeline

2. ❌ **Key Challenges & Solutions** - Line 391-436
   - `[PLACEHOLDER: Technical Challenges Section]`
   - Needs: Hardest technical problems, data quality handling, performance optimization, edge cases, trade-offs

3. ❌ **Lessons Learned** - Line 570-608
   - `[PLACEHOLDER: Reflections & Insights Section]`
   - Needs: What surprised you, advice for others, technical/product/business lessons

4. ❌ **Future Vision** - Line 610-648
   - `[PLACEHOLDER: Long-Term Vision Section]`
   - Needs: 3-5 year outlook, productization potential, adjacent problems, career goals

5. ❌ **Build Timeline** - Line 650-690
   - `[PLACEHOLDER: Project Timeline Section]`
   - Needs: Week-by-week breakdown, milestones, pivot points, time allocation, first user feedback

**Action Required:**
- Matt needs to fill in these 5 placeholder sections with personal narrative
- Technical content serves as scaffolding, personal story provides authenticity

**Cross-references:**
- Referenced by: `index.md`
- References: Technical architecture doc (roadmap), UX doc (design decisions)

---

## Wireframe Files (Interactive HTML)

### `/wireframes/` - 9 HTML files ✅ ALL COMPLETE

**Status:** All wireframes functional and deployed

1. ✅ `homepage_wireframe.html` - Entry point with starter cards
2. ✅ `banking_landing_page_wireframe.html` - Industry-specific view (55 projects, 16 capabilities)
3. ✅ `cross_industry_landing_page_wireframe.html` - Transformation patterns across sectors
4. ✅ `explore_stories_table_view_wireframe.html` - High-density browsing for recruiters
5. ✅ `explore_stories_card_view_wireframe.html` - Visual story previews
6. ✅ `explore_stories_timeline_view_wireframe.html` - Chronological career progression
7. ✅ `explore_stories_detail_view_wireframe.html` - Full STAR story with metrics
8. ✅ `ask_mattgpt_landing_wireframe.html` - Conversational search entry
9. ✅ `ask_mattgpt_conversation_wireframe.html` - Live chat with source citations
10. ✅ `about_matt_wireframe.html` - Career journey, competencies, leadership philosophy
11. ✅ `architecture_evolution_slide_wireframe.html` - MVP → Production → Enterprise roadmap

**Cross-references:**
- All linked from: `index.md`
- All annotated in: `/docs/03-ux-design-process.md`
- Supporting data: `/data/wireframe_annotations_all.csv`

**Outstanding Work:**
- ❌ **Wireflow diagrams** showing connections between these screens

---

## Component Library Files

### `/components/component_inventory.md` ✅ COMPLETE

**Status:** Complete component catalog

**Content:** (19KB file)
- Detailed specifications for all reusable UI components
- Button styles, form elements, navigation patterns, cards, etc.

**Cross-references:**
- Referenced by: `index.md`
- Supports: All wireframes

---

### `/components/sitemap_navigation.md` ✅ COMPLETE (for app, not doc site)

**Status:** Complete for MattGPT app navigation

**Content:** (8KB file, lines 1-281)
- Site hierarchy for the MattGPT app
- Global navigation components
- Page-to-page flows
- 3 user journey maps (Recruiter, Technical Deep Dive, Quick Reference)
- Cross-linking strategy
- Navigation UX principles
- Analytics tracking recommendations
- Mobile navigation considerations
- Implementation checklist

**Important Note:** This documents navigation for the **MattGPT app itself**, NOT for this design spec documentation site.

**Outstanding Work:**
- ❌ Could create separate navigation doc for the design spec site (lower priority)

**Cross-references:**
- Referenced by: `index.md`
- Related to: `/docs/03-ux-design-process.md` (user flows section overlaps)

---

## Brand Assets

### `/brand-kit/brand_kit_preview.html` ✅ COMPLETE

**Status:** Complete visual brand guide (9MB HTML file)

**Content:**
- Logo variations and usage guidelines
- Color palette with hex codes
- Typography specifications
- Icon library
- Visual examples

**Cross-references:**
- Referenced by: `index.md`
- Assets used in: All wireframes, documentation

---

### `/brand-kit/` subfolders ✅ COMPLETE

- `/brand-kit/logos/` - Multiple logo variations (Agy character, full logo, etc.)
- `/brand-kit/chat_avatars/` - User and AI avatars for chat interface
- `/brand-kit/favicons/` - Favicon variants for different platforms
- `/brand-kit/hero/` - Hero images
- `/brand-kit/svg/` - SVG assets
- `/brand-kit/thinking_indicator/` - Loading animation frames

---

## Image Assets

### `/images/logos/` ✅ COMPLETE

**Files:**
1. `agy_transparent.png` - Transparent background version
2. `agy_character.png` - Just the dog character
3. `agy_hero.png` - Hero version (used as site logo in `_config.yml`)
4. `agy_full.png` - Full logo with text
5. `full_logo_horizontal_glow.png` - Logo with glow effect

**Usage:**
- `agy_hero.png` is the site logo (configured in `_config.yml` line 14)

---

### `/images/architecture/` ✅ COMPLETE

**Files:**
9 SVG architecture diagrams including:
- `rag_workflow.svg` - RAG pipeline visualization
- `site_architecture.svg` - Information architecture diagram
- Various technical flow diagrams

**Cross-references:**
- Referenced by: `index.md` (Design Artifacts section)
- Related to: `/docs/02-technical-architecture.md`

---

### `/images/screenshots/` ✅ COMPLETE

**Files:**
9 PNG screenshots of the Streamlit MVP app

**Cross-references:**
- Could be referenced in `/docs/04-building-mattgpt.md` (not currently linked)

---

### `/images/wireframes/` ✅ COMPLETE

**Files:**
1 wireframe mockup image

---

### `/images/ui-elements/` ✅ COMPLETE

**Files:**
10 PNG files - animation frames for loading indicators

---

## Data Files

### `/data/wireframe_annotations_all.csv` ✅ COMPLETE

**Status:** Complete (34KB CSV file)

**Content:**
- Detailed annotations for all wireframe components
- Spreadsheet format with component IDs, descriptions, interactions

**Cross-references:**
- Supports: All wireframe HTML files
- Related to: `/docs/03-ux-design-process.md` (wireframe specs)

---

## Configuration Files

### `/_config.yml` ✅ COMPLETE

**Status:** Complete Jekyll configuration

**Key Settings:**
- Line 11: `theme: jekyll-theme-minimal`
- Line 14: `logo: /mattgpt-design-spec/images/logos/agy_hero.png`
- Line 7-8: `baseurl` and `url` for GitHub Pages
- Line 82-91: `keep_files` and `include` rules for HTML wireframes

**Cross-references:**
- Controls: How GitHub Pages builds the site
- Affects: All page rendering and navigation

---

### `/index.md` ✅ COMPLETE

**Status:** Complete landing page (6.5KB file)

**Sections:**
1. ✅ What is MattGPT - Line 13-22
2. ✅ Why This Matters - Line 26-34
3. ✅ Documentation links - Line 38-54
   - Links to all 4 core docs
4. ✅ Interactive Wireframes - Line 58-78
   - Links to all 11 wireframe HTML files
5. ✅ Design System & Components - Line 82-91
   - Links to component inventory, sitemap, brand guide
6. ✅ Design Artifacts - Line 95-102
   - Links to architecture diagrams, brand assets
7. ✅ Key Concepts - Line 106-128
8. ✅ What This Demonstrates - Line 132-149
9. ✅ Live Application - Line 153-157
10. ✅ Contact - Line 161-168

**All links use absolute paths with `/mattgpt-design-spec/` prefix**

**Cross-references:**
- Hub for entire site
- Links to: All docs, all wireframes, all components

---

## Reference Files

### `/mattgpt_site_architecture.pdf` ✅ COMPLETE

**Status:** Source material (2.8MB, 29 pages)

**Purpose:**
- Original PowerPoint slides exported as PDF
- Source for all documentation content
- Reference for visual design decisions

**Content extracted to:**
- Slides 3-7 → `/docs/01-product-vision.md`
- Slide 8 → `/docs/02-technical-architecture.md` (partial)
- Slides 9-26 → `/docs/03-ux-design-process.md`
- Slide 25, 29 → `/docs/04-building-mattgpt.md` (partial)

---

### `/PROGRESS.md` ✅ COMPLETE

**Status:** High-level roadmap (kept up-to-date)

**Purpose:**
- Big-picture project status
- Major milestones and phases
- Backlog items

**Update frequency:** End of major milestones

---

### `/CONTEXT.md` ✅ COMPLETE

**Status:** Session state snapshot (this file should be updated frequently)

**Purpose:**
- Current working state
- Outstanding work
- Known issues and resolutions
- Resume protocol for context loss

**Update frequency:** End of each major work session

---

### `/MAPPING.md` ✅ THIS FILE

**Purpose:**
- Detailed file-by-file, section-by-section inventory
- Status indicators (complete vs incomplete)
- Cross-references between files
- Outstanding work at granular level

**Update frequency:** When files change or new sections are added

---

## Outstanding Work (Detailed)

### Priority 1: Visual Wireflow Diagrams ❌

**What's needed:**
- Visual diagrams showing screen-to-screen transitions for the 6 user flows documented in `/docs/03-ux-design-process.md` (lines 96-187)

**Specific flows to diagram:**
1. Banking Browse-First User (Homepage → Banking Landing → Categories → Stories → Detail)
2. Search-First User (Homepage → Search → Results → Transparency Banner → View All)
3. Capability-First User (Homepage → Capability Filter → Cross-Industry Results → Detail)
4. Conversational User (Homepage → Ask MattGPT → Response + Sources → Detail → Pivot)
5. Cross-Industry User (Homepage → Cross-Industry Landing → Categories → Stories → Detail)
6. Direct Search (Homepage → Search → Complete Results → Browse/Filter)

**Format options:**
- Mermaid.js diagrams (text-based, embeddable in markdown)
- Figma prototypes with flow arrows (export as images)
- Miro/Mural flowcharts (export as images)
- Hand-drawn diagrams (scan as images)

**Where to add:**
- Create new folder: `/images/wireflows/`
- Add visual diagrams (SVG or PNG)
- Update `/docs/03-ux-design-process.md` to embed these diagrams in section 2
- Link from `index.md` in Design Artifacts section

**Status:** Not started

---

### Priority 2: Personal Narrative Sections ❌

**File:** `/docs/04-building-mattgpt.md`

**Specific sections needing Matt's input:**

1. **Why I Built This** (Line 61-85)
   - What triggered the decision?
   - Previous portfolio frustrations?
   - Specific job search scenarios?
   - Personal motivation beyond professional need?
   - When did you start? Initial scope?

2. **Key Challenges & Solutions** (Line 391-436)
   - Hardest technical problems encountered?
   - How did you handle data quality/consistency?
   - Performance optimization stories?
   - Edge cases that broke the system?
   - MVP shortcuts and trade-offs?

3. **Lessons Learned** (Line 570-608)
   - What surprised you most?
   - Advice for someone starting similar project?
   - Technical lessons (libraries, patterns, anti-patterns)?
   - Product lessons (user feedback, features)?
   - Business lessons (positioning, value prop)?

4. **Future Vision** (Line 610-648)
   - Where do you see this in 3-5 years?
   - Could this become a product for other consultants?
   - What adjacent problems could this solve?
   - How might AI/LLM evolution change the roadmap?
   - How does this fit into your broader career trajectory?

5. **Build Timeline** (Line 650-690)
   - Week-by-week breakdown of initial build
   - Key milestones and pivot points
   - Time spent on different phases (data vs code vs testing)
   - When did you get first user feedback?
   - Major version releases and feature additions

**Status:** Awaiting Matt's input

---

### Priority 3: Test All Links on Live Site ❌

**What to test:**
1. All documentation links from `index.md`
2. All wireframe links (ensure no 404s)
3. All internal cross-references between docs
4. All image links (logos, diagrams)
5. All external links (LinkedIn, email, Streamlit app)

**How to test:**
- Manual click-through on https://mcpugmire1.github.io/mattgpt-design-spec/
- Check browser console for 404 errors
- Verify all images load
- Test on mobile/tablet viewports

**Status:** Not started (waiting for wireflow completion and personal narrative)

---

### Nice-to-Have: Custom CSS for Minimal Theme 🎨

**What's needed:**
- Create `/assets/css/style.scss`
- Apply brand colors from `/brand-kit/`
- Style navigation, links, headings with brand palette
- Add subtle gradients or accents
- Maintain clean minimal structure

**Why nice-to-have:**
- Content is complete and functional
- Minimal theme works but is very plain
- Branding would make it more polished
- Not blocking any other work

**Status:** Deferred (lower priority than content completion)

---

## Cross-Reference Matrix

| File | References (outbound) | Referenced By (inbound) |
|------|---------------------|------------------------|
| `index.md` | All docs, wireframes, components | None (it's the root) |
| `01-product-vision.md` | `02-technical-architecture.md` | `index.md`, `03-ux-design-process.md` |
| `02-technical-architecture.md` | `01-product-vision.md` | `index.md`, `04-building-mattgpt.md` |
| `03-ux-design-process.md` | All wireframes, `component_inventory.md` | `index.md` |
| `04-building-mattgpt.md` | `02-technical-architecture.md`, `03-ux-design-process.md` | `index.md` |
| All wireframes | None (static HTML) | `index.md`, `03-ux-design-process.md` |
| `component_inventory.md` | None | `index.md`, wireframes |
| `sitemap_navigation.md` | None | `index.md` |
| `brand_kit_preview.html` | All brand assets | `index.md` |
| `_config.yml` | `agy_hero.png` | Controls all Jekyll builds |

---

## File Dependency Graph

```
index.md (ROOT)
├── docs/
│   ├── 01-product-vision.md ✅
│   ├── 02-technical-architecture.md ✅
│   ├── 03-ux-design-process.md ✅ (missing visual wireflows)
│   └── 04-building-mattgpt.md ⚠️ (missing personal narrative)
│
├── wireframes/ ✅
│   ├── [11 HTML files - all complete]
│   └── ❌ MISSING: Visual flow diagrams connecting these screens
│
├── components/ ✅
│   ├── component_inventory.md
│   └── sitemap_navigation.md
│
├── brand-kit/ ✅
│   └── brand_kit_preview.html
│       └── [All brand assets]
│
└── images/ ✅
    ├── logos/ (used by _config.yml)
    ├── architecture/
    ├── screenshots/
    ├── wireframes/
    ├── ui-elements/
    └── ❌ MISSING: /wireflows/ folder
```

---

## How to Use This File

**When Claude Code crashes and you restart:**

1. I read `CONTEXT.md` first (high-level state)
2. I read this file (`MAPPING.md`) for detailed inventory
3. I check specific files mentioned in your prompt
4. I ask clarifying questions before making changes

**When you make progress:**

1. Update the relevant section in this file with ✅ or ❌ status
2. Update line number references if sections move
3. Update "Outstanding Work" section
4. Commit with descriptive message

**When adding new files:**

1. Add entry to relevant section
2. Document what it contains (section-by-section)
3. Note cross-references
4. Mark status (complete/partial/placeholder)

---

*This file should be updated whenever file structure changes or major sections are completed/added.*
