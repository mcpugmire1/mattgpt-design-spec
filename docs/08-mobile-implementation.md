# Mobile Implementation

**MattGPT: Mobile-Responsive Design**

> This document details the mobile-first responsive design implementation for the MattGPT portfolio application, including breakpoints, patterns, and component behaviors.

---

## Table of Contents

1. [Overview](#overview)
2. [Responsive Breakpoints](#responsive-breakpoints)
3. [Mobile-First Patterns](#mobile-first-patterns)
4. [Component Behaviors](#component-behaviors)
5. [Touch Optimization](#touch-optimization)
6. [Performance Considerations](#performance-considerations)
7. [Testing Strategy](#testing-strategy)

---

## Overview

MattGPT implements production-quality mobile CSS with 200+ lines of responsive styling (global_styles.py:399-596). The approach prioritizes mobile experience while maintaining desktop functionality.

**Key Stats:**
- **Mobile Breakpoint:** < 767px
- **Tablet Breakpoint:** 768px - 1024px
- **Desktop Breakpoint:** > 1024px
- **Mobile CSS:** 200 lines (global_styles.py:399-596)
- **Touch Target Minimum:** 44px

---

## Responsive Breakpoints

### Breakpoint Definitions

```css
/* Mobile First - Base styles apply to mobile */
/* Default: Mobile < 767px */

/* Tablet */
@media (min-width: 768px) and (max-width: 1024px) {
    /* Tablet-specific enhancements */
}

/* Desktop */
@media (min-width: 1024px) {
    /* Desktop-specific enhancements */
}

/* Mobile Overrides (when needed) */
@media (max-width: 767px) {
    /* Explicit mobile-only styles */
}
```

### Why These Breakpoints?

- **767px (Mobile):** Captures iPhone, Android phones in portrait
- **768px (Tablet Start):** iPad portrait and larger tablets
- **1024px (Desktop):** Standard laptop screens and above

### Device Coverage

| Device Category | Width Range | Breakpoint |
|----------------|-------------|------------|
| Mobile Portrait | 320-767px | < 767px |
| Mobile Landscape | 568-926px | < 767px or 768-1024px |
| Tablet Portrait | 768-834px | 768-1024px |
| Tablet Landscape | 1024-1194px | > 1024px |
| Laptop | 1280-1920px | > 1024px |
| Desktop | 1920px+ | > 1024px |

---

## Mobile-First Patterns

### 1. Stacking Layouts

**Desktop:** Multi-column grid
**Mobile:** Single column stack

```css
/* Base (Mobile) */
.filter-row {
    display: flex;
    flex-direction: column;
    gap: var(--spacing-sm);
}

/* Desktop */
@media (min-width: 1024px) {
    .filter-row {
        flex-direction: row;
        gap: var(--spacing-md);
    }
}
```

**Implementation Example:**
- Filters stack vertically on mobile (explore_stories.py)
- Search input full-width
- Industry/Capability in 2-column grid on mobile

### 2. Collapsible Navigation

**Desktop:** Full navigation visible
**Mobile:** Hamburger menu

```css
@media (max-width: 767px) {
    .nav-links {
        display: none; /* Hidden by default */
    }

    .nav-links.active {
        display: flex;
        flex-direction: column;
    }

    .hamburger {
        display: block; /* Show hamburger */
    }
}
```

**Implementation:**
- Hamburger menu < 768px
- Vertical stack of links
- Touch-friendly 44px targets

### 3. Horizontal Scroll Tables

**Desktop:** Full table layout
**Mobile:** Horizontal scroll with sticky first column

```css
@media (max-width: 767px) {
    .table-container {
        overflow-x: auto;
        -webkit-overflow-scrolling: touch;
    }

    table {
        min-width: 600px; /* Forces horizontal scroll */
    }

    th:first-child,
    td:first-child {
        position: sticky;
        left: 0;
        background: var(--bg-surface);
        z-index: 1;
    }
}
```

**Implementation:**
- Preserve table functionality on mobile
- Sticky first column for context
- Smooth touch scrolling

### 4. Adaptive Typography

**Desktop:** Larger text
**Mobile:** Optimized for readability

```css
/* Base (Mobile) */
h1 {
    font-size: 28px;
    line-height: 1.2;
}

/* Desktop */
@media (min-width: 1024px) {
    h1 {
        font-size: 48px;
        line-height: 1.1;
    }
}
```

### 5. Contextual Content

**Desktop:** Show all details
**Mobile:** Progressive disclosure

```css
@media (max-width: 767px) {
    .secondary-info {
        display: none; /* Hide on mobile */
    }

    .card.expanded .secondary-info {
        display: block; /* Show when expanded */
    }
}
```

---

## Component Behaviors

### Navigation Bar

**Desktop (> 1024px):**
- Full horizontal nav with all links visible
- Logo left, links center, CTA right
- Hover states

**Tablet (768-1024px):**
- Slightly condensed spacing
- All links still visible
- Touch-friendly targets

**Mobile (< 767px):**
- Logo + hamburger menu
- Vertical stacked links (hidden by default)
- Full-width dropdown on menu open

### Filters (Explore Stories)

**Desktop:**
```
┌────────────────────────────────────────────┐
│ [Search    ] [Industry▼] [Capability▼]    │
│ ▸ Advanced  [Client▼] [Role▼] [Domain▼]   │
└────────────────────────────────────────────┘
```

**Mobile:**
```
┌──────────────────┐
│ [Search        ] │
│ [Industry▼]      │
│ [Capability▼]    │
│ ▸ Advanced       │
│   [Client▼]      │
│   [Role▼]        │
│   [Domain▼]      │
└──────────────────┘
```

**Implementation:**
- Vertical stack on mobile
- Dropdowns full-width
- Advanced filters collapsed by default

### Timeline View

**Desktop:**
- Era badges left-aligned (190px width)
- Timeline center line
- Cards alternate left/right

**Mobile:**
- Era badges move above cards
- Single-column card layout
- Left-aligned timeline
- No alternating sides

**Implementation Location:** timeline_view.py

### Story Cards

**Desktop:**
- 3-column grid
- Fixed card heights
- Hover effects

**Tablet:**
- 2-column grid
- Responsive card heights

**Mobile:**
- Single column
- Full-width cards
- Tap instead of hover

### Related Projects Grid (Ask MattGPT)

**Desktop:**
- 3-column grid
- Buttons side-by-side

**Mobile:**
- Single column
- Stacked buttons
- Full-width story detail expansion

---

## Touch Optimization

### 1. Target Sizes

**Minimum Touch Target:** 44x44px (Apple HIG, Android Material Design)

```css
/* Button sizing */
button {
    min-height: 44px;
    min-width: 44px;
    padding: 12px 24px;
}

/* Link spacing */
a {
    padding: 12px;
    margin: 4px 0;
}
```

### 2. Touch-Friendly Spacing

```css
/* Add breathing room between interactive elements */
.button-group button {
    margin: 8px 4px; /* Prevents accidental taps */
}
```

### 3. Swipe Gestures

```css
/* Enable smooth momentum scrolling */
.scrollable {
    -webkit-overflow-scrolling: touch;
    overflow-y: auto;
}
```

### 4. Active States

Provide visual feedback on touch:

```css
button:active {
    transform: scale(0.98);
    opacity: 0.8;
}
```

### 5. Disable Hover-Only Features

```css
@media (hover: none) {
    /* Remove hover-dependent features on touch devices */
    .tooltip-on-hover {
        display: none;
    }
}
```

---

## Performance Considerations

### 1. Image Optimization

```html
<!-- Responsive images -->
<img
    src="image-mobile.jpg"
    srcset="image-mobile.jpg 480w,
            image-tablet.jpg 768w,
            image-desktop.jpg 1200w"
    sizes="(max-width: 767px) 100vw,
           (max-width: 1024px) 50vw,
           33vw"
/>
```

### 2. Lazy Loading

```python
# In Streamlit components
st.image(url, loading="lazy")
```

### 3. Minimize Reflows

```css
/* GPU-accelerated animations */
.animated {
    will-change: transform, opacity;
    transform: translateZ(0);
}
```

### 4. Critical CSS

Load essential styles first, defer non-critical:

```python
# In global_styles.py
def load_critical_styles():
    """Load above-the-fold styles immediately"""

def load_non_critical_styles():
    """Defer below-fold styles"""
```

### 5. Font Loading Strategy

```css
/* System font stack for instant render */
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
```

---

## Testing Strategy

### Device Testing Matrix

| Device | OS | Browser | Priority |
|--------|----|---------| ---------|
| iPhone 14 | iOS 17 | Safari | High |
| iPhone SE | iOS 16 | Safari | High |
| Galaxy S23 | Android 13 | Chrome | High |
| iPad Pro | iOS 17 | Safari | Medium |
| iPad Mini | iOS 16 | Safari | Medium |
| Pixel 7 | Android 13 | Chrome | Medium |

### Breakpoint Testing

Test at these specific widths:
- **320px:** Smallest mobile (iPhone SE portrait)
- **375px:** Standard mobile (iPhone 14 portrait)
- **768px:** Tablet portrait (iPad)
- **1024px:** Tablet landscape / small laptop
- **1440px:** Standard desktop
- **1920px:** Large desktop

### Testing Tools

**Browser DevTools:**
- Chrome DevTools Device Mode
- Firefox Responsive Design Mode
- Safari Responsive Design Mode

**Online Tools:**
- BrowserStack (real device testing)
- Responsively App (multi-device preview)

**Manual Testing:**
- Physical devices (iOS + Android)
- Network throttling (3G/4G simulation)

### Test Scenarios

1. **Navigation:**
   - [ ] Hamburger menu opens/closes
   - [ ] Links are tappable (44px min)
   - [ ] Active states visible

2. **Filters:**
   - [ ] Vertical stack on mobile
   - [ ] Dropdowns full-width
   - [ ] Advanced filters collapsible

3. **Tables:**
   - [ ] Horizontal scroll works
   - [ ] Sticky first column
   - [ ] Touch scrolling smooth

4. **Cards:**
   - [ ] Single column on mobile
   - [ ] Full-width layout
   - [ ] Tap targets adequate

5. **Forms:**
   - [ ] Inputs zoom correctly (16px min font)
   - [ ] Submit buttons accessible
   - [ ] Validation messages visible

---

## Known Issues & Workarounds

### Issue 1: Streamlit Mobile Input Zoom

**Problem:** iOS Safari zooms in on inputs with font-size < 16px

**Workaround:**
```css
input, select, textarea {
    font-size: 16px !important; /* Prevents zoom */
}
```

### Issue 2: Sticky Position on iOS

**Problem:** Sticky elements jitter on scroll

**Workaround:**
```css
.sticky-element {
    position: -webkit-sticky;
    position: sticky;
    -webkit-transform: translateZ(0);
}
```

### Issue 3: Viewport Height on Mobile

**Problem:** 100vh includes browser chrome

**Workaround:**
```css
.full-height {
    min-height: 100vh;
    min-height: -webkit-fill-available;
}
```

---

## Implementation Files

### Primary Files
- **global_styles.py:399-596** - 200 lines of mobile CSS
- **mobile_overrides.py** - Additional mobile-specific fixes
- **timeline_view.py** - Mobile timeline layout
- **explore_stories.py** - Mobile filter stacking

### Component-Specific Mobile CSS
- **navbar.py** - Hamburger menu
- **footer.py** - Stacked footer layout
- **story_detail.py** - Mobile card layout
- **conversation_helpers.py** - Mobile Related Projects grid

---

## Mobile UX Best Practices

### 1. Thumb-Friendly Zones

Place primary actions in the "thumb zone" (bottom 2/3 of screen on mobile).

### 2. Reduce Cognitive Load

- Show 1-2 filters at a time on mobile
- Use progressive disclosure
- Prioritize primary actions

### 3. Maintain Context

- Sticky headers on scroll
- Breadcrumbs for deep navigation
- Back buttons for detail views

### 4. Fast Feedback

- Instant visual response to taps
- Loading indicators for async actions
- Optimistic UI updates

### 5. Offline Considerations

- Graceful degradation for poor networks
- Cache critical assets
- Clear error messages

---

## Roadmap

### Current State (Streamlit)
✅ Mobile-responsive CSS (200 lines)
✅ Touch-optimized controls
✅ Horizontal scroll tables
✅ Stacking layouts
✅ Hamburger navigation

### Phase 2 (React Migration - Q1 2026)
- PWA capabilities (offline support)
- App-like gestures (swipe navigation)
- Native scrolling performance
- Touch gesture libraries
- Advanced animations

---

## Related Documentation

- [02-technical-architecture.md](/mattgpt-design-spec/docs/02-technical-architecture) - Overall architecture
- [07-css-architecture.md](/mattgpt-design-spec/docs/07-css-architecture) - CSS system
- [component_inventory.md](/mattgpt-design-spec/components/component_inventory) - Component specs
- [sitemap_navigation.md](/mattgpt-design-spec/components/sitemap_navigation) - Mobile navigation

---

*Last Updated: December 2025*
*Version: 1.0 (Initial Mobile Implementation Documentation)*
