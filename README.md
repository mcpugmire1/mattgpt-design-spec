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
| 📊 [**Progress Log**](PROGRESS.md) | See development timeline and decisions |

---

## 📚 Documentation Structure

This repo organizes design documentation into four key areas:

### 1. Product Strategy & Vision
- **[01-product-vision.md](docs/01-product-vision.md)** *(Coming Soon)*
  - The Credibility Engine: WHY/HOW/WHAT framework
  - Target user personas (Recruiter, Hiring Manager, Content User)
  - Product scope & boundaries
  - Two-layer governance model (STAR + Tagging)

### 2. Technical Architecture
- **[02-technical-architecture.md](docs/02-technical-architecture.md)** *(Coming Soon)*
  - RAG (Retrieval-Augmented Generation) pipeline
  - Four-phase system: Ingestion → Storage → Retrieval → Delivery
  - Hybrid search architecture (semantic + keyword)
  - Technology stack decisions

### 3. UX Design & Wireframes
- **[03-ux-design-process.md](docs/03-ux-design-process.md)** *(Coming Soon)*
  - Design principles and process
  - User journey mapping
  - Navigation architecture
  - Component specifications

### 4. Development Journey
- **[04-building-mattgpt.md](docs/04-building-mattgpt.md)** *(Coming Soon)*
  - The challenge: Better portfolio presentation
  - What I learned building this (RAG, Python, LLMs)
  - Technical innovations and decisions
  - Next phase: React refactor

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
- **[Component Library](components/)** - Detailed component specifications *(Coming Soon)*
- **[Site Navigation Flow](components/)** - Complete navigation structure *(Coming Soon)*
- **[Annotation Specifications](data/)** - Detailed wireframe annotations *(Coming Soon)*

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
├── PROGRESS.md                 # Development log and session continuity
│
├── /docs/                      # Strategic documentation
│   ├── 01-product-vision.md
│   ├── 02-technical-architecture.md
│   ├── 03-ux-design-process.md
│   └── 04-building-mattgpt.md
│
├── /wireframes/                # Interactive HTML prototypes
│   └── [9 working wireframes]
│
├── /components/                # Component specifications
│   ├── component_inventory.md
│   └── sitemap_navigation.md
│
├── /data/                      # Structured data & annotations
│   └── wireframe_annotations_all.csv
│
└── /images/                    # Diagrams and screenshots
```

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

**Current Phase:** Initial Setup & Content Migration

See [PROGRESS.md](PROGRESS.md) for detailed task tracking, session logs, and development decisions.

**Next Steps:**
- Extract content from design specification slides
- Complete documentation files in `/docs/`
- Enable GitHub Pages for business-friendly presentation
- Cross-link with code repository

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

**Last Updated:** October 2024
**Version:** 1.0 (Initial Documentation Setup)