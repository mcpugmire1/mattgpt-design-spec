# MattGPT Site Architecture (December 2025)

## Page Hierarchy

```
Homepage (/)
├── Banking Landing Page (/banking)
│   └── View All 55 Stories → Explore Stories (filtered: Banking)
│
├── Cross-Industry Landing Page (/cross-industry)
│   └── View All 51 Stories → Explore Stories (filtered: Cross-Industry)
│
├── Explore Stories (/explore)
│   ├── Table View (default)
│   ├── Card View
│   ├── Timeline View (5 Eras)
│   └── Detail View (inline expansion)
│
├── Ask MattGPT (/ask)
│   ├── Landing Page
│   └── Conversation View
│
└── About Matt (/about)
```

## Navigation Structure

**Global Navigation (present on all pages):**
- Homepage
- Explore Stories
- Ask MattGPT
- About Matt

## Homepage Starter Cards → Destinations

| Starter Card | Destination | Filter Applied |
|-------------|-------------|----------------|
| **Product Innovation & Strategy** | Explore Stories | Capability: Product Innovation |
| **App Modernization** | Explore Stories | Capability: App Modernization |
| **Financial Services & Payments** (55) | Banking Landing Page | Industry: Banking |
| **Cross-Industry Transformation** (51) | Cross-Industry Landing Page | Industry: Cross-Industry |
| **Consulting & Transformation** (51) | Explore Stories | Industry: Cross-Industry |
| **Teams & Talent Development** (300+) | Explore Stories | Outcome: Team Development |
| **Quick Question** | Ask MattGPT | n/a |

## Key Features by Page

### Homepage
- Hero section with tagline
- 7 starter cards (capability/industry/action-based)
- Mobile-responsive grid layout

### Banking Landing Page
- 16 capability categories
- Client tags (JPMorgan, RBC, Capital One, etc.)
- "View All 55 Stories" CTA → Explore Stories

### Cross-Industry Landing Page
- 51 projects across 15+ transformation capabilities
- Industry filter chips
- "View All Stories" CTA → Explore Stories

### Explore Stories
- **3 View Modes:**
  - Table View (high-density browsing)
  - Card View (visual story previews)
  - Timeline View (5 Era-based career progression)
- **Filters:**
  - Primary: Industry, Domain Category, Client, Capability, Outcome Type
  - Advanced: Role, Date Range, Technologies
- **Detail View:** Inline expansion below selected story
  - Full STAR format
  - Key Metrics tiles
  - Related Projects
  - Export to PDF

### Ask MattGPT
- **Landing Page:**
  - Hero intro to Agy
  - Starter prompts (6 examples)
  - How Agy Searches modal
- **Conversation View:**
  - Chat interface with Agy
  - Semantic search with confidence scoring
  - Source citations with clickable links
  - Related projects suggestions
  - Conversation history

### About Matt
- Professional summary
- Career journey (timeline)
- Core competencies
- Leadership philosophy
- Contact information

## Technical Architecture

### Frontend
- **Framework:** Streamlit (Python)
- **Pages:** `ui/pages/` directory
  - `home.py`
  - `banking_landing.py`
  - `cross_industry_landing.py`
  - `explore_stories.py`
  - `ask_mattgpt/` (modular 9-file structure)
  - `about_matt.py`

### Backend Services
- `services/rag_service.py` - Semantic search
- `services/semantic_router.py` - Query validation
- `services/embedding_service.py` - OpenAI embeddings
- `utils/scoring.py` - Confidence scoring

### Data Layer
- **Vector DB:** Pinecone (semantic search)
- **LLM:** OpenAI GPT-4o-mini
- **Embeddings:** OpenAI text-embedding-3-small (1536 dims)
- **Stories:** 130+ STAR-formatted projects (JSONL)

### Query Validation
- **Semantic Router:** Dual-threshold (0.80/0.72)
- **Pattern Filters:** 10+ regex categories (nonsense_filters.jsonl)
- **Intent Families:** 10 categories (background, behavioral, technical, etc.)

## Mobile Responsive
- **Breakpoints:** 767px (mobile), 768-1024px (tablet), 1024+ (desktop)
- **Mobile Features:**
  - Stacking layouts
  - Touch-optimized controls
  - Horizontal scroll tables
  - Collapsible filters
  - Hamburger menu

## Design System
- **Brand Color:** Purple (#8B5CF6)
- **Dark Mode:** CSS variables (global_styles.py)
- **Components:** Reusable component library (ui/components/)
- **Typography:** Consistent heading hierarchy
- **Spacing:** Standardized margins/padding

---

**Status:** Phase 1 MVP (Complete - December 2025)
**Next:** Phase 2 React Rebuild (Q1 2025)
