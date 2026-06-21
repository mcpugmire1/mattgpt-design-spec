# MattGPT: The Build

The design and build record for [MattGPT](https://askmattgpt.streamlit.app) — an AI-powered portfolio assistant built on a RAG pipeline over 100+ STAR-structured career stories.

Read it as the audit, not the advertisement. The published spec is at [mcpugmire1.github.io/mattgpt-design-spec](https://mcpugmire1.github.io/mattgpt-design-spec).

Source code: [llm_portfolio_assistant](https://github.com/mcpugmire1/llm_portfolio_assistant)

---

## Repository structure

```
mattgpt-design-spec/
├── index.md                        # Landing page — the build record
├── README.md                       # This file
│
├── docs/                           # Published specification
│   ├── 01-product-vision.md
│   ├── 02-technical-architecture.md
│   ├── 03-ux-design-process.md
│   ├── 04-building-mattgpt.md
│   ├── 05-agy-voice-guide.md
│   ├── 07-css-architecture.md
│   ├── 08-mobile-implementation.md
│   ├── 09-api-reference.md
│   ├── 10-data-model.md
│   ├── 11-testing-and-quality.md
│   ├── 12-data-pipeline.md
│   ├── audit-2026-06-15.md
│   └── working/                    # Transitory drafts (lifecycle-declared)
│
├── _layouts/                       # Jekyll layout overrides
├── _includes/                      # Shared partials (doc-nav, mermaid)
├── _data/                          # Auto-generated facts (CI pipeline)
│   └── facts.yml                   # Derived from source repo on every push
│
├── assets/css/style.scss           # Brand styling
├── wireframes/                     # Interactive HTML prototypes
├── images/                         # Diagrams, architecture, screenshots
├── brand-kit/                      # Logos, avatars, favicons
└── archive/                        # Retired session artifacts
```

---

*Built by [Matthew Pugmire](https://www.linkedin.com/in/mattpugmire/)*
