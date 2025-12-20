# Ask MattGPT Landing Page - Visual Spec Checklist

**CRITICAL: DO NOT rewrite working code. Only update CSS and HTML styling to match these specs.**

## Header Section
- [ ] Background: `linear-gradient(135deg, #667eea 0%, #764ba2 100%)`
- [ ] Avatar: 64px circle with 3px white border and shadow
- [ ] Title: "Ask MattGPT" in 32px white font
- [ ] Subtitle: "Meet Agy 🐾 — Tracking down insights from 20+ years of transformation experience"
- [ ] "How Agy searches" button in top-right with glass morphism effect:
  ```css
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.3);
  ```

## Status Bar
- [ ] Three status items centered horizontally
- [ ] "Semantic search: active" with green pulse dot animation
- [ ] "Pinecone index: ready"
- [ ] "130 stories: indexed"
- [ ] Pulse animation on the active indicator:
  ```css
  @keyframes pulse {
      0%, 100% { opacity: 1; }
      50% { opacity: 0.5; }
  }
  ```

## Welcome Section (Main Card)
- [ ] White card with `border-radius: 24px` and subtle shadow
- [ ] Agy avatar: 96px circle, centered
- [ ] Avatar hover: scale(1.05) with smooth transition
- [ ] Title: "Hi, I'm Agy 🐾" centered, 28px
- [ ] Two-paragraph intro:
  - First paragraph: `color: #374151`, `font-size: 18px`, `font-weight: 500`
  - Second paragraph: `color: #6B7280`, `font-size: 17px`
- [ ] All text fades in with staggered animation:
  ```css
  @keyframes fadeInUp {
      from { opacity: 0; transform: translateY(20px); }
      to { opacity: 1; transform: translateY(0); }
  }
  ```

## Suggested Questions Cards
- [ ] 2-column grid layout (1 column on mobile)
- [ ] Each card:
  - White background with `border: 2px solid #E5E7EB`
  - `border-radius: 12px`
  - `padding: 20px 24px`
  - Emoji icon (24px) on left
  - Question text on right (16px, `color: #2C363D`)
- [ ] Hover effect:
  ```css
  border-color: #8B5CF6;
  background: #F9FAFB;
  box-shadow: 0 4px 12px rgba(139, 92, 246, 0.12);
  transform: translateY(-2px);
  ```
- [ ] **Note:** Cards are visual inspiration only - users type queries in the input below. Do NOT add click handlers to cards.

## Input Area
- [ ] Text input styling:
  ```css
  padding: 20px 24px;
  font-size: 17px;
  border: 2px solid #E5E7EB;
  border-radius: 16px;
  background: #FAFAFA;
  ```
- [ ] Input focus state:
  ```css
  border-color: #8B5CF6;
  background: white;
  box-shadow: 0 0 0 3px rgba(139, 92, 246, 0.1);
  ```
- [ ] "Ask Agy 🐾" button:
  ```css
  background: #8B5CF6;
  color: white;
  border-radius: 12px;
  padding: 12px 32px;
  font-weight: 600;
  ```
- [ ] Hint text below: "Powered by OpenAI GPT-4 with semantic search across 115 project case studies" in 12px gray

## "How It Works" Panel (Expandable)
- [ ] Slides down with animation when button clicked
- [ ] Background: `#f8f9fa`
- [ ] Three feature sections with large emoji icons (32px) on left
- [ ] Example questions in gray rounded boxes
- [ ] Close button or auto-collapse functionality

## Colors Reference
- Purple gradient start: `#667eea`
- Purple gradient end: `#764ba2`
- Purple accent: `#8B5CF6`
- Purple hover: `#7C3AED`
- Success green: `#10B981`
- Dark text: `#2C363D`
- Medium text: `#374151`
- Light text: `#6B7280`
- Border: `#E5E7EB`
- Background: `#FAFAFA`

## Responsive Breakpoint
```css
@media (max-width: 768px) {
    .suggested-questions {
        grid-template-columns: 1fr !important;
    }
}
```

## What NOT to Change
- ❌ Don't replace the input handling logic (button click handler works correctly)
- ❌ Don't replace the semantic search integration
- ❌ Don't replace the navigation logic
- ❌ Don't change how session_state works
- ❌ Don't add click handlers to suggestion cards
- ✅ ONLY update CSS and HTML structure to match these visual specs
