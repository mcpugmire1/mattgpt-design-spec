# Ask MattGPT Chat Interface - Visual Spec Checklist

**CRITICAL: DO NOT rewrite working code. Only update CSS and HTML styling to match these specs.**

## Header Section (Same as Landing)
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

## Status Bar (Same as Landing)
- [ ] Three status items horizontally
- [ ] "Semantic search: active" with green pulse dot
- [ ] "Pinecone index: ready"
- [ ] "130 stories: indexed"
- [ ] Green dot pulse animation on active status

## Chat Container Layout
- [ ] Messages area takes flex: 1 (fills available space)
- [ ] Background: `#fafafa`
- [ ] Scrollable overflow-y
- [ ] Padding: `30px`

## Message Structure

### User Messages
- [ ] Layout: Avatar (left) + Content bubble (right)
- [ ] Avatar: 40px circle with person icon (👤)
  ```css
  background: #7f8c8d;
  color: white;
  opacity: 0.5;
  width: 40px;
  height: 40px;
  ```
- [ ] Content bubble:
  ```css
  background: #e3f2fd;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  ```

### AI Messages (Agy Responses)
- [ ] Layout: Avatar (left) + Enhanced content card (right)
- [ ] Avatar: 48px circle with Agy image
  ```css
  background: white;
  border: 2px solid #e0e0e0;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 4px;
  ```
- [ ] Enhanced content card:
  ```css
  background: white;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
  border-left: 4px solid #8B5CF6;
  ```

## Thinking Indicator
- [ ] Shows at top of AI response while "thinking"
- [ ] Layout: Animated tennis ball icon + "🐾 Tracking down insights..." text
- [ ] Style:
  ```css
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  background: #f0f0f0;
  border-radius: 6px;
  font-size: 13px;
  color: #7f8c8d;
  margin-bottom: 12px;
  ```
- [ ] Animation: Tennis ball cycles through 3 frames (chase_48px_1.png, _2.png, _3.png)
- [ ] Fade out after 2 seconds:
  ```css
  animation: fadeOutSmooth 0.5s ease-out 2s forwards;
  
  @keyframes fadeOutSmooth {
      to {
          opacity: 0;
          transform: translateY(-8px);
      }
  }
  ```

## Message Text Styling
- [ ] Font size: 15px
- [ ] Color: `#2c3e50`
- [ ] Line height: 1.6
- [ ] Strong text should be bold for emphasis

## Source Links Section
- [ ] Appears below message text
- [ ] Separated by top border: `border-top: 1px solid #e0e0e0`
- [ ] Margin top: 16px, padding top: 16px
- [ ] Title: "📚 Related Projects" (12px, uppercase, `color: #7f8c8d`)
- [ ] Each link styled as a chip:
  ```css
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: #F3F4F6;
  border: 2px solid #E5E7EB;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  color: #2563EB;
  text-decoration: none;
  transition: all 0.2s ease;
  ```
- [ ] Link hover state:
  ```css
  background: #EEF2FF;
  border-color: #8B5CF6;
  transform: translateY(-1px);
  ```
- [ ] Icon color: `#8B5CF6`

## Action Buttons (Below AI Response)
- [ ] Three buttons: "👍 Helpful", "📋 Copy", "🔗 Share"
- [ ] Style:
  ```css
  padding: 6px 12px;
  background: white;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  font-size: 12px;
  color: #555;
  cursor: pointer;
  transition: all 0.2s ease;
  ```
- [ ] Hover state:
  ```css
  background: #f5f5f5;
  border-color: #ccc;
  ```
- [ ] Clicked state (for Helpful button):
  ```css
  background: #10B981;
  color: white;
  border-color: #10B981;
  /* Add checkmark: " ✓" */
  ```

## Input Area (Bottom Fixed)
- [ ] Fixed at bottom of chat container
- [ ] Background: white
- [ ] Border top: `2px solid #e0e0e0`
- [ ] Padding: `20px 30px`
- [ ] Input styling:
  ```css
  padding: 14px 18px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 15px;
  ```
- [ ] Input focus state:
  ```css
  border-color: #8B5CF6;
  box-shadow: 0 0 0 3px rgba(139, 92, 246, 0.1);
  ```
- [ ] "Ask Agy 🐾" button:
  ```css
  padding: 14px 28px;
  background: #8B5CF6;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 600;
  ```
- [ ] Button hover:
  ```css
  background: #7C3AED;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(139, 92, 246, 0.3);
  ```
- [ ] Hint text: "Powered by OpenAI GPT-4 with semantic search across 115 project case studies" (12px, centered, gray)

## Message Spacing
- [ ] Gap between avatar and content: `12px`
- [ ] Margin between messages: `24px`
- [ ] Gap between action buttons: `8px`

## How It Works Panel (Same as Landing)
- [ ] Slides down with animation when button clicked
- [ ] Background: `#f8f9fa`
- [ ] Three feature sections with emoji icons
- [ ] Example questions in gray boxes
- [ ] Follows same styling as landing page

## Colors Reference (Same as Landing)
- Purple gradient start: `#667eea`
- Purple gradient end: `#764ba2`
- Purple accent: `#8B5CF6`
- Purple hover: `#7C3AED`
- Success green: `#10B981`
- User message bg: `#e3f2fd`
- AI message border: `#8B5CF6` (left border, 4px)
- Link blue: `#2563EB`
- Dark text: `#2c3e50`
- Gray text: `#7f8c8d`
- Border: `#e0e0e0`

## Avatar Assets
- User avatar: 👤 emoji or icon, 40x40px
- Agy avatar (48x48px): `/brand-kit/chat_avatars/agy_avatar_48_dark.png`
- Agy avatar (64x64px header): `/brand-kit/chat_avatars/agy_avatar_64_dark.png`
- Thinking frames: `/brand-kit/thinking_indicator/chase_48px_[1-3].png`

## Key Visual Differences from Landing Page
1. **Messages area** instead of welcome section
2. **User messages** have light blue background (#e3f2fd)
3. **AI messages** have purple left border (4px solid #8B5CF6)
4. **Thinking indicator** with animated tennis ball
5. **Source links** displayed as interactive chips below response
6. **Action buttons** for helpful/copy/share
7. Input placeholder says "💬 Ask a follow-up question..." instead of longer text

## What NOT to Change
- ❌ Don't replace message rendering logic (the _render_ask_transcript function)
- ❌ Don't replace chat history management
- ❌ Don't replace semantic search integration
- ❌ Don't change how source links are generated
- ❌ Don't replace the thinking animation logic
- ❌ Don't modify the transcript structure or card format
- ✅ ONLY update CSS and HTML structure to match these visual specs

## Implementation Notes

### Thinking Animation
- The tennis ball should cycle through 3 frames every ~300ms (0.9s full cycle)
- After 2 seconds, the entire thinking indicator should fade out
- This is typically shown while waiting for semantic search + LLM response

### Source Links
- Should be generated dynamically from your semantic search results
- Each link should map to a specific story in your 130+ story database
- On click, should navigate to full story detail or Explore Stories filtered view

### Action Buttons
- "Helpful" should track positive feedback (store in session state or database)
- "Copy" should copy the message text to clipboard
- "Share" could generate a shareable link or open share dialog
