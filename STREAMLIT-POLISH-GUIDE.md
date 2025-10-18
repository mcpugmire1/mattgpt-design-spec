# Streamlit App Polish Guide
## Ship Your Portfolio App in 1-2 Weeks

**Goal:** Apply "lipstick to the pig" - make the current Streamlit app presentable enough to use NOW for job search while you build the React version.

---

## Week 1: Brand Refresh (3-5 hours)

### Day 1: Apply Agy Brand Colors (1 hour)

**File:** `/ui/components.py`

Replace the current blue color scheme with MattGPT brand colors:

```python
# Find and replace in css_once() function:

# OLD (Blue scheme):
background: linear-gradient(135deg, #4a90e2, #6bb6ff);
color: #4a90e2 !important;
background: radial-gradient(circle at 30% 70%, rgba(74, 144, 226, 0.1) 0%, transparent 50%);

# NEW (Agy Purple/Indigo scheme):
background: linear-gradient(135deg, #8B5CF6, #6366F1);  /* Purple → Indigo */
color: #6366F1 !important;  /* Primary Indigo */
background: radial-gradient(circle at 30% 70%, rgba(139, 92, 246, 0.1) 0%, transparent 50%);

# Update ALL instances of #4a90e2 → #6366F1
# Update ALL instances of #6bb6ff → #8B5CF6
```

**Brand Color Palette:**
```python
# Add this at the top of css_once() for consistency:
:root {
    --primary-purple: #8B5CF6;
    --primary-indigo: #6366F1;
    --primary-blue: #3498DB;
    --success-green: #2ECC71;
    --success-dark: #27AE60;
    --dark-navy: #2C3E50;
    --medium-gray: #6B7280;
    --light-gray: #E5E7EB;
}
```

---

### Day 2: Add Agy Logo (30 minutes)

**File:** `app.py` (line 307-310)

```python
# Current:
st.set_page_config(
    page_title="MattGPT — Matt's Story",
    page_icon="🤖",
    layout="wide",
)

# Updated with Agy:
st.set_page_config(
    page_title="MattGPT — AI Portfolio Assistant",
    page_icon="🐕",  # Plott Hound emoji (or use actual Agy image)
    layout="wide",
    initial_sidebar_state="collapsed",  # Cleaner without sidebar
)
```

**Optional:** Add Agy logo to header
```python
# In render_home_hero_and_stats() or at top of app:
st.image("/path/to/agy_transparent.png", width=100)
st.markdown("# MattGPT")
```

---

### Day 3-4: Update Ask MattGPT System Prompt (2 hours)

**Current Challenge:** Find where you define the system prompt and replace it with the Agy approach.

**New System Prompt (based on your design spec):**

```python
# Add this to app.py (or extract to config file):

SYSTEM_PROMPT = """
**Your Purpose: The Credibility Engine**

You are MattGPT, an AI assistant that helps users discover Matt Pugmire's professional experience through structured STAR stories (Situation, Task, Action, Result).

**Core Directive: Anchor Every Answer in Proof**

DO:
✅ Anchor every answer in specific projects (citing Client, Title, Outcome)
✅ Lead with outcomes, then methodology (e.g., "Achieved 4x acceleration by implementing...")
✅ Infer user intent (Interview Prep, Due Diligence, Pitch) to tailor the response structure
✅ Use semantic search across STAR stories, 5P taxonomy, and competencies to prioritize relevance

**The Archetype: Trusted, Pragmatic Advisor**

Tone: Warm, confident, professional—never robotic or buzzword-heavy
Voice blend:
- Strategic Advisor (60%) - Executive-ready, outcome-focused
- Pragmatic Operator (30%) - Grounded in results, implementation-focused

**Non-Negotiable Guardrails:**

❌ NO generic career advice or philosophical answers
   → Only discuss Matt's specific project experience
   → No hypotheticals, no general industry commentary

❌ NO corporate buzzword soup or robotic language
   → Use concrete, human language
   → Avoid jargon without context

❌ NO unsupported claims
   → Every statement must reference a specific project
   → If no relevant project exists, say so directly

**Response Structure:**

1. **Lead with the outcome** (What was achieved)
2. **Cite the project** (Client, Title, brief context)
3. **Explain the approach** (How it was done)
4. **Provide evidence** (Metrics, results from STAR)
5. **Offer next steps** (Related projects, follow-up questions)

**Example Good Response:**
"Matt has extensive experience in payment modernization. At JPMorgan Chase, he led a Core Banking Transformation that migrated 10M+ accounts to a cloud-native platform, reducing transaction processing time by 60% (from 4s to 1.5s average).

The approach involved:
- Architecting a microservices-based payment gateway
- Implementing real-time data validation
- Leading a cross-functional team of 25 engineers

This demonstrates strong capabilities in enterprise architecture, cloud migration, and large-scale system design. Would you like to explore other payment transformation projects or dive deeper into the technical architecture?"

**Example Bad Response (DON'T DO THIS):**
"Matt is an experienced technology leader with expertise in various domains. He has worked on multiple transformation initiatives and demonstrates strong leadership skills across different industries."
→ ❌ Too vague, no specific projects, buzzword-heavy, no proof

**Remember:** You are the Credibility Engine. Every response should make the user think: "Matt consistently delivers measurable transformation results — and here's the specific proof."
"""
```

**Where to add it:**
Find where you make OpenAI API calls and add this as the system message:

```python
# Look for something like this in your ask_mattgpt logic:
response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[
        {"role": "system", "content": SYSTEM_PROMPT},  # ← Add this
        {"role": "user", "content": user_question},
        # ... context/sources
    ]
)
```

---

### Day 5: Quick UI Improvements (1 hour)

**Small touches that make a big difference:**

1. **Better Hero Text**
```python
# Update hero in render_home_hero_and_stats():
st.markdown("""
    <div class='matt-hero'>
        <h1>MattGPT</h1>
        <p>AI-powered portfolio assistant showcasing 115+ enterprise transformation projects through structured STAR stories and intelligent search.</p>
    </div>
""", unsafe_allow_html=True)
```

2. **Add Footer with Links**
```python
# At bottom of app.py:
st.markdown("---")
st.markdown("""
<div style='text-align: center; color: #6B7280; font-size: 14px;'>
    <p>Built by <a href='https://www.linkedin.com/in/mattpugmire/' target='_blank'>Matt Pugmire</a>
    | <a href='https://github.com/mcpugmire1/llm_portfolio_assistant' target='_blank'>View Source</a>
    | <a href='https://github.com/mcpugmire1/mattgpt-design-spec' target='_blank'>Design Spec</a></p>
    <p style='margin-top: 8px;'>Powered by OpenAI GPT-4, Pinecone, and Streamlit</p>
</div>
""", unsafe_allow_html=True)
```

3. **Improve Page Titles**
```python
# Make sure each tab/section has clear titles:
st.markdown("## 🏠 Home")  # or "## 🔍 Explore Stories", "## 💬 Ask MattGPT", etc.
```

---

## Week 2: Content & Testing (2-3 hours)

### Day 6: Add "About This App" Section (30 min)

Create a simple "About" or "How It Works" section:

```python
# Add to Home tab or create dedicated About tab:
with st.expander("ℹ️ About MattGPT"):
    st.markdown("""
    **What is MattGPT?**

    MattGPT is an AI-powered portfolio assistant that showcases Matt Pugmire's professional experience through:

    - 📊 **115+ Enterprise Projects** - Banking, healthcare, retail transformations
    - 📖 **STAR-Formatted Stories** - Situation, Task, Action, Result methodology
    - 🤖 **Intelligent Search** - Semantic + keyword hybrid search via Pinecone
    - 💬 **Conversational AI** - Ask questions, get answers with specific project citations

    **How to Use:**
    - **Explore Stories:** Browse by industry, capability, or client
    - **Ask MattGPT:** Ask questions about Matt's experience
    - **Search:** Use keywords or natural language to find relevant projects

    **Tech Stack:** Python, Streamlit, OpenAI GPT-4, Pinecone Vector Database

    [View Design Spec](https://mcpugmire1.github.io/mattgpt-design-spec/) |
    [View Source Code](https://github.com/mcpugmire1/llm_portfolio_assistant)
    """)
```

---

### Day 7: Testing & Polish (1-2 hours)

**Quick Testing Checklist:**

1. **Test All Tabs**
   - [ ] Home page loads
   - [ ] Explore Stories works (table/card/timeline views)
   - [ ] Ask MattGPT chat works
   - [ ] About page loads

2. **Test Search**
   - [ ] Search returns results
   - [ ] Filters work
   - [ ] Story detail pages load

3. **Test Ask MattGPT**
   - [ ] New system prompt works
   - [ ] Responses cite specific projects
   - [ ] Sources are clickable
   - [ ] Tone is warm and professional (not robotic)

4. **Visual Check**
   - [ ] Brand colors applied (purple/indigo)
   - [ ] No broken images
   - [ ] Spacing looks clean
   - [ ] Mobile responsive (test on phone)

5. **Performance**
   - [ ] Page loads in < 3 seconds
   - [ ] Search responds in < 2 seconds
   - [ ] Ask MattGPT responds in < 10 seconds

---

## Deploy & Share (1 hour)

### Update Streamlit Cloud Deployment

1. **Commit Changes**
```bash
cd /Users/matthewpugmire/Projects/portfolio/llm_portfolio_assistant
git add .
git commit -m "Polish app with Agy branding and improved system prompt"
git push
```

2. **Wait for Auto-Deploy**
   - Streamlit Cloud should auto-deploy from main branch
   - Check https://askmattgpt.streamlit.app/ in 2-3 minutes

3. **Update README**
   - Add screenshots
   - Update description with new branding
   - Link to design spec

---

## Share With Recruiters

### Create a One-Pager (Email Template)

```markdown
Subject: Portfolio Demo: MattGPT - AI-Powered Career Assistant

Hi [Name],

I've built an AI-powered portfolio assistant that showcases my experience through 115+ enterprise transformation projects. Rather than a static resume, you can:

🔍 **Explore Projects:** Filter by industry, capability, or client
💬 **Ask Questions:** Natural language Q&A with specific project citations
📊 **See Proof:** Every answer backed by structured STAR stories with metrics

**Try it:** https://askmattgpt.streamlit.app/

**Design Spec:** https://mcpugmire1.github.io/mattgpt-design-spec/

This demonstrates:
- AI/ML implementation (RAG architecture, GPT-4, vector search)
- Product thinking (UX design, information architecture)
- Full-stack development (Python, Streamlit, cloud deployment)

Happy to discuss how I built this or answer any questions about my experience.

Best,
Matt
```

---

## Optional: Advanced Polish (If You Have Time)

### Add Analytics (30 min)
```python
# Track usage in your app:
import streamlit.components.v1 as components

# Google Analytics or simple logging:
def log_event(event_type, metadata=None):
    # Log to file or external service
    print(f"Event: {event_type}, Data: {metadata}")

# Call in key actions:
log_event("search_executed", {"query": query, "results_count": len(results)})
log_event("ask_question", {"question": question})
```

### Add Sharing Links (15 min)
```python
# Let users share interesting stories:
st.markdown(f"""
<a href='https://www.linkedin.com/sharing/share-offsite/?url=https://askmattgpt.streamlit.app/story/{story_id}' target='_blank'>
    Share on LinkedIn
</a>
""", unsafe_allow_html=True)
```

---

## Success Criteria

**You're ready to share when:**

- ✅ App loads without errors
- ✅ Agy brand colors applied throughout
- ✅ Ask MattGPT uses new system prompt (warm, concrete, proof-based)
- ✅ All features work (search, browse, chat)
- ✅ You've tested it on desktop + mobile
- ✅ You can explain how it works in 30 seconds

**Target:** Ship in 7-10 days max. Don't perfect it - just make it presentable.

---

## After You Ship

**Week 3+: Use It While Building React**

- Share with 5-10 recruiters/hiring managers
- Gather feedback
- Track which features get used most
- Use feedback to inform React rebuild priorities
- Keep improving Streamlit version until React is ready

**The React version can be perfect. This version just needs to be good enough.**

---

## Need Help?

If you get stuck on any of these steps, let me know and I can:
1. Generate the exact code snippets you need
2. Review your changes before you push
3. Help debug any issues
4. Draft the recruiter outreach email

**Remember:** Ship now, perfect later. Get this in front of people ASAP! 🚀
