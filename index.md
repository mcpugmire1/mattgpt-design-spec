---
layout: default
title: Home
nav_order: 0
---

# MattGPT: Design Specification

**Complete Product Blueprint for Strategy, Auditable Architecture, and Technical Execution**

---

## What is MattGPT?

MattGPT is an **AI-powered portfolio** that transforms 20 years of Fortune 500 transformation experience into an intelligent, searchable interface. Instead of static resume bullets, it provides:

- ✅ **Verifiable proof** through structured STAR stories
- ✅ **Semantic search** across 130+ real projects
- ✅ **Pattern recognition** to surface relevant examples
- ✅ **Auditable sources** for every AI-generated response

**Tagline:** *"Matt doesn't claim credibility — he proves it in real time."*

---

## Why This Matters

This isn't just a portfolio — it's a **demonstration of modern AI product development**:

- 🧠 **RAG Architecture** (Retrieval-Augmented Generation)
- 🔍 **Semantic Search** (vector-based meaning matching)
- 📊 **Data Governance** (Two-layer validation system)
- 🎨 **User-Centered Design** (Three distinct personas)
- ⚡ **Production Application** (Mobile-responsive Streamlit with Era-based timeline and advanced filtering)

---

## Documentation

### Core Strategy & Vision

**[📘 Product Vision](/mattgpt-design-spec/docs/01-product-vision)**
The Credibility Engine — WHY, HOW, WHAT framework, user personas, data governance model, and AI system prompt.

**[🏗️ Technical Architecture](/mattgpt-design-spec/docs/02-technical-architecture)**
Two-phase evolution from Streamlit MVP to React rebuild, tech stack decisions, and strategic trade-offs.

### Design & Implementation

**[🎨 UX Design Process](/mattgpt-design-spec/docs/03-ux-design-process)**
Complete site architecture, user flows, and detailed wireframe specifications for every view.

**[🔨 Building MattGPT](/mattgpt-design-spec/docs/04-building-mattgpt)**
The development journey, technical challenges, semantic search implementation, and product roadmap.

**[🐾 Agy Voice Guide](/mattgpt-design-spec/docs/05-agy-voice-guide)**
Complete brand voice documentation for Agy, the Plott Hound AI assistant — personality traits, response templates, and tone guidelines.

**[🎯 Explore Stories Filter Redesign](/mattgpt-design-spec/docs/06-explore-stories-filter-redesign)**
Phase 4 filter architecture with primary/advanced pattern, progressive disclosure, and landing page integration.

**[🎨 CSS Architecture](/mattgpt-design-spec/docs/07-css-architecture)**
Complete CSS design system with variables, breakpoints, dark mode support, and component conventions.

**[📱 Mobile Implementation](/mattgpt-design-spec/docs/08-mobile-implementation)**
Production mobile CSS with responsive breakpoints, touch optimization, and component-specific mobile behaviors.

---

## Interactive Wireframes

> **Note:** These wireframes represent the original design specification (October 2025). The live application includes additional features implemented since: Era-based Timeline View, mobile responsive design, and advanced filtering. See [askmattgpt.streamlit.app](https://askmattgpt.streamlit.app) for current state.

Explore clickable HTML prototypes of the MattGPT interface:

### Core Views
- **[Homepage](/mattgpt-design-spec/wireframes/index.html)** - Entry point with starter cards
- **[Banking Landing Page](/mattgpt-design-spec/wireframes/banking_landing_page.html)** - Financial Services projects with capability breakdown
- **[Cross-Industry Landing Page](/mattgpt-design-spec/wireframes/cross_industry_landing_page.html)** - Non-banking projects across industries

### Explore Stories Interface
- **[Table View](/mattgpt-design-spec/wireframes/explore_stories_table_wireframe.html)** - High-density browsing for recruiters
- **[Card View](/mattgpt-design-spec/wireframes/explore_stories_cards_wireframe.html)** - Visual story previews
- **[Timeline View](/mattgpt-design-spec/wireframes/explore_stories_timeline_wireframe.html)** - Era-based career progression (5 career phases)
- **[Mobile View](/mattgpt-design-spec/wireframes/explore_stories_mobile_wireframe.html)** - Mobile-optimized story browsing

### Ask MattGPT (AI Interface)
- **[Landing Page](/mattgpt-design-spec/wireframes/ask_mattgpt_landing_wireframe.html)** - Conversational search entry
- **[Conversation View](/mattgpt-design-spec/wireframes/ask_mattgpt_wireframe.html)** - Live chat with source citations

### About Matt
- **[About Matt Page](/mattgpt-design-spec/wireframes/about_matt_wireframe.html)** - Career journey, competencies, leadership philosophy

---

## Design System & Components

**[🗺️ Sitemap & Navigation](/mattgpt-design-spec/components/sitemap_navigation)**
Complete information architecture, user flows, and navigation patterns.

**[📦 Component Inventory](/mattgpt-design-spec/components/component_inventory)**
Catalog of reusable UI components with specifications.

**[🎨 Brand Guidelines](/mattgpt-design-spec/brand-kit/brand_kit_preview.html)**
Complete brand kit with logos, colors, typography, and usage guidelines.

---

## Design Artifacts

### Architecture Diagrams
- [RAG Build + Run Architecture](/mattgpt-design-spec/images/architecture/tech_rag_architecture.png) - Complete RAG lifecycle with 4-phase implementation
- [Site Architecture](/mattgpt-design-spec/images/architecture/site_architecture_updated.md) - Page hierarchy & navigation structure (December 2025)
- [Architecture Evolution](/mattgpt-design-spec/wireframes/architecture_evolution_slide_wireframe.html) - Streamlit MVP to React Rebuild roadmap

### New Components & Features
- **How Agy Searches Modal** - 3-step RAG search flow visualization
- **Related Projects Grid** - 3-column context-aware project recommendations
- **Match Confidence Indicators** - Visual confidence bars (green/orange/red thresholds)
- **Timeline Era Groups** - 5 career phases with progressive disclosure
- **Advanced Filter Pattern** - Primary + collapsible advanced filters

---

## Key Concepts

### The Credibility Engine

MattGPT replaces generic claims with **instant, quantifiable proof**:

- ❌ NOT: "Matt is experienced in agile transformation"
- ✅ YES: "Matt accelerated delivery 4x at JPMorgan Chase by implementing CI/CD pipelines and automated testing frameworks." [Source: Agile Transformation at JPMorgan Chase]

### Two-Layer Governance

**Layer 1: Integrity (STAR Method)**
Every project includes Situation, Task, Action, Result with measurable metrics.

**Layer 2: Intelligence (Tagging)**
5P taxonomy + semantic tags enable semantic search and pattern recognition.

### User Personas

1. **Recruiter** - Speed, scalability, filtering (breadth and comparison)
2. **Hiring Manager** - Depth, narrative structure, metrics (verifiability)
3. **Content User (Matt)** - Quick retrieval, synthesis (interview prep)

---

## What This Demonstrates

### Product Leadership
- ✅ Strategic positioning (WHY/HOW/WHAT framework)
- ✅ User research (3 distinct personas with different needs)
- ✅ MVP scoping (conscious trade-offs, not technical debt)
- ✅ Roadmap planning (NOW/NEXT/LATER phases)

### Technical Execution
- ✅ Modern AI architecture (RAG pipeline, vector search)
- ✅ Data governance (mandatory STAR validation)
- ✅ Performance optimization (semantic search with confidence scoring)
- ✅ Scalable design (modular architecture → microservices → enterprise)

### Design Thinking
- ✅ Information architecture (6 user flows, 9 view specifications)
- ✅ Component systems (reusable patterns, design tokens)
- ✅ Interaction design (detailed wireframe annotations)
- ✅ Accessibility (keyboard nav, screen readers, contrast)

---

## Live Application

**Try the working MVP:** [askmattgpt.streamlit.app](https://askmattgpt.streamlit.app)

**GitHub Repository:** [github.com/mcpugmire1/mattgpt-design-spec](https://github.com/mcpugmire1/mattgpt-design-spec)

---

## Contact

**Matt Pugmire**
Product & Platform Leader | Digital Transformation Director

📧 [mpugmire@gmail.com](mailto:mpugmire@gmail.com)
💼 [LinkedIn](https://www.linkedin.com/in/mattpugmire/)
🤖 [Ask Agy](https://askmattgpt.streamlit.app) - AI assistant powered by 130+ project stories

---

*This design specification represents the complete product blueprint from discovery through detailed technical and UX specifications. It demonstrates end-to-end product development: from strategic vision through user research, technical architecture, and meticulous execution.*

**Last Updated:** December 2025
