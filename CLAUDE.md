# MattGPT Design Spec - Claude Working Agreement

## Project Overview
Design specification and documentation repository for MattGPT - an AI-powered portfolio assistant. This repo contains product strategy, technical architecture, UX design, and wireframes.

## Git Commit Rules

### Do
- Write clear, descriptive commit messages
- List changed files in commit body for documentation updates
- Stage specific files explicitly (not `git add .`)

### Don't
- **NEVER add Co-Authored-By lines with Anthropic email**
- Don't use `--no-verify` or skip hooks
- Don't force push to main

### Commit and Push Are Separate Gates
**Commit and push are two separate gates requiring two separate approvals.** A commit approval ("commit", "yes commit") is not a push approval. After committing, stop and wait. Do not run `git push` until the user explicitly types "push", "go ahead and push", or similar. Combining `git commit && git push` in a single command is not acceptable. This rule exists because pushing to `origin/main` deploys to GitHub Pages — it is irreversible without a force push.

April 2026 incident (in the MattGPT repo): a commit-then-push chain executed when only the commit had been approved. The push triggered an unauthorized production deploy. The fix is procedural — separate gates, separate words. The same rule applies here.

### Commit Message Format
```
Short summary (50 chars max)

Detailed description of what changed and why.

Changes:
- Bullet list of key changes

Files Updated:
- List of modified files
```

**NO Co-Authored-By line. Ever.**

## Documentation Standards

### Accuracy
- Wireframes in `/wireframes/` are the golden source for UI
- Architecture diagrams must match implementation in llm_portfolio_assistant
- All thresholds/constants must match `config/constants.py` in implementation repo

### Updates
- Update `CONTEXT.md` when making significant changes
- Update version numbers in documentation files
- Update "Last Updated" dates

### Documentation Restraint
Default to **not** creating new markdown files. Most analysis, investigation results, and intermediate findings belong in commit messages, inline updates to existing docs, or CONTEXT.md entries — not in standalone files.

Before creating a new `.md` file, justify why it can't go into:
- An existing doc (CONTEXT.md, or the numbered docs under `docs/`)
- A commit message
- An inline update to the relevant spec section

If a new file is genuinely needed and is transitory, it goes in `docs/working/` with a lifecycle declaration and a defined deletion target. Permanent new docs at the top level require explicit user approval.

### Sync with Implementation
When implementation changes in `llm_portfolio_assistant`:
1. Update relevant sections in `docs/02-technical-architecture.md`
2. Update thresholds/constants across all files
3. Update architecture diagrams if flow changes
4. Update CONTEXT.md with changes
5. Commit with descriptive message

## File Structure
```
mattgpt-design-spec/
├── CLAUDE.md              # This file - working agreement
├── CONTEXT.md             # Current project status
├── README.md              # Project overview
├── /docs/                 # Strategic documentation
│   ├── 01-product-vision.md
│   ├── 02-technical-architecture.md
│   ├── 03-ux-design-process.md
│   ├── 04-building-mattgpt.md
│   └── 05-agy-voice-guide.md
├── /wireframes/           # Interactive HTML prototypes
├── /images/architecture/  # Diagrams and flows
└── /brand-kit/            # Brand assets
```

## Key Principles
- **Documentation accuracy** - Must match implementation
- **Version control** - Track all changes in git
- **Clean commits** - No Co-Authored-By lines
- **Context preservation** - Update CONTEXT.md regularly
