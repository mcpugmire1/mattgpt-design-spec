# CSS Architecture

**MattGPT: Streamlit CSS Design System**

> This document outlines the CSS architecture, design system, and responsive implementation for the MattGPT portfolio application.

---

## Table of Contents

1. [Overview](#overview)
2. [CSS Variable System](#css-variable-system)
3. [Breakpoint Strategy](#breakpoint-strategy)
4. [Component Styling Conventions](#component-styling-conventions)
5. [Dark Mode Implementation](#dark-mode-implementation)
6. [File Structure](#file-structure)
7. [Best Practices](#best-practices)

---

## Overview

The MattGPT CSS architecture uses a modern, variable-based approach to ensure consistency, maintainability, and theme support across the application.

**Key Principles:**
- CSS variables for all colors, spacing, and typography
- Mobile-first responsive design
- Dark mode support via CSS variables
- Component-scoped styling patterns
- Streamlit override strategies

---

## CSS Variable System

### Implementation Location
`llm_portfolio_assistant/ui/styles/global_styles.py:28-122`

### Core Variables

```css
:root {
  /* Brand Colors */
  --color-primary: #8B5CF6;           /* Purple accent */
  --color-primary-dark: #7C3AED;      /* Darker purple (hover) */
  --color-primary-light: rgba(139, 92, 246, 0.1); /* Light purple bg */

  /* Neutral Colors */
  --color-dark: #2c3e50;              /* Primary text */
  --color-gray: #7f8c8d;              /* Secondary text */
  --color-light-gray: #f8f9fa;        /* Light backgrounds */
  --color-border: #e0e0e0;            /* Borders */

  /* Semantic Colors */
  --bg-surface: #ffffff;              /* Card backgrounds */
  --bg-hover: #f5f5f5;                /* Hover states */
  --text-primary: #2c3e50;            /* Primary text */
  --text-muted: #7d8590;              /* Muted text */
  --accent-purple: #8B5CF6;           /* Interactive elements */

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

### Usage Examples

```css
/* Good - Uses variables */
.card {
    background: var(--bg-surface);
    border: 1px solid var(--color-border);
    border-radius: var(--radius-md);
    padding: var(--spacing-md);
    box-shadow: var(--shadow-md);
}

/* Bad - Hardcoded values */
.card {
    background: #ffffff;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
    padding: 24px;
}
```

---

## Breakpoint Strategy

### Breakpoint Definitions

```css
/* Mobile-first approach */
/* Base styles: Mobile < 767px (no media query needed) */

/* Tablet */
@media (min-width: 768px) and (max-width: 1024px) {
    /* Tablet-specific styles */
}

/* Desktop */
@media (min-width: 1024px) {
    /* Desktop enhancements */
}

/* Mobile-specific overrides */
@media (max-width: 767px) {
    /* Mobile-specific adjustments */
}
```

### Implementation Location
`llm_portfolio_assistant/ui/styles/global_styles.py:399-596` (200 lines of mobile CSS)

### Breakpoint Usage Patterns

**Mobile (< 767px):**
- Single-column layouts
- Stacked navigation
- Full-width elements
- Touch-optimized controls
- Horizontal scroll tables

**Tablet (768-1024px):**
- 2-column grids
- Collapsed sidebar navigation
- Optimized spacing

**Desktop (1024px+):**
- Multi-column layouts
- Full navigation
- Hover states
- Larger spacing

---

## Component Styling Conventions

### File Organization

```
llm_portfolio_assistant/ui/styles/
├── global_styles.py        # CSS variables, base styles, mobile queries
├── mobile_overrides.py     # Additional mobile-specific overrides
└── __init__.py
```

### Component-Scoped CSS

Each component that needs custom styling uses class-based scoping:

```python
# In component file
st.markdown("""
<style>
.my-component {
    background: var(--bg-surface);
    padding: var(--spacing-md);
}

.my-component:hover {
    background: var(--bg-hover);
}
</style>
""", unsafe_allow_html=True)
```

### Streamlit Override Patterns

Streamlit applies its own CSS through emotion-cache. To override:

```css
/* Target Streamlit-generated classes */
[data-testid="stButton"] button {
    background: var(--accent-purple) !important;
    color: white !important;
    border-radius: var(--radius-md) !important;
}

/* Use attribute selectors for stability */
[class*="st-key-my_button"] button {
    /* Component-specific overrides */
}
```

---

## Dark Mode Implementation

### Variable Approach

Dark mode is implemented by swapping CSS variable values:

```css
/* Light mode (default) */
:root {
    --bg-surface: #ffffff;
    --text-primary: #2c3e50;
    --border-color: #e0e0e0;
}

/* Dark mode */
[data-theme="dark"] {
    --bg-surface: #1a1a1a;
    --text-primary: #e0e0e0;
    --border-color: #3a3a3a;
}
```

### Implementation Status
✅ CSS variable system in place (global_styles.py:28-122)
⏳ Dark mode toggle UI - pending implementation

### Future Enhancement
```python
# Planned dark mode toggle
if st.toggle("Dark Mode", key="theme_toggle"):
    st.session_state["theme"] = "dark"
else:
    st.session_state["theme"] = "light"
```

---

## File Structure

### Current Architecture

```
llm_portfolio_assistant/
└── ui/
    ├── styles/
    │   ├── global_styles.py       # 612 lines
    │   │   ├── CSS Variables      # Lines 28-122
    │   │   ├── Base Styles        # Lines 123-398
    │   │   └── Mobile Queries     # Lines 399-596
    │   ├── mobile_overrides.py    # Mobile-specific fixes
    │   └── __init__.py
    │
    ├── components/
    │   ├── navbar.py              # Includes nav-specific CSS
    │   ├── footer.py              # Includes footer-specific CSS
    │   ├── story_detail.py        # Includes card-specific CSS
    │   └── ...
    │
    └── pages/
        └── ask_mattgpt/
            └── styles.py          # 39.0 KB (Ask MattGPT-specific styles)
```

### Style Loading Pattern

```python
# In app.py or page files
from ui.styles.global_styles import load_global_styles

load_global_styles()  # Injects CSS into page
```

---

## Best Practices

### 1. Always Use CSS Variables

```css
/* ✅ Good */
color: var(--text-primary);
background: var(--bg-surface);

/* ❌ Bad */
color: #2c3e50;
background: #ffffff;
```

### 2. Mobile-First Approach

```css
/* ✅ Good - Base styles for mobile, enhance for desktop */
.card {
    padding: var(--spacing-sm);
}

@media (min-width: 1024px) {
    .card {
        padding: var(--spacing-lg);
    }
}

/* ❌ Bad - Desktop-first requires more overrides */
.card {
    padding: var(--spacing-lg);
}

@media (max-width: 767px) {
    .card {
        padding: var(--spacing-sm);
    }
}
```

### 3. Consistent Spacing Scale

Use the spacing variables instead of arbitrary values:

```css
/* ✅ Good */
margin: var(--spacing-md);
padding: var(--spacing-sm) var(--spacing-md);

/* ❌ Bad */
margin: 24px;
padding: 16px 24px;
```

### 4. Component Isolation

Keep component-specific styles scoped to prevent conflicts:

```css
/* ✅ Good - Scoped to component */
.story-card {
    /* Story card specific styles */
}

.story-card .title {
    /* Nested selector */
}

/* ❌ Bad - Too generic */
.card {
    /* Could conflict with other cards */
}
```

### 5. Streamlit Override Strategy

```css
/* Use !important sparingly, only for Streamlit overrides */
[data-testid="stButton"] button {
    background: var(--accent-purple) !important;
}

/* Prefer specificity over !important when possible */
.stApp .my-component button {
    background: var(--accent-purple);
}
```

### 6. Performance Optimization

```css
/* Avoid expensive properties on animations */
.animated-element {
    /* ✅ GPU-accelerated */
    transform: translateX(10px);
    opacity: 0.8;

    /* ❌ Triggers reflow */
    margin-left: 10px;
    width: 200px;
}
```

---

## Migration Roadmap

### Phase 1: Streamlit (Current)
✅ CSS variables implemented
✅ Mobile-responsive design
✅ Component-scoped styling
⏳ Dark mode toggle UI

### Future: React Migration (if forcing function emerges)
- Convert to CSS Modules or Styled Components
- Implement full dark mode with system preference detection
- Add CSS-in-JS for dynamic theming
- Optimize bundle size with tree-shaking

---

## Related Documentation

- [02-technical-architecture.md](/mattgpt-design-spec/docs/02-technical-architecture) - Overall architecture
- [08-mobile-implementation.md](/mattgpt-design-spec/docs/08-mobile-implementation) - Mobile-specific patterns
- [component_inventory.md](/mattgpt-design-spec/components/component_inventory) - Component specifications

---

*Last Updated: April 29, 2026 (React migration reframe)*
*Version: 1.1*
