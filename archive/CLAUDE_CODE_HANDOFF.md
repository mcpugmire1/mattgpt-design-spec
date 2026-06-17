# Claude Code Handoff Guide - Ask MattGPT Pages

## Current State ✅
- **Functionality works perfectly** - button submits, backend processes, messages render
- **Just needs visual styling** to match wireframes
- Two functions to style: `render_landing_page()` and `render_conversation_view()`

## Opening Prompt for New Claude Code Session

```
I need CSS styling updates for ui/pages/ask_mattgpt.py.

IMPORTANT CONTEXT:
- The app works perfectly - all functionality is correct
- I just need the visual appearance to match my wireframe designs
- Two functions need styling: render_landing_page() and render_conversation_view()

CRITICAL RULES:
- DO NOT change any Python logic or function calls
- DO NOT modify button handlers or input processing
- DO NOT change transcript structure or session state
- ONLY update CSS in st.markdown() blocks
- ONLY update HTML class names to match CSS selectors

I'll provide visual spec checklists for both views.
Let's start with render_landing_page().
```

---

## For Landing Page

**Give Claude Code:**
```
Update render_landing_page() CSS to match this spec:

[paste VISUAL_SPEC_CHECKLIST.md]

Key requirements:
1. Purple gradient header with glass-morphism "How Agy searches" button
2. Status bar with green pulse animation on "active" status
3. Suggested question cards in 2-column grid with hover effects
4. Input field with purple focus ring
5. All CSS classes must be applied to corresponding HTML elements

DO NOT:
- Change the button click handler
- Modify input processing
- Touch any session_state logic
- Change how stories are passed

Show me what CSS currently exists, then what needs to change.
```

**Use file:** `VISUAL_SPEC_CHECKLIST.md`

---

## For Chat/Conversation Page

**Give Claude Code:**
```
Update render_conversation_view() CSS to match this spec:

[paste VISUAL_SPEC_CHAT_INTERFACE.md]

Key requirements:
1. User messages: light blue background (#e3f2fd)
2. AI messages: white card with purple left border (4px solid #8B5CF6)
3. Source links as interactive chips with hover effects
4. Input at bottom with purple focus state

DO NOT:
- Change message rendering logic
- Modify transcript iteration
- Touch source link generation
- Change chat history processing

Show me what CSS currently exists, then what needs to change.
```

**Use file:** `VISUAL_SPEC_CHAT_INTERFACE.md`

---

## Key Differences Between Pages

### Landing Page Has:
- Welcome section with Agy intro
- Suggested question cards (2-column grid)
- Large centered input area
- Avatar animations on hover

### Chat Page Has:
- Message history (user + AI messages)
- Thinking indicator with animated tennis ball
- Source links below AI responses
- Action buttons (helpful, copy, share)
- Bottom-fixed input area

---

## If Claude Code Asks Questions

**"Should I rewrite the component structure?"**
→ NO. Only update CSS.

**"Should I change how messages render?"**
→ NO. Only update the CSS styling of messages.

**"Should I replace the input handling?"**
→ NO. Only update the CSS styling of the input.

**"Should I update the semantic search logic?"**
→ NO. Don't touch any logic. Only CSS.

**"Can I refactor this to be cleaner?"**
→ NO. Just update CSS to match the specs.

---

## Validation Checklist

After Claude Code makes changes, verify:

### Landing Page
- [ ] Header has purple gradient
- [ ] Status bar shows green pulse animation
- [ ] Suggested question cards hover smoothly
- [ ] Input has purple focus ring
- [ ] All animations work (fadeInUp, pulse)
- [ ] Page still functions exactly the same

### Chat Page
- [ ] User messages are light blue
- [ ] AI messages have purple left border
- [ ] Thinking indicator animates (tennis ball)
- [ ] Source links hover correctly
- [ ] Action buttons work
- [ ] Page still functions exactly the same

---

## What Success Looks Like

✅ **Good outcome:**
- Styling matches wireframes pixel-perfect
- All functionality still works
- No errors in console
- Chat history, search, navigation all intact

❌ **Bad outcome:**
- Claude Code rewrote components
- Functionality broke
- Had to restore from backup
- Lost hours of work

---

## Emergency Recovery

If Claude Code breaks things:
1. Use git to revert: `git checkout HEAD -- <file>`
2. Or restore from your backup
3. Start over with clearer instructions
4. Be VERY explicit: "DO NOT TOUCH THE LOGIC"

---

## Pro Tips

1. **Have Claude Code work on ONE page at a time**
2. **Review each change before accepting it**
3. **If he starts rewriting functions, STOP HIM immediately**
4. **Keep reminding: "CSS only, no logic changes"**
5. **Use the checklists as a strict boundary**

---

## The Golden Rule

**If Claude Code suggests anything beyond CSS changes, say:**

"Stop. I don't need you to refactor or improve the code. I only need you to update the CSS styling to match the wireframe specs. Please show me just the CSS changes."
