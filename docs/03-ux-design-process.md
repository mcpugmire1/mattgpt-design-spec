# UX Design Process

**MattGPT: From Information Architecture to Detailed Wireframe Specifications**

> This document outlines the complete user experience design process, from site architecture and user flows through detailed component specifications for every view in the MattGPT application.

---

## Table of Contents

1. [Site Architecture & Information Hierarchy](#site-architecture--information-hierarchy)
2. [User Navigation Flows](#user-navigation-flows)
3. [Homepage Starter Cards](#homepage-starter-cards)
4. [Search Pipeline Architecture](#search-pipeline-architecture)
5. [View Specifications](#view-specifications)
   - [Homepage](#homepage-uiux-specification)
   - [Industry Landing Pages](#industry-landing-pages)
   - [Explore Stories (Table/Card/Timeline)](#explore-stories-views)
   - [Detail View](#detail-view-specification)
   - [Ask MattGPT](#ask-mattgpt-experience)
   - [About Matt](#about-matt-page)

---

## Site Architecture & Information Hierarchy

### Full Site Structure

```
Home Page
├── Navigation (Persistent)
│   ├── Explore Stories
│   ├── Ask MattGPT
│   └── About Matt
│
├── Industry/Domain Entry Points
│   ├── Product Innovation
│   ├── App Modernization
│   ├── Financial Services (55 projects) → Banking Landing Page
│   ├── Cross-Industry Transformation (51 projects) → Cross-Industry Landing
│   ├── Consulting & Transformation
│   └── Teams & Talent Development
│
└── Quick Question → Ask MattGPT

Banking Landing Page
├── Client Filter (JPMorgan Chase, RBC, Fiserv, etc.)
├── 16 Capability Categories
│   ├── Agile Transformation (8 projects)
│   ├── Modern Engineering (8 projects)
│   ├── Global Payments (7 projects)
│   └── [13 more categories]
└── Individual Project Stories

Cross-Industry Landing Page
├── Industries Served Filter
├── 15+ Transformation Capabilities
└── Individual Project Stories

Explore Stories
├── Filter Controls
│   ├── Primary: Industry, Domain Category, Client, Capability, Outcome Type
│   └── Advanced: Role, Date Range, Technologies
├── View Modes
│   ├── Table View (high-density browsing)
│   ├── Card View (visual story previews)
│   └── Timeline View (5 Era-based career progression)
└── Detail View (inline expansion with STAR format, Key Metrics, Related Projects)

Ask MattGPT
├── Landing Page
│   ├── Agy introduction and starter prompts
│   └── "How Agy Searches" modal (3-step RAG flow)
├── Conversation View
│   ├── Semantic search with confidence scoring
│   ├── Source citations with match confidence
│   ├── Related projects suggestions
│   └── Conversation history with context management
└── Query Validation
    ├── Semantic router (dual-threshold 0.80/0.72)
    └── Nonsense detection (10+ intent families)

About Matt
├── Hero Header
├── Career Evolution Timeline
├── Core Competencies Grid
├── Leadership Philosophy
├── How I Built MattGPT
└── Try Agy CTA
```

---

## User Navigation Flows

MattGPT supports six distinct user journey patterns, each optimized for different discovery behaviors.

### User Journey Diagrams

The following diagrams illustrate the high-level navigation paths for each user journey.

### Flow 1: Banking Browse-First User

**Journey:**
1. Lands on Home
2. Sees "Financial Services - 55 projects" card
3. Clicks "See Banking Projects →"
4. Arrives at Banking Landing Page
5. Sees 16 capability categories
6. Clicks "Agile Transformation (8)"
7. Sees 8 specific stories
8. Clicks story → reads full detail

**Persona:** Recruiter or hiring manager with specific industry focus
**Goal:** Validate industry-specific experience depth

![Flow 1: Banking Browse-First User](/mattgpt-design-spec/images/wireflows/flow1-banking-browse.svg)

---

### Flow 2: Explore Stories First

**Journey:**
1. Lands on Home
2. Clicks "Explore Stories" in navigation
3. Arrives at Explore Stories page (Table View default)
4. Uses filters (Industry, Domain, Client, Capability)
5. Switches between Table, Card, or Timeline views
6. Clicks story → Detail View expands inline
7. Can pivot to Banking Landing or Ask MattGPT

**Persona:** Recruiter doing comprehensive candidate review
**Goal:** Browse all stories with filtering and multiple view modes

![Flow 2: Search-First User](/mattgpt-design-spec/images/wireflows/flow2-search-first.svg)

---

### Flow 3: Capability-First User

**Journey:**
1. Lands on Home
2. Clicks "Product Innovation & Strategy"
3. Goes to Explore Stories (filtered by capability)
4. Sees stories across all industries
5. Can filter by industry if needed
6. Clicks story → reads full detail

**Persona:** Hiring manager seeking specific skill evidence
**Goal:** Find proof of capability regardless of industry

![Flow 3: Capability-First User](/mattgpt-design-spec/images/wireflows/flow3-capability-first.svg)

---

### Flow 4: Conversational User (Ask MattGPT)

**Journey:**
1. Lands on Home
2. Clicks "Quick Question" or "Ask MattGPT" in navigation
3. Arrives at Ask MattGPT Landing Page (Agy introduction, starter prompts)
4. Types question or clicks starter prompt
5. Semantic router validates query (accepts/rejects with suggestions)
6. Gets Agy response with confidence-scored sources
7. Clicks source chip → navigates to full story in Explore Stories
8. Can ask follow-up questions with conversation context

**Persona:** Time-constrained decision-maker or interview prep
**Goal:** Fast, natural language query with verifiable sources and context retention

![Flow 4: Conversational User](/mattgpt-design-spec/images/wireflows/flow4-conversational.svg)

---

### Flow 5: Cross-Industry User

**Journey:**
1. Lands on Home
2. Clicks "Cross-Industry"
3. Arrives at Cross-Industry Landing Page
4. Sees 51 cross-industry projects
5. Clicks "Digital Product Development (3)"
6. Sees 3 specific stories
7. Clicks story → reads full detail

**Persona:** Hiring manager seeking transferable patterns
**Goal:** Find methodology consistency across contexts

![Flow 5: Cross-Industry User](/mattgpt-design-spec/images/wireflows/flow5-cross-industry.svg)

---

### Flow 6: Client-Specific Filtering

**Journey:**
1. Lands on Home
2. Clicks "Explore Stories" or Banking Landing Page
3. Uses Client filter dropdown → selects "JPMorgan Chase"
4. Sees 33+ JPMorgan stories filtered
5. Can switch between Table, Card, Timeline views
6. Clicks story → Detail View with STAR format and metrics

**Persona:** Recruiter validating specific client experience
**Goal:** Confirm breadth and depth of work with target organization

![Flow 6: Direct Search](/mattgpt-design-spec/images/wireflows/flow6-direct-search.svg)

---

## Homepage Starter Cards

The homepage uses strategic entry point cards that route users to the most relevant experience based on their intent.

| Homepage Starter Card | Destination | What User Sees |
|----------------------|-------------|----------------|
| **Product Innovation & Strategy** | Explore Stories (capability filter) | Stories filtered by Product Innovation capability, across all industries |
| **App Modernization** | Explore Stories (capability filter) | Stories filtered by App Modernization capability, across all industries |
| **Financial Services & Payments** (55 projects) | Banking Landing Page | 16 capability categories, client tags, option to view all 55 stories |
| **Cross-Industry Transformation** (51 projects) | Cross-Industry Landing Page | 51 projects across 15+ transformation capabilities — delivery patterns that work across any industry |
| **Consulting & Transformation** (51 projects) | Explore Stories (cross-industry filter) | 51 cross-industry transformation stories, no industry landing needed |
| **Teams & Talent Development** (300+ professionals) | Explore Stories (outcome filter) | Stories focused on team development, upskilling, leadership |
| **Quick Question** | Ask MattGPT | Chat interface, conversational AI, semantic search |

---

## Search Pipeline Architecture

### How MattGPT Search Works

**System Metrics:**
- Stories Indexed: 130+
- Avg Response Time: 1.2s
- Retrieval Accuracy: 87%
- Vector Dimensions: 384

### Query Flow

```
User Question
    ↓
[Embedding + Intent Analysis]
    ↓
[Pinecone Vector Search]
    ↓
[Confidence Scoring & Ranking]
    ↓
[Top 3 Stories Retrieved]
    ↓
[Response Synthesis with Sources]
```

### Architecture Components

**Semantic Search Pipeline:**
- OpenAI embeddings (text-embedding-3-small)
- 1536-dimensional vector space
- Pinecone vector database with metadata filtering

**Semantic Retrieval:**
- Vector similarity matching via Pinecone
- Confidence scoring (high/low/none thresholds)
- Intent recognition for query understanding

**Data & Processing:**
- **Story Corpus:** 130+ structured narratives from Fortune 500 projects
- **Framework:** STAR/5P framework encoding
- **Metadata:** Rich tagging (client, domain, outcomes, metrics)

**Response Generation:**
- Context-aware retrieval (top-k=30)
- Multi-mode synthesis (Narrative/Key Points/Deep Dive)
- Source attribution with confidence scoring

### Search & Retrieval Details

**Semantic Search:**
- Pinecone cosine similarity (vector matching)
- Minimum similarity threshold: 0.15
- Top-k pool: 30 candidates before ranking
- Confidence-based result filtering

**Response Synthesis:**
- Rank top 3 stories by similarity score
- Generate 3 views from same sources:
  - **Narrative:** 1-paragraph summary
  - **Key Points:** 3-4 bullets
  - **Deep Dive:** Full STAR breakdown
- Interactive source chips with confidence %

**Key Differentiators:**
- Semantic retrieval with confidence thresholds ensures high-quality matches
- Multi-mode synthesis provides flexible presentation for different use cases
- Context locking allows follow-up questions on specific stories
- Off-domain gating with suggestion chips prevents poor matches

---

## View Specifications

### Homepage UI/UX Specification

#### Wireframe Components

| # | Element | Category | Key Details |
|---|---------|----------|-------------|
| 1 | Navigation Links | Interaction | Static top navigation; no dropdowns; entire label is clickable. Links persist on all pages. |
| 2 | Profile Headline/Copy | Content | Static introductory text. No animation or personalization. Center-aligned. |
| 3 | Stats Callouts | Technical/Data | Four equal-width data tiles (20+ Years, 130 Projects, 300+ Professionals, 15+ Clients). Non-interactive. |
| 4 | Category Icon | Visual/Technical | Decorative icon positioned top-left of each category card. No functional behavior. |
| 5 | Category Tags | Interaction/Logic | Non-clickable labels used to indicate subtopics. Displayed inline below description text. |
| 6 | Primary CTA Button | Interaction | Primary call-to-action on each category card. Full button is clickable; right-arrow included. |
| 7 | Product Question Link | Interaction | Text-only link beneath description. Clicking redirects to relevant case study view. |
| 8 | Quick Question Text | Interaction/Logic | Prompt text encouraging input. Paired with CTA button. |
| 9 | Background Gradient | Visual/Technical | Static background fill for hero and footer sections. No parallax or animation. |

#### Style Guidelines

**Typography:**
- H1 / Bold / Centered for main heading
- Supporting subtext: 2 lines max, regular weight

**Navigation Links:**
- Font weight: Medium
- Hover state: underline or color shift
- Active tab: visually distinguished

**Buttons / CTAs:**
- Primary buttons: gradient background + arrow icon
- Hover state: shadow lift or subtle scale
- Secondary links: text-only

**Cards / Sections:**
- 24px internal padding
- 48px spacing between sections
- Rounded corners and consistent shadow depth

**Gradients / Backgrounds:**
- Hero and footer: same purple/blue gradient
- No alternate or rotated versions
- Avoid harsh banding

**Spacing & Alignment:**
- 12-column grid layout or consistent center alignment
- Maintain equal height across paired cards

---

### Industry Landing Pages

Both Banking and Cross-Industry landing pages follow a consistent pattern optimized for capability-based exploration.

#### Banking Landing Page Structure

**Key Components:**

| # | Element | Category | Details |
|---|---------|----------|---------|
| 1 | Top Navigation Bar | Header | Dark navy background, 4 links: Homepage, Explore Stories, Ask MattGPT, About Matt |
| 2 | Page Title | Heading (H1) | "Financial Services / Banking" |
| 3 | Subtitle/Breadcrumb | Descriptive Text | "55 projects across 16 specialized areas — or ask Agy to find what you're looking for" |
| 4 | Client Filter Row | Filter Component | Pill-style buttons: JPMorgan Chase (33), RBC (11), Fiserv (7), American Express (3), Capital One (2), HSBC (2) |
| 5 | Section Header | Heading (H2) | "Explore by Capability" with supporting text |
| 6 | Capability Cards | Card Grid | 3-column responsive grid containing 16 capability category cards |
| 7 | Card Components | Composite Element | Each card: Icon, Title, Project count (purple text), Descriptive tags/keywords |
| 8 | Search CTA Section | Call-to-Action | "Can't find what you're looking for?" with "Ask Agy" button |
| 9 | Footer Contact Section | Footer | "Let's Connect" with contact methods |

**16 Capability Categories:**
- Agile Transformation (8)
- Modern Engineering (8)
- Global Payments (7)
- Technology Strategy (5)
- Program Management (4)
- Digital Product (3)
- Data & Analytics (3)
- Business Process (3)
- Cross-Functional (3)
- Cloud Transformation (2)
- Application Modernization (2)
- Enterprise Integration (2)
- Security & Compliance (2)
- DevOps (1)
- VPP Adoption (1)

#### Cross-Industry Landing Page Structure

**Unique Elements:**

- **Industries Served Filter:** Financial Services & Banking, Healthcare & Life Sciences, Manufacturing, Retail & Consumer Goods, Technology & SaaS, Telecommunications, Public Sector
- **15+ Transformation Categories:** Agile Transformation, Modern Engineering Practices, Digital Experience/eCommerce, Organization & Sustainable Innovation, Technology Strategy & Advisory, Program Management & Governance, DevOps & Continuous Delivery, Digital Product Development, Product Modernization, Workplace Practices, Cloud Transformation & Migration, Product Management & Innovation, Business Process Optimization, etc.

---

### Explore Stories Views

MattGPT provides three distinct view modes for browsing project stories, each optimized for different user preferences and use cases.

**Page Header & Subtitle:**
- **H1 Heading:** "Explore Stories"
- **Subtitle:** "Browse Matt's 130+ transformation projects by industry, client, or domain — or ask Agy 🐾 to help you find what you're looking for"

#### Shared Framework Elements

All three views share:
- **Filter Controls:** Industry, Domain Category, Client, Role dropdowns
- **Search Bar:** Full-width input with placeholder "Search by title, client, or keywords"
- **Results Summary:** "Browsing 1-10 of 130 projects"
- **View Switcher:** Toggle buttons for Table / Cards / Timeline
- **Ask Agy Link:** "Can't find what you're looking for? Ask Agy →"

**Filter Dropdown Options (Implementation Details):**

> **⚠️ Architecture Note:** Filter dropdown values are **dynamically generated** at runtime from JSONL source data and Pinecone index queries (as implemented in current `app.py`). The examples below represent current data state but will update as project data changes. Only filter labels ("Industry", "Domain Category", etc.) are hardcoded UI elements.

**Industry Filter:**
- All Industries (default)
- Financial Services / Banking *(dynamic)*
- Cross-Industry *(dynamic)*
- Healthcare *(dynamic)*
- Technology *(dynamic)*

**Domain Category Filter:**
- All Domains (default)
- Agile Transformation *(dynamic)*
- Modern Engineering *(dynamic)*
- Payments & Treasury *(dynamic)*
- Product Innovation *(dynamic)*

**Client Filter:**
- All Clients (default)
- JPMorgan Chase (33 projects) *(dynamic - count updates with data)*
- RBC (11 projects) *(dynamic - count updates with data)*
- Accenture (13 projects) *(dynamic - count updates with data)*
- Fiserv (7 projects) *(dynamic - count updates with data)*

**Role Filter:**
- All Roles (default)
- Director *(dynamic)*
- Senior Manager *(dynamic)*
- Manager *(dynamic)*

**View Switcher Options:**
- Table (data grid with sortable columns)
- Cards (visual card layout, 3-column grid)
- Timeline (Era-based view with 5 career phases)

---

#### Table View Specification

**Layout:** Four-column data table with sortable headers

**Columns:**
1. **PROJECT TITLE** - Blue/purple linked text, wraps to multiple lines
2. **CLIENT** - Blue pill-style tags (e.g., "JPMorgan Chase", "Walmart")
3. **ROLE** - Plain text (Director, Senior Manager, etc.)
4. **DOMAIN** - Multiple tags/categories separated by slashes

**Interaction:**
- Sortable column headers (click to sort)
- Hover state: light tint (#f5f7fa)
- Selected row: #e3f2fd background + 4px solid primary blue left border
- Clicking row expands Detail View inline below

**Style Guidelines:**
- Row Height: Minimum 48px with 12px vertical padding
- Typography: Project titles medium/bold, metadata muted gray
- Pagination: Active page filled button, 10 results per page
- Empty State: "No matching projects found. Adjust filters."

---

#### Card View Specification

**Layout:** 3-column responsive grid (→ 2-col tablet → 1-col mobile)

**Card Components:**
1. **Project Title** (clickable header)
2. **Client Badge** (pill, right-aligned)
3. **Summary Text** (first 2-3 lines of STAR story, truncated with ellipsis)
4. **Role Tag** (pill in footer)
5. **Domain Tag** (aligned opposite role)

**Interaction:**
- Full card selectable (cursor: pointer)
- Hover: Subtle card lift + shadow
- Clicking card loads detail pane below
- Selected card: light blue highlight or persistent border

**Style Guidelines:**
- Background: White with 1px light gray border (#e5e7eb)
- Border Radius: 8-12px
- Shadow (Hover): 0 2px 8px rgba(0,0,0,0.05)
- Spacing: 24px padding inside card, 32px vertical gap between rows
- Mobile: Collapse to single column, role/domain move below summary

---

#### Timeline View Specification

**Layout:** Vertical timeline with year markers and project entries

**Components:**
1. **Timeline Year Badges** - Left-aligned yearly group markers (sticky)
2. **Timeline Rail & Dots** - Vertical line with circular markers per project
3. **Timeline Entry Block** - Card-style block aligned to rail
4. **Project Title** (clickable)
5. **Client Badge** (pill, right-aligned)
6. **Metadata Line** (Role + Domain beneath title)

**Interaction:**
- Hover: Light tint or shadow
- Active Selection: Blue highlight + left border (same as table)
- Clicking entry opens detail pane below
- Selection persists across paging/filtering

**Style Guidelines:**
- Year Badge: Small rounded pill, left-aligned, fixed to group
- Rail & Dots: 2px vertical line with 12px dots per entry
- Entry Container: White background, 1px border, 8-12px radius
- Spacing: 24px vertical between entries, 48px before new year group
- Auto-Scroll: Scroll to detail on selection
- Mobile: Stack layout (dot above, metadata below)

---

### Detail View Specification

The Detail View expands inline below the selected story in any Explore Stories view mode.

#### Core Components

| # | Element | Category | Key Details |
|---|---------|----------|-------------|
| 1 | Header Title | Content | Project name, always visible, anchors auto-scroll |
| 2 | Metadata Row | Metadata | Client · Role · Dates · Domain (inline pills/badges) |
| 3 | Share Button | Interaction | Click → copy link to clipboard, shows success toast |
| 4 | Share Toast | Feedback | "✓ Link copied to clipboard!" - auto-dismiss 2-3s |
| 5 | Export Button | Interaction | Click → open print dialog (window.print()) |
| 6 | Export Toast | Feedback | "Print dialog opened — save as PDF" - auto-dismiss 2-3s |
| 7-10 | STAR Sections | Content | Situation, Task, Action, Result (with metrics, bold outcomes) |
| 11 | Technologies & Practices | Metadata | Tag pills, read-only (future: filter by tag) |
| 12 | Core Competencies | Metadata | Vertical list, read-only |
| 13 | Key Metrics | Data | 3-4 stat tiles max, consistent units (e.g., "5 mo", "2x", "100%", "80%+") |

#### STAR Method Display

**Situation:**
- Icon + section title
- Rich text paragraph
- Business context and challenge

**Task:**
- Icon + section title
- Specific objective or problem statement
- Scope definition

**Action:**
- Icon + section title
- Bulleted list supported
- Methodologies, decisions, execution details

**Result:**
- Icon + section title
- Measurable outcomes with metrics highlighted
- Bold key achievements

#### Sidebar: Key Metrics

**Example Metrics:**
- **5 mo** - Accelerated Go-to-Market
- **2x** - Teams at Internal Speed
- **100%** - Product Owner Acceptance
- **80%+** - Test Coverage

#### Style Guidelines

**Pane Layout:**
- Full-width below list
- Pushes pagination down
- Sticky inner header (title + actions)

**Section Blocks:**
- White background
- 1px border #E5E7EB
- 12-16px radius
- 16px padding
- 24px vertical gap

**Typography:**
- Title: Bold
- Section headings: Medium + icon
- Body: Regular
- Metadata: Muted gray

**Buttons (Share/Export):**
- Right-aligned
- Icon + label
- Tooltips: Share: "Copy link (MVP)", Export: "Open print dialog (MVP)"

**Toasts:**
- Bottom-center
- 2-3s duration
- Slide-up + fade-in
- Non-blocking
- Share: success/green
- Export: neutral/blue

**Auto-Scroll:**
- On open, scroll pane title to top of viewport
- Use: `scrollIntoView({behavior:'smooth'})`

**Responsive:**
- Mobile: sidebar stacks below sections
- Actions remain visible in sticky header

---

### Ask MattGPT Experience

The Ask MattGPT feature provides a conversational interface for exploring Matt's project portfolio through natural language queries.

#### Landing Page

**Hero Section:**
- **Title:** "Ask MattGPT"
- **Subtitle:** "Your AI-powered guide — Tracking down insights from 20+ years of transformation experience"
- **"How It Works" Button:** Top-right CTA, toggles expanded info panel
- **Status Strip:** "Semantic search active | Pinecone index: ready | 130 stories indexed"

**Intro Section:**
- **Headline:** "Hi, I'm Agy 🐾" (with dog avatar)
- **Description:** "I'm a Plott Hound — a breed known for tracking skills and determination. Perfect mascot for helping you hunt down insights from Matt's 130+ transformation projects."
- **Capability Statement:** "Ask me about specific methodologies, leadership approaches, or project outcomes. I understand context, not just keywords."

**Suggestion Cards (6 Examples):**
- 🚀 "How did Matt transform global payments at JPMorgan Chase?"
- 📊 "Track down Matt's innovation leadership stories"
- ⚡ "Find Matt's platform engineering projects"
- 🎯 "Show me Matt's GenAI work in healthcare"
- 👥 "How did Matt scale agile across 150+ people?"
- 🔍 "Show me how Matt handles stakeholders"

**Input Bar:**
- Sticky at bottom
- Placeholder: "Ask me anything — from building MattGPT to leading global programs..."
- Multi-line expand on typing
- Send button (gradient, disabled until input is non-empty)

**Footer:**
- "Powered by OpenAI GPT-4 with semantic search across 130 project case studies"

---

#### Expanded "How It Works" Panel

**Content:**
- **Title:** "How Agy Works"
- **Description:** "130+ Real Project Stories - Every answer is grounded in Matt's actual work across Fortune 500 companies — JPMorgan Chase, RBC, Capital One, and more."

**Three Core Capabilities:**

1. **Understands Intent, Not Just Keywords**
   - "Ask naturally — there's no need to memorize or change 'Show me platform engineering work'" — I understand business manager and conversational queries.

2. **See the Evidence**
   - Responses include links to full STAR stories with specific outcomes, metrics, and methodologies you can examine.

3. **"Try asking questions like:**
   - "How do you scale agile across large organizations?"
   - "Show me your platform engineering experience"
   - "What's your approach to stakeholder management?"
   - "How does Matt approach product-market fit validation?"

**Example Prompts:**
- Clicking any example prefills (not auto-sends) input field

---

#### Conversation View

**Layout:**
- Hero band remains at top with "How It Works" toggle
- Status strip persists
- Chat transcript: vertical stack of message pairs
- Auto-scrolls to most recent message

**Message Components:**

**User Message Bubble:**
- Left-aligned
- Light gray background (#F3F4F6)
- 12-16px padding
- 16px radius
- Avatar: "U" or user icon

**AI Response Bubble:**
- Elevated white container
- Soft shadow (elevation +1)
- Border radius: 16px
- Max-width: ~80% content width
- Avatar: Robot icon or "AI"
- Supports: Bold, italics, bullet lists, soft dividers

**AI Thinking Indicator:**
- Temporary placeholder
- "🐾 Tracking..." with animated dots
- Appears before answer renders

**Related Projects Tag Row:**
- Appears under AI response
- Pill-style chips with project titles
- Light background (#F1F5F9) + 1px border (#CBD5E1)
- Hover: darkens slightly
- Click: navigates to full project detail view (Explore Stories)

**Action Buttons (under AI response):**
- **Helpful:** Thumbs up/down toggle (turns green when active)
- **Copy:** Copies full answer to clipboard
- **Share:** Copies permalink to conversation

**Input Bar:**
- Sticky at bottom
- Same styling as landing
- Focus ring: #6366F1
- Enter = send
- Shift+Enter = newline
- Escape = clear focus

**Error Toasts:**
- Bottom-center
- Rounded
- Light shadow
- Auto-dismiss ~3s
- Examples: "Response copied!" / "Something went wrong — retry"

---

### About Matt Page

The About Matt page serves as both a professional introduction and a demonstration of the product's construction.

#### Header (Hero Band)

**Components:**
- **Avatar:** Circular placeholder or headshot (80-96px, left-aligned)
- **Name (H1):** "Matt Pugmire"
- **Role/Title:** "Digital Transformation Leader | Director of Technology Delivery"
- **Summary Paragraph:** 2-3 concise sentences positioning outcomes and scope
  - Example: "20+ years driving innovation, agile transformation, and application modernization across Fortune 500 companies. Proven track record of accelerating delivery 3-20x, scaling engineering teams to 150+ people, and building high-performing product organizations. Exploring opportunities to lead platform engineering, product innovation, and organizational transformation initiatives."

**Metric Badges Row:**
- **20+ Years** Experience
- **130** Projects Delivered
- **300+** Professionals Trained
- **15+** Enterprise Clients
- **3-20x** Delivery Acceleration

**Style:**
- Full-width purple→indigo gradient (120-160px height desktop, 96-120px mobile)
- Name: 28-34px, weight 700, white/near-white
- Role: 14-16px, weight 500, white at 80-90% opacity
- Metrics: Container with light surface, 1px border, 10-12px radius

---

#### Career Evolution Timeline

**Section Title:** "Career Evolution"
**Subtitle:** "From individual contributor to enterprise-scale transformation leader"

**Timeline Format:**
- Single vertical column
- Left-aligned year markers with right-aligned content blocks
- Vertical line connecting markers (1px solid #E5E7EB)

**Timeline Entries (Reverse Chronological):**

**2023-Present: Principal Consultant | Innovation & Upskilling**
- Accenture
- Focused on GenAI, cloud-native architecture, and building a LLM-powered portfolio assistant

**2016-2023: Director, Cloud Innovation Center**
- Accenture
- Led 25-person Innovation Centers (150+ engineers) | 85+ products | $300M+ revenue | 4x faster delivery

**2016-2023: Capability Development Lead, CloudFirst**
- Accenture
- Upskilled 240+ professionals | 48% proficiency increase | 92% faster delivery | Culture transformation

**2016-2018: Cloud Native Architecture Lead, Liquid Studio**
- Accenture
- Built cloud-native architecture | AWS deployment pipelines | Rapid proof-of-concept delivery

**2006-2016: Technology Architecture Manager, Payments**
- Accenture
- Key: Payments, E-payments, banking, and platform modernization

**2005-2008: Technology Manager**
- Accenture
- Led team of 12

**2000-2005: Startups & Consulting**
- Various
- Built technology stacks | ecommerce & CMS | Scaled migration solutions

---

#### Core Competencies

**Section Title:** "Core Competencies"

**Three Pillars (2-3 column grid):**

1. **Product & Innovation**
   - Cloud-Native Development
   - Platform Engineering
   - Digital Product Development
   - User-Centered Design
   - API-First Architecture

2. **Platform Engineering**
   - Cloud-Native Development
   - Microservices Architecture
   - CI/CD Automation
   - Infrastructure as Code
   - Site Reliability Engineering

3. **Agile at Scale**
   - Large-Scale Transformations
   - Team-Building Frameworks
   - Continuous Improvement
   - Metrics-Driven Delivery

---

#### Leadership Philosophy

**Section Title:** "Leadership Philosophy"

**Four Values (2x2 grid of gradient pill cards):**

1. **🎯 Outcomes Over Syntax**
   - Business value comes first. Technical excellence serves outcomes, not the other way around. I prioritize solutions that deliver measurable impact over architectural purity.

2. **📈 Experimentation Culture**
   - Innovation requires permission to fail. I create environments where teams can experiment safely, learn fast, and iterate toward breakthrough solutions.

3. **🤝 Servant Leadership**
   - Great leaders remove blockers, not make decisions. I focus on empowering teams, building trust, and creating conditions for autonomous, high-velocity delivery.

4. **📊 Continuous Learning**
   - Curiosity compounds. I invest in learning new technologies, testing assumptions, and sharing knowledge — because stagnation is the biggest risk.

---

#### How I Built MattGPT

**Section Title:** "How I Built MattGPT"

**The Problem:**
"Traditional portfolios are static PDFs that don't scale. Recruiters and hiring managers can't easily search 130+ projects by methodology, outcome, or domain. I wanted to create an intelligent, conversational interface that understands intent and surfaces verifiable proof."

**Tech Stack (Icon Pills):**
- Python 3.11
- Streamlit (MVP)
- OpenAI GPT-4
- Pinecone
- Sentence Transformers
- Pandas
- NumPy
- Netlify

**System Architecture Flow:**

```
Data Ingestion → Embedding → Vector Store → Semantic Search
     ↓              ↓            ↓              ↓
Source Data    AI Data Index  Pinecone    RAG Orchestrator
(JSONL)        (Vectors)                  (LLM)
```

**The Secret Sauce: Semantic Search with Confidence Scoring**

```python
def semantic_search(query: str, top_k: int = 30) -> dict:
    """
    Pure semantic retrieval with confidence-based filtering
    to ensure high-quality, relevant results.
    """
    # Vector similarity search via Pinecone
    hits = pinecone_query(embed(query), top_k=top_k)

    # Calculate confidence from top score
    top_score = max(h.get("pc_score", 0.0) for h in hits)
    confidence = "high" if top_score >= 0.25 else "low" if top_score >= 0.15 else "none"

    return {"results": hits, "confidence": confidence, "top_score": top_score}
```

**What MattGPT Can Do:**

**🧠 LLM-Powered**
- RAG (Retrieval-Augmented Generation) using structured STAR stories and semantic search
- Automated tag generation using ontology and keyword extraction
- Embedded classification to discover similar or complementary projects
- Ask for insights in plain English — no keyword guessing

**🔍 Semantic Search Strategy**
- Vector-based meaning matching - Understanding intent, not just keywords (eg. "bootstrap it" vs "start a new project")
- Confidence scoring - Three-tier system (high/low/none) ensures quality results
- Metadata filtering - Client, industry, domain, role filters for precise targeting
- Threshold-based gating - Prevents low-quality matches from appearing

**📊 Frontend (Streamlit)**
- Conversational UI interface with context-aware history persistence
- Multi-view presentation: Table, Timeline, and Card views for diverse user needs
- Responsive design and UI/UX optimized for both desktop and mobile
- Real-time filtering: Regular steering committee meetings with client decision makers
- Export & Share: PDF download or share via unique project-level URLs

**🔧 Backend & DevOps**
- Conversational workflow for candidate screening
- Context-aware UI using Streamlit's stateful UI for resume filtering
- Semantic embeddings using Sentence Transformers
- Metadata filtering: Sort, search, tag for instant access to what you need
- Modularizing: Logging for query analysis, response quality, and system health

---

#### Try Agy CTA

**Section:** "🎯 See It In Action"

**Content:**
"This isn't just a portfolio showcase — Agy 🐾 is a working AI assistant that can answer detailed questions about my 130+ projects, methodologies, and outcomes. Think of it as an interactive interview you can conduct on your own time."

**Try asking questions like:**
- "How did Matt scale engineering teams from 4 to 150+ people?"
- "What were the biggest challenges at the Accenture Innovation Center?"
- "Show me examples of agile transformation with measurable outcomes"
- "How does Matt approach product-market fit validation?"

**CTA Button:** "Ask Agy About My Experience →"

**Footer Note:** "Real AI assistant • 130+ projects • Instant answers • Available 24/7"

---

## Design System Guidelines

### Color Palette

**Primary:**
- Purple: #8B5CF6
- Indigo: #6366F1
- Blue: #3498DB

**Neutral:**
- Dark Navy: #2C3E50
- Medium Gray: #6B7280
- Light Gray: #E5E7EB
- Background: #F9FAFB
- White: #FFFFFF

**Semantic:**
- Success: Green
- Info: Blue
- Warning: Orange
- Error: Red

### Typography

**Font Stack:** System fonts (Arial, Helvetica, sans-serif for MVP)

**Scale:**
- H1: 28-34px, weight 700
- H2: 20-24px, weight 600
- H3: 16-18px, weight 600
- Body: 14-16px, weight 400
- Small/Meta: 12-14px, weight 400

**Line Height:** 1.5-1.7 for body text

### Spacing System

**Base Unit:** 4px

**Common Spacings:**
- 8px - Tight spacing (between related elements)
- 12px - Compact spacing (within cards)
- 16px - Standard spacing (most common)
- 24px - Section spacing (between groups)
- 32px - Large spacing (between major sections)
- 48px - Extra large (between page sections)

### Component Patterns

**Cards:**
- White background
- 1px border #E5E7EB
- Border radius: 8-12px
- Padding: 16-24px
- Shadow on hover: 0 2px 8px rgba(0,0,0,0.05)

**Buttons:**
- Primary: Gradient purple→blue, white text
- Secondary: Outlined, primary color border
- Height: 40-44px minimum
- Padding: 12-24px horizontal
- Border radius: 8-10px

**Pills/Tags:**
- Small rounded containers
- 1-2px border or light background
- Padding: 4-8px vertical, 8-12px horizontal
- Border radius: 12-16px

**Input Fields:**
- 1px border #E5E7EB
- Border radius: 8-10px
- Padding: 10-12px
- Focus ring: 2px #6366F1

---

## Responsive Breakpoints

- **Mobile:** < 768px (1 column, stacked layout)
- **Tablet:** 768px - 1024px (2 columns)
- **Desktop:** > 1024px (3+ columns)

**Mobile Adaptations:**
- Navigation: Hamburger menu
- Cards: Single column stack
- Tables: Horizontal scroll or card transformation
- Sidebars: Stack below main content
- Reduced padding/spacing

---

## Accessibility Guidelines

**Keyboard Navigation:**
- All interactive elements keyboard-focusable
- Visible focus indicators (2px ring #6366F1)
- Logical tab order
- Enter = activate, Escape = close/cancel

**Screen Readers:**
- Semantic HTML5 elements
- ARIA labels on interactive components
- Alt text on all images
- Proper heading hierarchy

**Color Contrast:**
- Minimum 4.5:1 for body text
- Minimum 3:1 for large text
- Never rely on color alone for meaning

**Touch Targets:**
- Minimum 44×44px for interactive elements
- Adequate spacing between touch targets

---

**Related Documentation:**
- [Product Vision](/mattgpt-design-spec/docs/01-product-vision) - Strategic positioning and user personas
- [Technical Architecture](/mattgpt-design-spec/docs/02-technical-architecture) - RAG pipeline and system design
- [Building MattGPT](/mattgpt-design-spec/docs/04-building-mattgpt) - Development journey and lessons learned

---

*Last Updated: December 2025*
*Version: 1.0 (Initial UX Design Documentation)*
