# MattGPT: Product Design & Strategy Documentation

**Professional design documentation for MattGPT - An AI-powered career portfolio assistant**

> This repo contains comprehensive product strategy, technical architecture, and UX design documentation for MattGPT, demonstrating end-to-end product thinking from concept through implementation.

---

## 🎯 What is MattGPT?

MattGPT is an **AI-powered portfolio interface** that eliminates doubt about a candidate's experience by acting as a **credibility engine**. Instead of making claims about skills and experience, it provides **proof through structured stories** using the STAR methodology (Situation, Task, Action, Result).

**The Problem:** Traditional portfolios rely on self-promotion and claims that recruiters must verify.

**The Solution:** An AI assistant trained on structured career stories that can:
- Answer specific questions about experience with concrete examples
- Provide metrics and outcomes from real projects
- Demonstrate technical depth through searchable, verified content
- Scale personal credibility to multiple conversations simultaneously

---

## 🔗 Quick Links

| Resource | Description |
|----------|-------------|
| 🚀 [**Live App**](https://askmattgpt.streamlit.app/) | Try the deployed application |
| 💻 [**Source Code**](https://github.com/mcpugmire1/llm_portfolio_assistant) | View the implementation |
| 🎨 [**Interactive Prototypes**](wireframes/) | Explore clickable wireframes |
| 📋 [**Project Context**](CONTEXT.md) | Current status, completed work, and next steps |

---

## 📚 Documentation Structure

This repo organizes design documentation into four key areas:

### 1. Product Strategy & Vision
- **[01-product-vision.md](docs/01-product-vision.md)**
  - The Credibility Engine: WHY/HOW/WHAT framework
  - Target user personas (Recruiter, Hiring Manager, Content User)
  - Product scope & boundaries
  - Two-layer governance model (STAR + Tagging)
  - AI system prompt and integrity guidelines

### 2. Technical Architecture
- **[02-technical-architecture.md](docs/02-technical-architecture.md)**
  - RAG (Retrieval-Augmented Generation) pipeline
  - 3-phase evolution: MVP → Production → Enterprise
  - Hybrid search architecture (semantic + keyword)
  - Technology stack decisions and migration plan

### 3. UX Design & Wireframes
- **[03-ux-design-process.md](docs/03-ux-design-process.md)**
  - Design principles and site architecture
  - User journey diagrams (6 flows)
  - Navigation architecture and filter specifications
  - Component specifications with complete implementation details

### 4. Development Journey
- **[04-building-mattgpt.md](docs/04-building-mattgpt.md)**
  - The challenge: Better portfolio presentation
  - What I learned building this (RAG, Python, LLMs)
  - Technical innovations and decisions
  - Roadmap: Phase 1 polish → Phase 2 React rebuild

### 5. Brand Voice Guide
- **[05-agy-voice-guide.md](docs/05-agy-voice-guide.md)**
  - Agy personality traits (Plott Hound AI assistant)
  - Voice principles and response templates
  - Tone calibration and situational responses

---

## 🎨 Design Artifacts

### Interactive Prototypes
Working HTML wireframes demonstrating key user flows:

| Wireframe | Description |
|-----------|-------------|
| [index.html](wireframes/index.html) | Main landing page |
| [about_matt_wireframe.html](wireframes/about_matt_wireframe.html) | About section |
| [ask_mattgpt_wireframe.html](wireframes/ask_mattgpt_wireframe.html) | AI chat interface |
| [ask_mattgpt_landing_wireframe.html](wireframes/ask_mattgpt_landing_wireframe.html) | Chat landing page |
| [explore_stories_cards_wireframe.html](wireframes/explore_stories_cards_wireframe.html) | Story browsing (card view) |
| [explore_stories_table_wireframe.html](wireframes/explore_stories_table_wireframe.html) | Story browsing (table view) |
| [explore_stories_timeline_wireframe.html](wireframes/explore_stories_timeline_wireframe.html) | Story browsing (timeline view) |
| [explore_stories_mobile_wireframe.html](wireframes/explore_stories_mobile_wireframe.html) | Mobile-optimized story view |
| [banking_landing_page.html](wireframes/banking_landing_page.html) | Industry-specific landing demo |

### Component Specifications
- **[Component Library](components/component_inventory.md)** - Detailed component specifications
- **[Site Navigation Flow](components/sitemap_navigation.md)** - Complete navigation structure and user journeys
- **[Annotation Specifications](data/wireframe_annotations_all.csv)** - 190+ rows of wireframe annotations

---

## 🧠 Key Concepts

**STAR Framework:** Structured storytelling methodology (Situation, Task, Action, Result) that ensures credibility through concrete examples.

**RAG Architecture:** Retrieval-Augmented Generation - AI technique that grounds responses in actual source documents rather than generating claims.

**The Credibility Engine:** Core product positioning - proof over claims, verification over self-promotion.

**Two-Layer Governance:**
- **Layer 1:** STAR methodology (ensures integrity and structure)
- **Layer 2:** Intelligent tagging (enables AI-powered search and filtering)

---

## 📂 Repository Structure

```
mattgpt-design-spec/
├── README.md                   # This file - project overview
├── CONTEXT.md                  # Current project status and session state
│
├── /docs/                      # Strategic documentation
│   ├── 01-product-vision.md
│   ├── 02-technical-architecture.md
│   ├── 03-ux-design-process.md
│   ├── 04-building-mattgpt.md
│   └── 05-agy-voice-guide.md
│
├── /wireframes/                # Interactive HTML prototypes
│   └── [9 working wireframes]
│
├── /components/                # Component specifications
│   ├── component_inventory.md
│   └── sitemap_navigation.md
│
├── /images/                    # Diagrams and visual assets
│   ├── architecture/          # RAG diagrams, site architecture
│   ├── wireflows/             # User journey diagrams (SVG)
│   ├── logos/                 # Agy brand assets
│   └── screenshots/           # App screenshots
│
├── /brand-kit/                 # Complete brand guidelines
│   └── [logos, colors, typography]
│
├── /data/                      # Structured data & annotations
│   └── wireframe_annotations_all.csv
│
└── /assets/                    # GitHub Pages styling
    └── css/style.scss         # Custom brand colors
```

---

## 📱 Mobile & Responsive Design

MattGPT includes production-quality mobile CSS with:
- **Breakpoints:** 767px (mobile), 768-1024px (tablet), 1024+ (desktop)
- **Mobile-First Patterns:** Stacking layouts, touch-optimized controls
- **Responsive Tables:** Horizontal scroll with preserved functionality
- **Adaptive Navigation:** Hamburger menu, collapsible filters

---

## 🚀 What This Demonstrates

This project showcases:

1. **Product Thinking:** Strategic vision → technical architecture → user experience
2. **AI/ML Application:** Practical implementation of RAG with vector search
3. **Full-Stack Development:** Python backend, Streamlit frontend, deployment pipeline
4. **UX Design:** User research → wireframes → prototypes → implementation
5. **Documentation:** Professional-grade technical and product documentation
6. **Problem Solving:** Innovative solution to portfolio credibility challenge

---

## 📝 Project Status

**Current Phase:** Streamlit Production Polish (December 2024)

See **[CONTEXT.md](CONTEXT.md)** for detailed project status, completed work, and next steps.

**Completed Since October 2024:**
- ✅ Timeline View with Era-based grouping
- ✅ Mobile-responsive CSS (breakpoints at 767px, 1024px)
- ✅ Modular Ask MattGPT architecture (9-file structure)
- ✅ Related Projects UX pattern
- ✅ Dark mode support via CSS variables
- ✅ Conversation history and context management
- ✅ Filter redesign (Phase 4)
- ✅ All 5 documentation files complete and published
- ✅ GitHub Pages live with custom brand styling
- ✅ All 9 wireframes deployed and accessible
- ✅ Architecture diagrams aligned with PDF design spec
- ✅ Comprehensive wireframe audit (100% accuracy)
- ✅ Agy Voice Guide integrated
- ✅ User journey diagrams (high-level navigation paths)

**Outstanding:**
- 🔄 Design spec alignment (addressed December 2024)
- 🔄 React migration planning (Phase 2)

**Next Steps:**
- Polish Streamlit MVP for immediate job search
- Plan Phase 2: React modern architecture rebuild

---

## 💡 Why This Approach?

Traditional portfolios rely on candidates making claims that recruiters must verify. MattGPT inverts this:

- **Instead of claiming expertise** → Provide searchable, structured proof
- **Instead of bullet points** → Deliver STAR-formatted stories with metrics
- **Instead of static resume** → Offer interactive AI interface
- **Instead of one-way communication** → Enable dynamic Q&A at scale

The result: **Credibility through demonstration, not declaration.**

---

## 🔗 Related Projects

- **[llm_portfolio_assistant](https://github.com/mcpugmire1/llm_portfolio_assistant)** - Source code for MattGPT application
- **[askmattgpt.streamlit.app](https://askmattgpt.streamlit.app/)** - Live deployed application

---

*Built by Matthew Pugmire | [LinkedIn](#) | [Portfolio](#)*

---

**Last Updated:** December 20, 2024 (Post-Audit Refresh)
**Version:** 2.0 (Documentation Updated - Implementation Aligned)