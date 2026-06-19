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
`llm_portfolio_assistant/ui/styles/global_styles.py`

### Core Variables

The canonical variable definitions live in the `:root` block at the top of `global_styles.py`. Read that file for current names and values. A copied table here re-drifts on every refactor — the audit-2026-06-15 record shows exactly how that happens.

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
`llm_portfolio_assistant/ui/styles/global_styles.py` (mobile `@media` blocks — check the file for current line positions)

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
✅ CSS variable system in place (`global_styles.py`)
✅ Dark mode toggle — implemented via Streamlit's native settings menu on desktop + custom mobile hamburger menu. Pragmatic reuse of platform capability rather than a custom-built toggle. The CSS variable system handles the actual theming; Streamlit handles the user-facing toggle mechanism.

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
✅ Mobile-responsive design (`ui/styles/mobile_overrides.py`)
✅ Component-scoped styling
✅ Dark mode toggle (Streamlit native settings menu)

### Future: React Migration (if forcing function emerges)
- Convert to CSS Modules or Styled Components
- Implement full dark mode with system preference detection
- Add CSS-in-JS for dynamic theming
- Optimize bundle size with tree-shaking

---

## Related Documentation

- [02-technical-architecture.md](/docs/02-technical-architecture) - Overall architecture
- [08-mobile-implementation.md](/docs/08-mobile-implementation) - Mobile-specific patterns
- [component_inventory.md](/mattgpt-design-spec/components/component_inventory) - Component specifications

---

*Last Updated: April 29, 2026 (React migration reframe)*
*Version: 1.1*
