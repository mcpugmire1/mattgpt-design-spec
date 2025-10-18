# Portfolio Site Map & Navigation Structure

## Site Hierarchy

```
Homepage (index.html)
│
├── About Matt (about_matt_wireframe.html)
│   └── → Ask Agy CTA
│
├── Explore Stories Hub
│   ├── Cards View (explore_stories_cards_wireframe.html) [DEFAULT]
│   ├── Table View (explore_stories_table_wireframe.html)
│   └── Timeline View (explore_stories_timeline_wireframe.html)
│   └── Each view → Project Detail Pane (in Table view)
│
└── Ask MattGPT
    ├── Landing Page (ask_mattgpt_landing_wireframe.html)
    └── Chat Interface (ask_mattgpt_wireframe.html)
```

## Navigation Components

### Global Navigation (All Pages)
```
Homepage | Explore Stories | Ask MattGPT | About Matt
```

**Links:**
- Homepage → `index.html`
- Explore Stories → `explore_stories_cards_wireframe.html` (default to Cards view)
- Ask MattGPT → `ask_mattgpt_landing_wireframe.html`
- About Matt → `about_matt_wireframe.html`

### Footer Navigation (All Pages)
```
📧 mcpugmire@gmail.com | 💼 LinkedIn | 🐾 Ask Agy
```

**Links:**
- Email → `mailto:mcpugmire@gmail.com`
- LinkedIn → `https://www.linkedin.com/in/matt-pugmire/`
- Ask Agy → `ask_mattgpt_landing_wireframe.html`

## Page-to-Page Flow

### Homepage Entry Points
1. **"About Matt"** button → `about_matt_wireframe.html`
2. **"Explore Stories"** button → `explore_stories_cards_wireframe.html`
3. **"Ask Agy 🐾"** button → `ask_mattgpt_landing_wireframe.html`

### About Matt Page Entry Points
1. **"Ask Agy About My Experience"** CTA → `ask_mattgpt_landing_wireframe.html`
2. **Global nav links** → Other pages

### Explore Stories Cross-View Switching
All three views include view switcher in top-right:
- **Cards** | Table | Timeline → `explore_stories_cards_wireframe.html`
- Cards | **Table** | Timeline → `explore_stories_table_wireframe.html`
- Cards | Table | **Timeline** → `explore_stories_timeline_wireframe.html`

### Ask Agy Entry Points Throughout Site
1. **Homepage hero** → "Ask Agy 🐾" button
2. **Global nav** → "Ask Agy 🐾" link (all pages)
3. **About Matt** → "Ask Agy About My Experience" CTA button
4. **Explore Stories search helpers** → "Ask Agy 🐾 →" inline links
5. **Explore Stories empty states** → "Ask Agy" CTA button
6. **Table detail pane** → "Ask Agy 🐾 About This" button (contextual)
7. **All page footers** → "🐾 Ask Agy" link

## User Journey Maps

### Journey 1: Recruiter First Visit (Most Common)
```
1. Land on Homepage (index.html)
   ↓ See hero stats + value prop

2. Click "About Matt" to understand background
   ↓ Read career timeline + technical depth

3. Scroll to "Try Agy" CTA
   ↓ Intrigued by AI assistant concept

4. Click "Ask Agy About My Experience"
   ↓ Land on Ask MattGPT Landing Page

5. Ask specific question about scaling teams
   ↓ Get detailed AI response with project references

6. Return to explore specific projects
   ↓ Navigate to Explore Stories

7. Filter by industry/role relevance
   ↓ Find 3-5 highly relevant projects

8. Return to contact via footer CTA
   ↓ Send LinkedIn message or email
```

**Key Conversion Points:**
- Homepage → About Matt (credibility building)
- About Matt → Ask Agy (engagement deepening)
- Ask Agy → Explore Stories (validation & detail)
- Footer → Contact (conversion)

### Journey 2: Technical Deep Dive
```
1. Land on Homepage
   ↓ Notice "115+ transformation projects"

2. Click "Explore Stories" immediately
   ↓ Land on Cards view

3. Filter by Domain: "Platform Engineering"
   ↓ See 20+ relevant projects

4. Switch to Table view for detailed comparison
   ↓ Sort by Outcome Impact

5. Click project to open detail pane
   ↓ Read STAR breakdown + metrics

6. Click "Ask Agy About This Project"
   ↓ Ask technical implementation questions

7. Navigate back to filter different domain
   ↓ Compare approaches across projects

8. Check About page for technical depth
   ↓ Validate architecture understanding

9. Contact via email
```

**Key Conversion Points:**
- Homepage → Explore Stories (immediate validation)
- Explore Stories detail → Ask Agy (engagement)
- Ask Agy responses → Credibility → Contact

### Journey 3: Quick Reference / Return Visitor
```
1. Direct link from LinkedIn/email
   ↓ Land on specific page

2. Use global nav to jump between sections
   ↓ Quick reference check

3. Use Ask Agy for specific question recall
   ↓ "What was that Azure migration outcome?"

4. Get answer + project reference
   ↓ Verify details

5. Return to conversation/interview prep
```

**Key Conversion Points:**
- Fast navigation (global nav on all pages)
- Ask Agy as knowledge retrieval (searchable portfolio)

## Cross-Linking Strategy

### Every Page Links to Ask Agy (7+ Entry Points)
**Rationale:** Ask Agy is the unique differentiator and engagement driver

### Explore Stories Views Cross-Link (View Switcher)
**Rationale:** Users have different browsing preferences (visual vs. data vs. chronological)

### Footer on Every Page
**Rationale:** Reduce friction for contact at any point in journey

### Contextual CTAs Based on Page Content
**Rationale:**
- Homepage → General exploration CTAs
- About Matt → "Ask Agy About My Experience" (contextual)
- Explore Stories → "Ask Agy About This Project" (specific)
- Search helpers → "Ask Agy" (alternative to filtering)

## Navigation UX Principles

### 1. Consistent Global Nav
- Same nav bar on every page
- Active state indication
- Mobile responsive collapse

### 2. Multiple Paths to Same Destination
- Ask Agy accessible from 7+ entry points
- Explore Stories accessible from homepage + nav
- Contact accessible from footer + hero

### 3. Contextual Assistance
- Search helpers when filtering might fail
- Empty state CTAs when no results
- Project-specific Ask Agy CTAs in detail views

### 4. Clear Visual Hierarchy
- Primary CTA: Purple buttons (#8B5CF6)
- Secondary actions: Links with underlines
- Tertiary: Footer links

### 5. Minimized Dead Ends
- Every page has footer with contact options
- Ask Agy available as fallback throughout
- Clear navigation back to hub pages

## Analytics Tracking Recommendations

### Key Events to Track
1. **Page views:** All pages
2. **Navigation clicks:** Global nav usage patterns
3. **CTA clicks:**
   - Homepage → About Matt
   - Homepage → Explore Stories
   - Homepage → Ask Agy
   - About Matt → Ask Agy CTA
   - Explore Stories → Ask Agy (search helpers)
   - Explore Stories → Ask Agy (detail pane)
   - Footer → Contact buttons
4. **Explore Stories interactions:**
   - View switching (Cards ↔ Table ↔ Timeline)
   - Filter usage
   - Project detail opens
5. **Ask Agy usage:**
   - Landing page → Chat interface
   - Question submissions
   - Follow-up questions
6. **Contact conversions:**
   - Email clicks
   - LinkedIn clicks
   - Time from entry to contact

### Funnel Metrics
```
Homepage Views
  ↓
About/Explore/Ask Agy Views (engagement)
  ↓
Multiple Page Views (interest depth)
  ↓
Ask Agy Usage (active evaluation)
  ↓
Contact Click (conversion)
```

## Mobile Navigation Considerations

### Hamburger Menu (< 768px)
```
☰ Menu
├── About Matt
├── Explore Stories
│   ├── Cards View
│   ├── Table View
│   └── Timeline View
└── Ask Agy 🐾
```

### Sticky Navigation
- Nav bar sticks to top on scroll
- Footer CTAs always accessible by scroll

### Touch-Friendly Targets
- Minimum 44px touch targets
- Adequate spacing between links
- Clear active states

## Implementation Checklist

- [ ] Verify all nav links point to correct files
- [ ] Test all cross-page navigation flows
- [ ] Confirm consistent nav styling across pages
- [ ] Validate footer appears on all pages
- [ ] Test view switcher functionality in Explore Stories
- [ ] Verify Ask Agy is accessible from all entry points
- [ ] Test mobile navigation collapse
- [ ] Confirm all external links (LinkedIn, email) work
- [ ] Validate active state indication
- [ ] Test keyboard navigation (tab order)
- [ ] Check link hover states
- [ ] Verify no broken links (404s)
