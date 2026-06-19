# MattGPT Design Spec - Claude Working Agreement

## Critical Rules
Read these before every session.

- **Commit and push are separate gates, always.** A commit approval is not a push approval. Stop after committing and wait for an explicit "push" instruction. Never chain `git commit && git push`. Pushing to `origin/main` deploys to GitHub Pages, which is irreversible without a force push. (April 2026 incident: commit-then-push chain executed when only the commit had been approved.)
- **No Co-Authored-By lines. Ever.** No `--no-verify`. No force push to main.
- **No em dashes anywhere in this repo.** Not in docs, not in commits, not in this file. Use a colon, comma, or rewrite the sentence.
- **A number in this file is a bug.** CLAUDE.md contains no constants, counts, thresholds, or file inventories. Those live in `_data/facts.yml`.
- **Inclusive, bias-free language throughout.** Describe people by scope, skill, and impact, not by demographic proxies. In practice: no "X+ years of experience" or tenure-span framing in published prose.
- **Read existing `_layouts` and `_includes` before creating anything.** If a layout or include already exists, use it. Never author a layout from scratch.
- **All CSS in `assets/css/style.scss`.** No new stylesheets. No inline styles.

---

## What This Repo Is
The record of how MattGPT was designed and operates. Read as audit, not pitch.

Audience is mixed: recruiters, dev managers, PMs, designers, peers, decision-makers, and Matt. Not engineers only. Write judgment-led. Every doc should be honest before impressive: state real maturity including limits, never imply practices not in place.

---

## Three Content Layers
This is the backbone. Every piece of content is one of these three things:

**State** - facts derived from code: counts, model names, thresholds, file inventories, eval results. Generated, never hand-edited. Sourced from `llm_portfolio_assistant` into `_data/facts.yml` by the sync workflow. Prose references `{{ site.data.facts.* }}`. Test: if a script could produce it, it is state. Generate it, do not type it.

**Judgment** - decisions, principles, reasoning, voice. Authored and owned. This is the credential. Cannot be generated; must be written.

**Sausage** - session notes, in-progress audits, working artifacts. Lives in `archive/` (excluded from the Jekyll build) or `docs/working/`. Never promoted to a published doc.

**Rules:**
- Never hand-edit a fact into prose. Reference `_data/facts.yml`.
- Never write process exhaust into a published doc. It goes to `archive/` or `docs/working/`.
- If you are unsure which layer a piece of content belongs to, ask before writing it.

---

## Git Commit Rules

### Do
- Write clear, descriptive commit messages
- List changed files in commit body for documentation updates
- Stage specific files explicitly, never `git add .`

### Don't
- Don't use `--no-verify` or skip hooks
- Don't force push to main

### Commit Message Format
```
Short summary (50 chars max)

Detailed description of what changed and why.

Changes:
- Bullet list of key changes

Files Updated:
- List of modified files
```

---

## Facts and Accuracy

Facts are single-sourced from `llm_portfolio_assistant` into `_data/facts.yml` by the sync workflow. Prose references `{{ site.data.facts.* }}`. Hand-editing a fact in a doc is a bug: fix it by updating the source and re-running the sync.

**Config sync:** `_config.yml` and `_config_netlify.yml` must stay identical on identity fields (title, description, url). The deployed build reads `_config_netlify.yml`. If they diverge, the live site reads stale identity. When editing either file, update both.

**Wireframes** are the UI source of truth unless production has moved past them, in which case the running app wins. When they conflict, flag it explicitly. Do not silently pick one.

**No overclaiming.** Docs state real maturity including limits. Never imply enterprise CI/CD, automated pipelines, or practices not actually in place.

**Change-triggered diagram review:** When `ARCHITECTURE.md` or any module it references changes, re-verify the current-state diagrams in `docs/02-technical-architecture.md` and `docs/11-testing-and-quality.md` (and any other diagram describing live architecture). This check rides with the architectural change commit, not a calendar cadence.

---

## Editing the Site - Anti-Greenfield Rules

Before creating any file, read what already exists:

- **Layouts:** check `_layouts/` first. Copy and modify an existing layout before authoring one from scratch.
- **Includes:** check `_includes/` first. Reuse the nav include. Do not duplicate link lists.
- **CSS:** all styles go in `assets/css/style.scss`. No new stylesheets. No inline styles.
- **Breakpoints:** 375px / 768px / 1024px. Match the implementation repo.

---

## Nav and Structure

Primary nav groups: **The thinking** / **The build** / **Reference and roadmap**.

Docs 06, 07, and 08 are not in the primary nav:
- 07 (CSS Architecture) and 08 (Mobile Implementation) cross-link from Technical Architecture (02)
- 06 (Explore Stories Filter Redesign) cross-links from its build doc

`CONTEXT.md` is the active project status doc. It stays excluded from the Jekyll build and is never archived. It is always current.

`archive/` is excluded from the Jekyll build. Sausage goes there.

Component inventory and sitemap are state: generate or drop them. Do not hand-maintain them.

`README.md` and `index.md` carry the same identity. Keep them in sync.

---

## Documentation Restraint
Default to **not** creating new `.md` files. Most findings belong in commit messages or inline updates to existing docs.

Before creating a new `.md` file, justify why it can't go into:
- An existing doc or inline update
- A commit message
- A `CONTEXT.md` entry

Transitory files go in `docs/working/` with a lifecycle declaration and a defined deletion target. Permanent new top-level docs require explicit approval.

---

## File Structure
```
mattgpt-design-spec/
├── index.md                          # Landing page - the build record
├── README.md                         # Project overview (mirrors index.md identity)
├── CONTEXT.md                        # Active project status; excluded from build
├── CLAUDE.md                         # This file
├── _config.yml                       # Jekyll config - identity fields must match _config_netlify.yml
├── _config_netlify.yml               # Deployed build config - this is what the live site reads
├── docs/                             # Published specification
│   ├── 01-product-vision.md
│   ├── 02-technical-architecture.md
│   ├── 03-ux-design-process.md
│   ├── 04-building-mattgpt.md
│   ├── 05-agy-voice-guide.md
│   ├── 06-explore-stories-filter-redesign.md  # Cross-linked from build doc
│   ├── 07-css-architecture.md                 # Cross-linked from 02
│   ├── 08-mobile-implementation.md            # Cross-linked from 02
│   ├── 09-api-reference.md
│   ├── 10-data-model.md
│   ├── 11-testing-and-quality.md
│   ├── 12-data-pipeline.md
│   ├── audit-2026-06-15.md
│   └── working/                      # Transitory files with deletion targets
├── _layouts/                         # Jekyll layouts - reuse before creating
├── _includes/                        # Jekyll includes - reuse nav, don't duplicate
├── _data/
│   └── facts.yml                     # Single source of truth for all facts
├── assets/
│   └── css/
│       └── style.scss                # All CSS lives here
├── wireframes/                       # UI source of truth (unless app has moved past)
├── images/
├── brand-kit/
└── archive/                          # Sausage; excluded from Jekyll build
```
