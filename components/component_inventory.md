# Component Inventory & Mapping

## Global Components (Used Across All Pages)

### 1. Global Navigation Bar
**Used on:** All 7 pages

**Specifications:**
- Background: `#2c3e50` (dark slate)
- Height: 70px
- Fixed positioning
- Links: Logo/Name | About Matt | Explore Stories | Ask Agy 🐾
- Active state: Bold text, slight color change
- Hover state: Opacity 0.8

**Implementation Notes:**
- Should be identical across all pages
- Mobile: Collapses to hamburger menu
- Logo/Name always links to homepage

**Files:**
```
✓ index.html
✓ about_matt_wireframe.html
✓ explore_stories_cards_wireframe.html
✓ explore_stories_table_wireframe.html
✓ explore_stories_timeline_wireframe.html
✓ ask_mattgpt_landing_wireframe.html
✓ ask_mattgpt_wireframe.html
```

---

### 2. Professional Footer
**Used on:** All 7 pages

**Specifications:**
- Background: `#2c3e50` (dark slate)
- Padding: 48px 40px
- Text color: White
- Sections:
  - Headline: "Let's Connect" (28px)
  - Subhead: Opportunity focus areas (16px, 0.9 opacity)
  - Availability: Immediate start, location, consulting (14px, 0.75 opacity)
  - CTA buttons: Email, LinkedIn, Ask Agy

**Button Styles:**
- Primary (Email): `background: #8B5CF6`
- Secondary (LinkedIn, Ask Agy): `background: rgba(255,255,255,0.1)`
- Padding: 12px 28px
- Border-radius: 8px
- Font-weight: 600

**Implementation Notes:**
- Footer content should be identical across all pages
- Responsive: Buttons stack vertically on mobile
- All links functional (mailto:, https://, internal)

**Files:**
```
✓ index.html
✓ about_matt_wireframe.html
✓ explore_stories_cards_wireframe.html
✓ explore_stories_table_wireframe.html
✓ explore_stories_timeline_wireframe.html
✓ ask_mattgpt_landing_wireframe.html
✓ ask_mattgpt_wireframe.html
```

---

### 3. SEO Meta Tags
**Used on:** All 7 pages

**Standard Tags:**
```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>[Page-specific title]</title>
<meta name="description" content="[Page-specific description]">
```

**Page-Specific Variations:**
| Page | Title Focus | Description Focus |
|------|-------------|-------------------|
| Homepage | Director of Technology Delivery | 20+ years, 115+ projects, availability |
| About Matt | Director-level experience | Team scaling, Fortune 500, transformation |
| Explore Stories (all) | Interactive Portfolio | 115+ projects, STAR methodology, AI search |
| Ask MattGPT | AI-Powered Portfolio Assistant | Chat with Agy, 115+ projects, GPT-4 powered |

**Implementation Notes:**
- All titles include "Matt Pugmire" and role level (Director/VP)
- All descriptions mention key numbers (20+ years, 115+ projects)
- All descriptions include "available" or "exploring opportunities"
- Optimized for recruiter searches

---

## Page-Specific Components

### 4. Hero Section
**Used on:** Homepage (index.html)

**Specifications:**
- Large headline with name + role
- Subheadline with value proposition
- 3-4 key stats in grid (projects, years, companies, scale)
- 3 primary CTAs: About Matt | Explore Stories | Ask Agy 🐾
- Background: Gradient or solid with accent
- Min-height: 600px

**Visual Hierarchy:**
1. Name (48-56px, bold)
2. Role/Tagline (24px)
3. Stats (large numbers with labels)
4. CTAs (prominent buttons)

**Implementation Notes:**
- Stats should be visually prominent
- CTAs should use purple accent for primary
- Responsive: Stack vertically on mobile

**Files:**
```
✓ index.html (primary hero)
✓ ask_mattgpt_landing_wireframe.html (alternate hero)
```

---

### 5. "Try Agy" CTA Section
**Used on:** About Matt page

**Specifications:**
- Background: Gradient overlay `rgba(139, 92, 246, 0.05)` to `rgba(165, 180, 252, 0.05)`
- White card with left border: 4px solid `#8B5CF6`
- Border-radius: 12px
- Box-shadow: `0 4px 12px rgba(0, 0, 0, 0.08)`
- Padding: 40px
- Max-width: 900px, centered

**Content Structure:**
1. Headline with emoji: "🎯 See It In Action"
2. Value prop paragraph with bold emphasis
3. "Try asking questions like:" label
4. 4 example questions (bulleted list)
5. Primary CTA button: "Ask Agy About My Experience"
6. Footer note: feature highlights

**Button Styling:**
```css
display: inline-flex;
align-items: center;
gap: 10px;
padding: 16px 36px;
background: #8B5CF6;
color: white;
border-radius: 10px;
font-weight: 600;
font-size: 16px;
box-shadow: 0 4px 12px rgba(139, 92, 246, 0.3);
```

**Implementation Notes:**
- Positioned after technical architecture section
- Should feel like an invitation, not a hard sell
- Example questions tailored to director-level evaluation

**Files:**
```
✓ about_matt_wireframe.html
```

---

### 6. Filter Bar
**Used on:** Explore Stories (all 3 views)

**Specifications:**
- Search input (top)
- Filter row: 4 columns (Industry | Domain Category | Client | Role)
- Grid: `grid-template-columns: repeat(4, 1fr)`
- Gap: 15px
- Margin: 15px top

**Filter Dropdowns:**
```
Industry: All Industries, Financial Services, Technology, Healthcare, Retail, Government
Domain Category: All Domains, Platform Engineering, Product Leadership, Transformation, Delivery Acceleration
Client: All Clients, JPMorgan Chase, Accenture, Capital One, [others]
Role: All Roles, Director, Senior Manager, Manager
```

**Search Helper Text:**
```
"Can't find what you're looking for? Ask Agy 🐾 →"
```
- Position: Right-aligned below search
- Font-size: 12px
- Color: `#7f8c8d`

**Implementation Notes:**
- Search input should be prominent
- Filters should persist when switching views
- Helper text uses `.search-helper` class with `.ask-agy-link` for link

**Files:**
```
✓ explore_stories_cards_wireframe.html
✓ explore_stories_table_wireframe.html
✓ explore_stories_timeline_wireframe.html
```

---

### 7. View Switcher
**Used on:** Explore Stories (all 3 views)

**Specifications:**
- Position: Top-right of content area
- Horizontal layout with 3 options: Cards | Table | Timeline
- Active state: Bold, purple underline
- Inactive state: Gray text, clickable
- Font-size: 14px
- Gap: 16px between options

**States:**
```css
/* Active */
font-weight: 600;
color: #2c3e50;
border-bottom: 2px solid #8B5CF6;

/* Inactive */
font-weight: 400;
color: #7f8c8d;
cursor: pointer;
```

**Implementation Notes:**
- Should reflect current view
- Clicking switches to alternate view (different HTML file)
- All 3 views should have identical view switcher

**Files:**
```
✓ explore_stories_cards_wireframe.html
✓ explore_stories_table_wireframe.html
✓ explore_stories_timeline_wireframe.html
```

---

### 8. Project Card (Cards View)
**Used on:** Explore Stories Cards View

**Specifications:**
- White background with border: `1px solid #e0e0e0`
- Border-radius: 8px
- Padding: 20px
- Box-shadow on hover: `0 4px 12px rgba(0, 0, 0, 0.1)`
- Transition: `all 0.2s ease`

**Content Structure:**
1. **Header Row:**
   - Company name (bold, 16px)
   - Role badge (right-aligned, pill)

2. **Project Title:**
   - Bold, 18px, color `#2c3e50`
   - 2-line limit with ellipsis

3. **Description:**
   - 14px, color `#555`
   - 3-line limit with ellipsis
   - Line-height: 1.6

4. **Tags Row:**
   - Industry tag
   - Domain category tag
   - 2-3 skill tags
   - Style: Small pills, light background

5. **Metrics Row:**
   - 2-3 key outcomes
   - Icon + metric format
   - Examples: "📈 3x faster delivery", "👥 150+ team members"

6. **Footer:**
   - "View Details" link (purple, right-aligned)

**Implementation Notes:**
- Cards should have consistent height when possible
- Hover state should be subtle but noticeable
- Truncation should use CSS `text-overflow: ellipsis`

**Files:**
```
✓ explore_stories_cards_wireframe.html
```

---

### 9. Project Table (Table View)
**Used on:** Explore Stories Table View

**Specifications:**
- Full-width table
- Header: Sticky on scroll, background `#f8f9fa`
- Alternating row colors: White and `#fafafa`
- Border: `1px solid #e0e0e0`

**Columns:**
1. Company (20%)
2. Project Title (30%)
3. Domain Category (15%)
4. Role (10%)
5. Key Outcome (20%)
6. Details (5% - link icon)

**Row Interaction:**
- Hover state: Light purple background `rgba(139, 92, 246, 0.05)`
- Click: Opens detail pane to right (slide-in animation)
- Active row: Slightly darker purple background

**Implementation Notes:**
- Table should be sortable by clicking column headers
- Detail pane should overlay on smaller screens
- Minimum column widths to prevent cramping

**Files:**
```
✓ explore_stories_table_wireframe.html
```

---

### 10. Project Detail Pane (Table View)
**Used on:** Explore Stories Table View

**Specifications:**
- Width: 40% of screen (or full on mobile)
- Position: Fixed right side
- Background: White
- Box-shadow: `-4px 0 12px rgba(0, 0, 0, 0.1)`
- Padding: 30px
- Scrollable

**Content Structure:**
1. **Header:**
   - Company name + role badge
   - Project title (large, bold)
   - Close button (X, top-right)

2. **Quick Info:**
   - Timeline
   - Team size
   - Budget/scope (if applicable)

3. **STAR Breakdown:**
   - Situation (expandable)
   - Task (expandable)
   - Action (expandable with bullet points)
   - Result (metrics, bold emphasis)

4. **Skills & Technologies:**
   - Tag pills with icons

5. **Ask Agy About This Project:**
   - Purple CTA button
   - Background: `#f8f9fa`
   - Border: `2px solid #e0e0e0`
   - Centered

**Implementation Notes:**
- Detail pane should slide in from right
- Close button should be prominent
- STAR sections should be collapsible for long content
- "Ask Agy" CTA specific to this project

**Files:**
```
✓ explore_stories_table_wireframe.html
```

---

### 11. Timeline View (Timeline View)
**Used on:** Explore Stories Timeline View

**Specifications:**
- Vertical timeline with center line
- Projects alternate left/right
- Timeline nodes: Purple circles on center line
- Connection lines: Dashed `#e0e0e0`

**Timeline Node:**
- Circle: 16px diameter, `background: #8B5CF6`
- White border: 3px
- Box-shadow: `0 0 0 4px rgba(139, 92, 246, 0.1)`

**Project Cards:**
- Similar to Cards view styling
- Positioned left or right of timeline
- Width: 45% of container
- Margin: 20px vertical gap

**Date Labels:**
- Positioned at timeline nodes
- Font-size: 12px, bold
- Color: `#7f8c8d`

**Implementation Notes:**
- Timeline should be chronologically sorted (most recent at top)
- Cards alternate left/right for visual balance
- Responsive: Stack vertically on mobile with left-aligned timeline

**Files:**
```
✓ explore_stories_timeline_wireframe.html
```

---

### 12. Empty State
**Used on:** Explore Stories (all views when filtered results are empty)

**Specifications:**
```css
.empty-state {
    padding: 80px 40px;
    text-align: center;
    background: #fafafa;
    border-radius: 12px;
    margin: 30px;
}
```

**Content Structure:**
1. Icon: 😕 or 🔍 (large, 48px)
2. Headline: "No Projects Found" (24px, bold)
3. Description: "Try adjusting your filters or..." (16px, `#7f8c8d`)
4. CTA Button: "Ask Agy" (purple, prominent)

**Button:**
```css
.empty-state-btn {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    padding: 12px 28px;
    background: #8B5CF6;
    color: white;
    border-radius: 8px;
    font-weight: 600;
    font-size: 15px;
}
```

**Implementation Notes:**
- Only shown when filter combination returns 0 results
- Should be visually distinct but not alarming
- CTA should be clear alternative action

**Files:**
```
✓ explore_stories_cards_wireframe.html (commented example)
✓ explore_stories_table_wireframe.html (implementation)
✓ explore_stories_timeline_wireframe.html (implementation)
```

---

### 13. Badge Components
**Used on:** Multiple pages for roles, tags, categories

**Types:**

**Role Badge:**
```css
display: inline-block;
padding: 4px 12px;
border-radius: 12px;
font-size: 12px;
font-weight: 600;
background: #e8f5e9;  /* Green tint */
color: #2e7d32;
```

**Domain Badge:**
```css
background: #e3f2fd;  /* Blue tint */
color: #1565c0;
```

**Industry Badge:**
```css
background: #f3e5f5;  /* Purple tint */
color: #6a1b9a;
```

**Skill Tag:**
```css
background: #f5f5f5;  /* Gray */
color: #555;
border: 1px solid #e0e0e0;
```

**Implementation Notes:**
- Badges should be consistent across all views
- Color coding helps with quick scanning
- Hover state: Slightly darker background

**Files:**
```
✓ explore_stories_cards_wireframe.html
✓ explore_stories_table_wireframe.html
✓ explore_stories_timeline_wireframe.html
✓ about_matt_wireframe.html
```

---

### 14. Primary CTA Button
**Used on:** Homepage, About Matt, Explore Stories, Ask MattGPT

**Specifications:**
```css
.primary-cta {
    display: inline-flex;
    align-items: center;
    gap: 8-10px;
    padding: 12-16px 28-36px;
    background: #8B5CF6;
    color: white;
    border-radius: 8-10px;
    font-weight: 600;
    font-size: 15-16px;
    text-decoration: none;
    transition: all 0.2s ease;
    box-shadow: 0 4px 12px rgba(139, 92, 246, 0.3);
}

.primary-cta:hover {
    background: #7C3AED;
    box-shadow: 0 6px 16px rgba(139, 92, 246, 0.4);
    transform: translateY(-1px);
}
```

**Variants:**
- Large (Homepage hero): 16px font, 16px padding
- Medium (Most CTAs): 15px font, 12px padding
- Small (Cards, inline): 14px font, 10px padding

**Implementation Notes:**
- Always purple (`#8B5CF6`)
- Always includes hover animation
- Often includes emoji (🐾 for Agy-related)
- Clear contrast with background

**Files:**
```
✓ index.html (hero CTAs)
✓ about_matt_wireframe.html (Try Agy CTA)
✓ explore_stories_cards_wireframe.html (empty state)
✓ explore_stories_table_wireframe.html (Ask Agy About This)
✓ ask_mattgpt_landing_wireframe.html (Start Chat)
```

---

### 15. Ask Agy Inline Link
**Used on:** Explore Stories (all views) - search helpers

**Specifications:**
```css
.ask-agy-link {
    color: #8B5CF6;
    font-weight: 600;
    text-decoration: underline;
    text-decoration-color: rgba(139, 92, 246, 0.3);
    text-underline-offset: 2px;
    transition: all 0.2s ease;
}

.ask-agy-link:hover {
    text-decoration-color: #8B5CF6;
    color: #7C3AED;
}
```

**Context Usage:**
- Search helper text: "Can't find what you're looking for? Ask Agy 🐾 →"
- Empty state descriptions
- Project detail panes

**Implementation Notes:**
- Distinct from regular links (more prominent)
- Always includes 🐾 emoji
- Subtle underline becomes solid on hover

**Files:**
```
✓ explore_stories_cards_wireframe.html
✓ explore_stories_table_wireframe.html
✓ explore_stories_timeline_wireframe.html
```

---

## Component Reuse Matrix

| Component | Homepage | About Matt | Cards | Table | Timeline | Ask Landing | Ask Chat |
|-----------|----------|------------|-------|-------|----------|-------------|----------|
| Global Nav | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Footer | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| SEO Meta | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Hero | ✓ | - | - | - | - | ✓ | - |
| Try Agy CTA | - | ✓ | - | - | - | - | - |
| Filter Bar | - | - | ✓ | ✓ | ✓ | - | - |
| View Switcher | - | - | ✓ | ✓ | ✓ | - | - |
| Project Cards | - | - | ✓ | - | ✓ | - | - |
| Project Table | - | - | - | ✓ | - | - | - |
| Detail Pane | - | - | - | ✓ | - | - | - |
| Timeline | - | - | - | - | ✓ | - | - |
| Empty State | - | - | ✓ | ✓ | ✓ | - | - |
| Role Badge | - | ✓ | ✓ | ✓ | ✓ | - | - |
| Domain Badge | - | ✓ | ✓ | ✓ | ✓ | - | - |
| Primary CTA | ✓ | ✓ | ✓ | ✓ | - | ✓ | - |
| Ask Agy Link | - | - | ✓ | ✓ | ✓ | - | - |

## Component Library Structure

### Recommended File Organization (for implementation)

```
/css
  ├── global.css          → Nav, Footer, SEO, Typography
  ├── components.css      → Reusable components
  │   ├── buttons.css     → Primary CTA, Secondary buttons
  │   ├── badges.css      → Role, Domain, Industry, Skill tags
  │   ├── cards.css       → Project cards, CTA cards
  │   ├── forms.css       → Search inputs, Filter dropdowns
  │   └── empty-state.css → Empty state styling
  ├── homepage.css        → Homepage-specific styles
  ├── about.css           → About Matt-specific styles
  ├── explore.css         → Explore Stories shared styles
  │   ├── cards-view.css
  │   ├── table-view.css
  │   └── timeline-view.css
  └── ask-agy.css         → Ask MattGPT styles
```

### CSS Variable System (Recommended)

```css
:root {
  /* Brand Colors */
  --color-primary: #8B5CF6;
  --color-primary-dark: #7C3AED;
  --color-primary-light: rgba(139, 92, 246, 0.1);

  /* Neutral Colors */
  --color-dark: #2c3e50;
  --color-gray: #7f8c8d;
  --color-light-gray: #f8f9fa;
  --color-border: #e0e0e0;

  /* Typography */
  --font-size-xl: 48px;
  --font-size-lg: 28px;
  --font-size-md: 18px;
  --font-size-sm: 14px;
  --font-size-xs: 12px;

  /* Spacing */
  --spacing-xs: 8px;
  --spacing-sm: 16px;
  --spacing-md: 24px;
  --spacing-lg: 40px;
  --spacing-xl: 60px;

  /* Shadows */
  --shadow-sm: 0 2px 4px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 6px 16px rgba(0, 0, 0, 0.15);
  --shadow-primary: 0 4px 12px rgba(139, 92, 246, 0.3);

  /* Border Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-pill: 20px;

  /* Transitions */
  --transition-fast: all 0.15s ease;
  --transition-base: all 0.2s ease;
  --transition-slow: all 0.3s ease;
}
```

## Implementation Checklist

### Phase 1: Global Components
- [ ] Create global.css with nav, footer, typography
- [ ] Verify nav consistency across all 7 pages
- [ ] Verify footer consistency across all 7 pages
- [ ] Test mobile navigation collapse
- [ ] Validate all footer links (email, LinkedIn, Ask Agy)

### Phase 2: Reusable Components
- [ ] Create buttons.css (Primary CTA variants)
- [ ] Create badges.css (Role, Domain, Industry, Skill)
- [ ] Create forms.css (Search input, Filter dropdowns)
- [ ] Create empty-state.css
- [ ] Test Ask Agy link styling consistency

### Phase 3: Page-Specific Components
- [ ] Homepage hero section
- [ ] About Matt "Try Agy" CTA section
- [ ] Project cards (Cards view)
- [ ] Project table + detail pane (Table view)
- [ ] Timeline layout (Timeline view)
- [ ] Filter bar consistency across 3 views

### Phase 4: Responsive Design
- [ ] Mobile nav (hamburger menu)
- [ ] Mobile hero (stacked layout)
- [ ] Mobile cards (single column)
- [ ] Mobile table (horizontal scroll or cards fallback)
- [ ] Mobile timeline (single column, left-aligned)
- [ ] Mobile footer (stacked buttons)

### Phase 5: Interactive States
- [ ] Hover states on all buttons
- [ ] Hover states on cards/rows
- [ ] Active states on nav links
- [ ] Active states on view switcher
- [ ] Focus states for keyboard navigation
- [ ] Transition animations

### Phase 6: Cross-Browser Testing
- [ ] Chrome/Edge (Chromium)
- [ ] Firefox
- [ ] Safari (Desktop + iOS)
- [ ] Mobile browsers (iOS Safari, Chrome Mobile)

## Notes for Implementation Team

1. **Start with Global Components First**
   - Nav and footer create consistency
   - CSS variables enable easy theming

2. **Component-Based Approach**
   - Build each component in isolation
   - Test reusability before scaling

3. **Mobile-First CSS**
   - Start with mobile styles
   - Add desktop enhancements with media queries

4. **Accessibility Considerations**
   - Semantic HTML (nav, footer, main, section)
   - ARIA labels for interactive elements
   - Keyboard navigation support
   - Sufficient color contrast (WCAG AA)

5. **Performance Optimization**
   - Minimize CSS file size
   - Use CSS Grid/Flexbox (avoid float)
   - Optimize images and icons
   - Lazy load below-fold content

6. **Ask Agy Integration Points**
   - Ensure all 7+ entry points are functional
   - Consistent styling across all instances
   - Track clicks for analytics
