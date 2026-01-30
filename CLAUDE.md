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
