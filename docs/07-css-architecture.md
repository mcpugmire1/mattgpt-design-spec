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
- Desktop-first responsive design: desktop styles are the base, mobile changes are `max-width` overrides
- Dark mode support via CSS variables
- Component-scoped styling patterns
- Streamlit override strategies

---

## CSS Variable System

### Implementation Location
`llm_portfolio_assistant/ui/styles/global_styles.py`

### Core Variables

The canonical variable definitions live in the `:root` block at the top of `global_styles.py`. Read that file for current names and values. A copied table here re-drifts on every refactor; the audit-2026-06-15 record shows exactly how that happens.

### Usage Examples

```css
/* Good - Uses variables */
.card {
    background: var(--bg-surface);
    border: 1px solid var(--border-color);
    box-shadow: var(--hover-shadow);
}

/* Bad - Hardcoded values */
.card {
    background: #ffffff;
    border: 1px solid #e0e0e0;
}
```

For spacing and border-radius, check existing patterns in `global_styles.py`: there are no dedicated `--spacing-*` or `--radius-*` variables; use consistent values from what is already in the file.

---

## Breakpoint Strategy

### Breakpoint Definitions

```css
/* Desktop-first: base styles are the desktop layout (no media query) */

/* Mobile overrides: where every new mobile change goes */
@media (max-width: 767px) {
    /* Mobile adjustments; must not change desktop */
}

/* Tablet, where a layout needs it */
@media (min-width: 768px) and (max-width: 1023px) {
    /* Tablet adjustments */
}
```

The app repo's rule is that mobile changes go in `@media (max-width: 767px)` blocks and must not break desktop, checked at 375px, 767px and 1024px+. `min-width` queries are rare. Older blocks also use `max-width: 768px`, so a viewport exactly 768px wide gets mobile styles from some rules and not others; a few narrower queries (for example 480px) handle small phones.

### Implementation Location
`llm_portfolio_assistant/ui/styles/global_styles.py` (mobile `@media` blocks, check the file for current line positions)

### Breakpoint Usage Patterns

**Mobile (< 767px):**
- Single-column layouts
- Stacked navigation
- Full-width elements
- Touch-optimized controls
- Horizontal scroll tables

**Tablet (768-1023px):**
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
    padding: 16px;
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
    border-radius: 8px !important;
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
    --bg-surface: ...;
    --text-primary: ...;
    --border-color: ...;
}

/* Dark mode: same names, dark values */
body.dark-theme {
    --bg-surface: ...;
    --text-primary: ...;
    --border-color: ...;
}
```

The values live in the `:root` and `body.dark-theme` blocks of `global_styles.py`; as with the core variables, they are not copied here.

### Implementation Status
CSS variable system in place (`global_styles.py`)
Dark mode toggle: Streamlit's native settings menu handles dark mode on both desktop and mobile. The CSS variable system handles the actual theming; Streamlit handles the user-facing toggle mechanism.

---

## File Structure

### Current Architecture

```
llm_portfolio_assistant/
└── ui/
    ├── styles/
    │   ├── global_styles.py       # CSS variables, base styles, mobile queries
    │   ├── mobile_overrides.py    # Additional mobile-specific overrides
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
            └── styles.py          # Ask Agy-specific styles
```

### Style Loading Pattern

```python
# In app.py or page files
from ui.styles.global_styles import apply_global_styles

apply_global_styles()  # Injects CSS into page
```

---

## Best Practices

### 1. Always Use CSS Variables

```css
/* Good */
color: var(--text-primary);
background: var(--bg-surface);

/* Bad */
color: #2c3e50;
background: #ffffff;
```

### 2. Desktop Base, Mobile Overrides

```css
/* Base: the desktop layout */
.card {
    padding: 24px;
}

/* Mobile: override only what changes */
@media (max-width: 767px) {
    .card {
        padding: 16px;
    }
}
```

New mobile rules use `max-width: 767px`, not `768px`, and are checked at desktop width too.

### 3. Reuse Existing Spacing Values

There are no spacing or radius variables. Before adding a value, find what similar components in `global_styles.py` already use and match it, rather than inventing a new size.

### 4. Component Isolation

Keep component-specific styles scoped to prevent conflicts:

```css
/* Good - Scoped to component */
.story-card {
    /* Story card specific styles */
}

.story-card .title {
    /* Nested selector */
}

/* Bad - Too generic */
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
    /* GPU-accelerated */
    transform: translateX(10px);
    opacity: 0.8;

    /* Triggers reflow */
    margin-left: 10px;
    width: 200px;
}
```

---

## Migration Roadmap

### Phase 1: Streamlit (Current)
CSS variables implemented
Mobile-responsive design (`ui/styles/mobile_overrides.py`)
Component-scoped styling
Dark mode toggle (Streamlit native settings menu)

### Future: React Migration (if forcing function emerges)
- Convert to CSS Modules or Styled Components
- Implement full dark mode with system preference detection
- Add CSS-in-JS for dynamic theming
- Optimize bundle size with tree-shaking

---

## Related Documentation

- [02-technical-architecture.md](02-technical-architecture) - Overall architecture
- [08-mobile-implementation.md](08-mobile-implementation) - Mobile-specific patterns

---

*Last updated: {{ site.data.page_dates['07-css-architecture'] }}*
