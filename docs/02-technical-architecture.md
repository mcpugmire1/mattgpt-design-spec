# Technical Architecture

**MattGPT: From MVP to Production-Scale Architecture**

> This document outlines the technical architecture evolution of MattGPT, demonstrating intentional trade-off awareness and strategic technical planning from rapid prototyping to enterprise-grade scalability.

---

## Architecture Roadmap Overview

MattGPT's architecture follows a three-phase evolution strategy:

1. **Phase 1 (MVP):** Rapid validation with minimal investment
2. **Phase 2 (Production):** Scalable architecture for 10K+ concurrent users
3. **Phase 3 (Enterprise):** Enterprise resilience and observability at scale

---

## Phase 1: MVP - Rapid Validation

**Status:** ✅ Complete (December 2025)
**Timeline:** Current State
**Primary Goal:** Validate product-market fit with minimal investment

### Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| 🐍 **Frontend/Backend** | Streamlit (Monolith) | Rapid prototyping framework |
| 🤖 **LLM** | OpenAI GPT-4 | Natural language processing |
| 🌲 **Vector Database** | Pinecone | Semantic search and embeddings |
| 📦 **Dependencies** | Python venv + pip | Local development environment |

### Accepted Trade-offs

The MVP phase consciously accepted limitations to accelerate learning:

- **Mobile-responsive implementation:** Production-quality mobile CSS with breakpoints at 767px (mobile), 768-1024px (tablet), 1024+ (desktop)
- **Limited scalability:** Estimated ~100 concurrent users (not load-tested; Streamlit Community Cloud limits apply)
- **No caching layer:** Direct database queries
- **Modular architecture:** Refactored from monolithic app.py to component-based structure
- **Minimal observability:** Basic logging only

### Key Decisions

**Why Streamlit?**
- ✅ Initial MVP build time: 2 weeks vs 3+ months for React*
- ✅ Python-native (matches ML/AI ecosystem)
- ✅ Built-in state management
- ✅ Rapid iteration without frontend complexity

*Note: Continuous refinement and feature additions ongoing since launch.

**What We Validated:**
- RAG architecture effectiveness
- System prompt design
- STAR methodology implementation
- User experience patterns
- Search relevance tuning

### Architecture Evolutions Achieved (December 2025)

**Modular Component Structure:**
- Refactored monolithic ask_mattgpt.py (4,696 lines) into 9-file directory
- Separated concerns: backend_service.py, conversation_view.py, conversation_helpers.py
- Reusable component library (ui/components/)

**Design System:**
- CSS variables for light/dark mode support (global_styles.py:28-122)
- Standardized breakpoints: 767px (mobile), 768-1024px (tablet), 1024+ (desktop)
- Consistent purple brand (#8B5CF6) across all views

**Timeline View Innovation:**
- Era-based project grouping (timeline_view.py)
- 5 career eras with date range calculation
- Progressive disclosure pattern (6 most recent per era)

**Mobile Implementation:**
- 200 lines of responsive CSS (global_styles.py:399-596)
- Touch-optimized controls and stacking layouts
- Horizontal scroll tables with preserved functionality

---

## Phase 2: Production Scale

**Status:** 🎯 Planned (Q2 2025)
**Timeline:** Target State
**Primary Goal:** Scalable architecture for 10K+ concurrent users

### Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| ⚛️ **Frontend** | React + Next.js | Modern, performant UI framework |
| 🚀 **API Layer** | FastAPI Gateway | High-performance async API |
| 🦜 **RAG Framework** | LangChain/LlamaIndex | Orchestration and tooling |
| ☁️ **Infrastructure** | AWS/Azure | Cloud deployment and scaling |

### Architecture Improvements

**Micro-Frontend Approach:**
- Component-based architecture
- Independent deployment pipelines
- Shared design system
- Progressive enhancement

**API Gateway Pattern:**
- Service isolation
- Rate limiting and throttling
- Authentication/authorization layer
- API versioning support

**Scalability Features:**
- Horizontal auto-scaling
- Load balancing
- CDN for static assets
- Database connection pooling

**Mobile-First Design:**
- Responsive layouts
- Touch-optimized interactions
- Progressive web app (PWA) support
- Offline capabilities

---

## Phase 3: Enterprise-Grade

**Status:** 🔮 Future Vision (2026+)
**Timeline:** Aspirational State
**Primary Goal:** Enterprise resilience and observability at scale

### Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| 🌐 **CDN** | CloudFlare | Global content delivery |
| 💾 **Caching** | Redis | Response caching layer |
| 📊 **Monitoring** | DataDog/New Relic | APM and observability |
| 🔄 **Orchestration** | Airflow | ETL and workflow management |

### Enterprise Capabilities

**High Availability:**
- Multi-region deployment
- 99.9% uptime SLA
- Automated failover
- Disaster recovery planning

**Observability:**
- Real-time monitoring and alerting
- Distributed tracing
- Log aggregation and analysis
- Performance metrics dashboard

**Optimization:**
- A/B testing infrastructure
- Feature flagging system
- Performance profiling
- Cost optimization monitoring

**Data Pipeline:**
- Automated ETL workflows
- Data quality validation
- Incremental updates
- Version control for embeddings

---

## Strategic Rationale

### 1. De-Risk with MVP-First Thinking

**Decision:** Start with Streamlit monolith instead of production-ready stack

**Rationale:**
- Validated RAG architecture effectiveness before infrastructure investment
- Tested system prompt design with real users
- Refined user experience patterns
- Minimized wasted effort on premature optimization

**Result:** 2-week build time vs 3+ months for React-based solution

---

### 2. Preserve Core IP Across Phases

**Decision:** Keep data pipeline architecture unchanged across all phases

**What Stays Constant:**
- Embedding generation process
- Pinecone indexing strategy
- STAR taxonomy and metadata
- Search relevance algorithms

**What Evolves:**
- Presentation layer (Streamlit → React)
- API architecture (monolith → microservices)
- Infrastructure (local → cloud)

**Benefit:** Minimizes technical debt and preserves investment in core IP

---

### 3. Demonstrate Trade-Off Awareness

**MVP Limitations Were Intentional Shortcuts:**

| Limitation | Business Justification |
|-----------|----------------------|
| Desktop-only | Target users (recruiters, hiring managers) primarily use desktop |
| 100-user limit | Portfolio showcase doesn't require scale |
| Monolithic architecture | Faster development, simpler deployment |
| No caching | Query latency acceptable for MVP validation |

**vs. Accidental Technical Debt:**
- Not "ran out of time to implement mobile"
- Not "didn't know how to scale"
- **Consciously traded features for speed**

---

### 4. Align Architecture to Business Goals

**Phase 1: Portfolio Showcase**
- **Target:** 100 concurrent users
- **Purpose:** Demonstrate technical capability to employers
- **Stack:** Streamlit monolith (appropriate for scale)

**Phase 2: Public Product**
- **Target:** 10K concurrent users
- **Purpose:** Viable product for public use
- **Stack:** React + FastAPI (horizontally scalable)

**Phase 3: Revenue-Generating SaaS**
- **Target:** Enterprise customers
- **Purpose:** White-label credibility platform
- **Stack:** Enterprise observability and multi-tenancy

**Principle:** Stack complexity matches business maturity

---

## Current State Summary

### What's Live Today (Phase 1 - December 2025)

✅ Production Streamlit application at [askmattgpt.streamlit.app](https://askmattgpt.streamlit.app)
✅ RAG pipeline with GPT-4 and Pinecone vector search
✅ 130+ STAR-structured project stories
✅ Semantic + keyword hybrid search with confidence scoring
✅ **Timeline View** with Era-based career progression
✅ **Mobile-responsive design** (breakpoints: 767px, 1024px)
✅ **Dark mode support** via CSS variables
✅ **Modular architecture** (9-file ask_mattgpt/ structure)
✅ Conversation history and context management
✅ Related Projects UX pattern

### What's Next (Phase 2 - Q2 2025)

🎯 React + Next.js frontend refactor
🎯 FastAPI backend with microservices architecture
🎯 Enhanced mobile PWA capabilities
🎯 Cloud deployment (AWS/Azure)
🎯 CI/CD pipeline automation

### Future Vision (Phase 3 - 2026+)

🔮 Multi-region global deployment
🔮 Enterprise observability platform
🔮 Real-time performance monitoring
🔮 A/B testing infrastructure
🔮 White-label SaaS offering

---

## Technical Diagrams

For visual representations of the architecture, see:

- [RAG Architecture Diagrams](/images/architecture/) - System design and data flow
- [Interactive Wireframes](/wireframes/) - UI/UX design decisions
- [Architecture Evolution Slide](/wireframes/architecture_evolution_slide_wireframe.html) - Visual timeline

---

## Key Takeaways

1. **MVP-first approach** validated product concept before infrastructure investment
2. **Core IP preservation** ensures data pipeline remains stable across phases
3. **Conscious trade-offs** demonstrate strategic thinking vs accidental debt
4. **Business-aligned evolution** scales complexity with business needs
5. **Production-ready roadmap** shows clear path to enterprise scale

---

**Related Documentation:**
- [Product Vision](/mattgpt-design-spec/docs/01-product-vision) - Strategic positioning
- [UX Design Process](/mattgpt-design-spec/docs/03-ux-design-process) - Design decisions
- [Building MattGPT](/mattgpt-design-spec/docs/04-building-mattgpt) - Development journey

---

*Last Updated: December 2025 (Post-Audit Refresh)*
*Version: 1.1 (Updated with Implementation Details)*
