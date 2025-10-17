# MattGPT Design Spec - Progress Log

**Last Updated:** [Today's Date]  
**Current Phase:** Initial Setup & Content Migration  
**Project Goal:** Create professional design documentation for MattGPT portfolio project

---

## 🎯 CURRENT STATE (Start Here)

**What I'm working on right now:**
Organizing design documentation from PPT into a structured, maintainable format with Markdown source and business-friendly outputs.

**Immediate next step:**
Work with Claude Code to extract PPT slides 3-12 into Markdown files and set up GitHub Pages.

**Blockers:**
None currently - ready to execute.

---

## ✅ COMPLETED

### Initial Analysis & Planning
- [x] Identified problem: Weeks of work trapped in PowerPoint
- [x] Decided on Markdown + GitHub Pages approach
- [x] Confirmed existing assets: HTML wireframes, MD files, CSV annotations
- [x] Planned repo structure
- [x] Created this PROGRESS.md file

### Existing Assets Inventory
- [x] Working Streamlit app (deployed at askmattgpt.streamlit.app)
- [x] GitHub repo with code: llm_portfolio_assistant
- [x] 7+ HTML wireframe prototypes (clickable, working)
- [x] wireframe_component_inventory.md (detailed component specs)
- [x] wireframe_sitemap_navigation.md (complete nav structure)
- [x] wireframe_annotations_all.csv (190 rows of specs)
- [x] banking_landing_page.html (working prototype)
- [x] cross_industry_landing_page.html (working prototype)

---

## 📁 TARGET REPO STRUCTURE
```
mattgpt-design-spec/
├── README.md (master overview with links to everything)
├── PROGRESS.md (this file - session continuity)
│
├── /docs/
│   ├── 01-product-vision.md (extract from PPT slides 3-7)
│   │   - The Credibility Engine (WHY/HOW/WHAT)
│   │   - Target user personas (Recruiter, Hiring Manager, Content User)
│   │   - Product scope & boundaries
│   │
│   ├── 02-technical-architecture.md (extract from PPT slides 8-12)
│   │   - Two-layer governance model
│   │   - RAG pipeline architecture
│   │   - Tech stack decisions
│   │   - Search pipeline details
│   │
│   ├── 03-ux-design-process.md (reference existing files)
│   │   - Link to wireframes
│   │   - Link to component inventory
│   │   - Link to sitemap/navigation
│   │   - Design principles
│   │
│   └── 04-building-mattgpt.md (NEW - my story)
│       - The problem: Need better portfolio presentation
│       - The solution: AI-powered interface
│       - What I learned (RAG, Python, LLMs)
│       - Technical innovations
│       - Next phase: React refactor
│
├── /wireframes/ (move HTML prototypes here)
│   ├── index.md (catalog/index of all wireframes)
│   ├── index.html
│   ├── about_matt_wireframe.html
│   ├── banking_landing_page.html
│   ├── cross_industry_landing_page.html
│   ├── explore_stories_cards_wireframe.html
│   ├── explore_stories_table_wireframe.html
│   ├── explore_stories_timeline_wireframe.html
│   ├── ask_mattgpt_landing_wireframe.html
│   └── ask_mattgpt_wireframe.html
│
├── /components/ (move existing MD files here)
│   ├── component_inventory.md
│   └── sitemap_navigation.md
│
├── /data/
│   └── wireframe_annotations_all.csv
│
└── /images/ (if needed for diagrams)
    └── [architecture diagrams, screenshots]
```

---

## 🧠 KEY DECISIONS MADE (Don't Re-Debate)

| Decision | Rationale | Date |
|----------|-----------|------|
| Use Markdown as source of truth | Developer currency, version control, multiple outputs possible | Today |
| GitHub Pages for business-friendly output | Free, automatic, professional presentation | Today |
| Keep HTML wireframes (don't migrate to Figma) | Already done, working, clickable - no need to redo | Today |
| No personal backstory about Accenture in public docs | Keep professional tone, avoid raising questions about separation | Today |
| Two repos: code repo + design spec repo | Separate concerns: working app vs. documentation | Today |
| Markdown → multiple outputs (GitHub Pages, Notion, PDF) | Write once, output many formats for different audiences | Today |

---

## 📝 CONTENT TO EXTRACT FROM PPT

### PPT Slides 3-7 → 01-product-vision.md
- **Slide 3:** The Credibility Engine (WHY/HOW/WHAT framework)
  - Eliminate Doubt (the problem)
  - Act as Credibility Engine (the approach)
  - Specific Outcomes (the result)
  
- **Slide 4:** Target User Personas
  - Recruiter (speed, scalability, filtering)
  - Hiring Manager (depth, structure, metrics)
  - Content User / Matt (quick retrieval, synthesis)
  
- **Slide 5:** Data Model & Two-Layer Governance
  - Layer 1: Mandatory STAR Method (integrity)
  - Layer 2: Tagging Systems (AI intelligence)
  
- **Slide 6:** AI Core System Prompt & Integrity Mandate
  - Purpose as Credibility Engine
  - Core directives
  - Guardrails
  
- **Slide 7:** Project Scope & Boundaries (MVP definition)
  - Must Have vs. Must Never Do
  - In Scope vs. Out of Scope

### PPT Slides 8-12 → 02-technical-architecture.md
- **Slide 8:** Technology & RAG Architecture
  - Four-phase pipeline (Ingestion, Storage, Retrieval, Delivery)
  - Production stack choices
  
- **Slide 9:** Site Architecture & Information Flow
  - Page hierarchy
  - Navigation structure
  
- **Slide 10:** Key User Navigation & Menus
  - Six user journeys mapped
  
- **Slide 11:** Homepage Starter Card Specification
  - Entry points and routing
  
- **Slide 12:** MattGPT Search Pipeline
  - Hybrid search (semantic + keyword)
  - Response synthesis modes

### Slides 13-26: Wireframe Specs
**Status:** Already documented via:
- HTML prototypes (interactive)
- wireframe_annotations_all.csv (190 rows)
- component_inventory.md (detailed specs)
- sitemap_navigation.md (flows)

**Action:** Reference these existing files, don't duplicate.

---

## 🚧 CURRENT TASKS

### Phase 1: Setup & Organization (Target: This Weekend)
- [ ] Create GitHub repo: `mattgpt-design-spec`
- [ ] Set up folder structure (as outlined above)
- [ ] Move existing files:
  - [ ] wireframe_component_inventory.md → /components/
  - [ ] wireframe_sitemap_navigation.md → /components/
  - [ ] wireframe_annotations_all.csv → /data/
  - [ ] All HTML files → /wireframes/
- [ ] Create /wireframes/index.md (catalog of prototypes)
- [ ] Commit initial structure

### Phase 2: Content Extraction (Target: This Weekend)
- [ ] Extract PPT slides 3-7 → /docs/01-product-vision.md
- [ ] Extract PPT slides 8-12 → /docs/02-technical-architecture.md
- [ ] Create /docs/03-ux-design-process.md (mostly links to existing files)
- [ ] Write /docs/04-building-mattgpt.md (my story - NEW content)
- [ ] Commit all docs

### Phase 3: Master README (Target: This Weekend)
- [ ] Write README.md with:
  - [ ] Project overview
  - [ ] Links to live app + code repo
  - [ ] Links to all docs
  - [ ] Links to wireframes
  - [ ] Quick navigation
- [ ] Commit README

### Phase 4: Business-Friendly Output (Target: Next Week)
- [ ] Enable GitHub Pages (Settings → Pages → Enable)
- [ ] Choose clean theme
- [ ] Review output at mcpugmire1.github.io/mattgpt-design-spec
- [ ] Test all links work
- [ ] Adjust formatting if needed

### Phase 5: Integration (Target: Next Week)
- [ ] Update llm_portfolio_assistant/README.md
  - [ ] Add link to design spec repo
  - [ ] Add "Product Strategy" section
  - [ ] Add "What This Demonstrates" section
- [ ] Link both repos to each other
- [ ] Declare v1.0 shipped

### Phase 6: Promotion (Target: Next Week)
- [ ] LinkedIn post announcing MattGPT + design process
- [ ] Share with 3-5 trusted contacts for feedback
- [ ] Apply to jobs with portfolio links

---

## 🔄 SESSION LOG

### Session 1: [Date] - Discovery & Planning
**Tool:** Chat Claude  
**Duration:** ~2 hours  
**Goal:** Figure out how to finish documentation without fighting PowerPoint

**What I did:**
- Discussed tool options (PPT vs Figma vs Notion vs Markdown)
- Realized I already have HTML prototypes + MD files + CSV annotations
- Decided on Markdown → GitHub Pages approach
- Planned repo structure
- Created this PROGRESS.md

**Key insights:**
- I'm not starting over - I'm organizing what exists
- PPT was fighting me because it's the wrong tool
- MD = source of truth, GitHub Pages = beautiful output
- No need to migrate to Figma (HTML prototypes are better)
- ~5 hours of focused work to finish, not weeks

**Decisions made:**
- Use Markdown for source
- Keep HTML wireframes as-is
- Output to GitHub Pages for business presentation
- No personal backstory in public docs

**Next session:** Work with Claude Code to create repo structure and extract PPT content

---

## 🆘 CLAUDE SESSION HANDOFF

**Copy/paste this into new Claude Code sessions:**
```
I'm organizing my MattGPT design documentation. Here's my current state:

PROJECT: Portfolio documentation for MattGPT - an AI-powered career portfolio assistant I built

CONTEXT:
- I have a working Streamlit app (deployed): https://askmattgpt.streamlit.app/
- I have the code in GitHub: github.com/mcpugmire1/llm_portfolio_assistant
- I have 7+ working HTML wireframe prototypes (clickable)
- I have detailed MD documentation files already started
- I have a 26-page PPT with strategy/architecture I need to extract

CURRENT TASK:
I need to create a new repo (mattgpt-design-spec) and organize everything into:
- /docs/ - Markdown documentation extracted from PPT
- /wireframes/ - My HTML prototypes
- /components/ - Existing MD files (component specs, sitemap)
- /data/ - CSV annotations

IMMEDIATE GOAL:
[Fill in what you need help with this session]

See full progress log at: [repo-url]/PROGRESS.md

Can you help me with [specific task]?
```

---

## 📚 REFERENCE LINKS

**My Projects:**
- **Live App:** https://askmattgpt.streamlit.app/
- **App Code Repo:** https://github.com/mcpugmire1/llm_portfolio_assistant
- **Design Spec Repo:** [Will add when created]
- **GitHub Pages Site:** [Will add when enabled]

**Files to Reference:**
- Original PPT: `MattGPT Discovery & Design Spec.pptx` (26 pages, on my computer)
- Existing MD: wireframe_component_inventory.md, wireframe_sitemap_navigation.md
- CSV: wireframe_annotations_all.csv (190 rows)
- HTML: 7+ wireframe prototype files

**Key Concepts:**
- STAR Framework: Situation, Task, Action, Result (structured stories)
- RAG Architecture: Retrieval-Augmented Generation (how the AI works)
- The Credibility Engine: Core product positioning (proof over claims)
- Two-Layer Governance: STAR (integrity) + Tagging (intelligence)

---

## 💡 REMINDERS FOR FUTURE ME

**When Claude Code crashes:**
1. Check `git status` to see what was committed
2. Look at this PROGRESS.md to see what I was doing
3. Update this file with what happened
4. Start new session with CLAUDE SESSION HANDOFF context

**After each work session (even short ones):**
1. Update CURRENT STATE
2. Check off completed items
3. Add to SESSION LOG
4. Commit this file to GitHub

**Don't overthink:**
- Good enough > perfect
- Ship v1.0, iterate later
- Markdown is source, outputs are generated
- Goal is job-ready portfolio, not perfect docs

**Remember the goal:**
- Finish documentation so I can confidently show my work
- Have professional artifacts for interviews
- Move on to job hunting by next week
- Stop spinning on tools and ship

---

## 🎯 SUCCESS CRITERIA

**v1.0 = Shipped when:**
- ✅ All content from PPT is in Markdown
- ✅ GitHub Pages site is live and looks professional
- ✅ All HTML wireframes are organized and linked
- ✅ README explains the project clearly
- ✅ Both repos link to each other
- ✅ I can confidently share URLs in interviews

**Then I can:**
- Apply to jobs with these links
- Share with recruiters/hiring managers
- Use in interview conversations
- Move on to next project

---

_Last updated: [Update this after every session]_
_Next update: [When you'll work on this next]_
```

---

## **How to Use This with Claude Code**

### **Step 1: Save This File**
1. Copy everything above (the markdown between the backticks)
2. Create `PROGRESS.md` in your project folder
3. Update the date and fill in any placeholders

### **Step 2: Start Claude Code Session**
1. Open Claude Code
2. Navigate to your project folder
3. Paste this into Claude Code:
```
I'm organizing my MattGPT design documentation. I have a PROGRESS.md file that explains everything. Can you:

1. Read my PROGRESS.md file
2. Help me create the GitHub repo structure outlined in it
3. Start organizing my existing files into that structure

The file is in my current directory. Let's start by creating the folder structure.