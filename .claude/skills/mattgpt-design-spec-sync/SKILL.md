---
name: mattgpt-design-spec-sync
description: Drift pass that checks the MattGPT design spec docs against the app repo's origin/main, fixes them one doc per commit with Matt's approval, verifies the live site after Matt pushes, and hands app-side defects to the backlog session. Use when asked to sync, audit, or run a drift pass on the spec.
---

# MattGPT design spec sync

Brings the published spec in line with what `llm_portfolio_assistant` actually
does. The spec repo's CLAUDE.md applies in full; this skill adds the pass itself.

## Scope

- Source of truth is the app repo's `origin/main`. Run `git fetch` there first.
  Never read the working tree: another session may have uncommitted work in it.
  Use `git show origin/main:<path>` and `git grep <pattern> origin/main -- <paths>`.
- Pull the spec repo first (`git pull`; rebase and autostash are configured).
- Targets: published pages (`index.md`, `docs/*.md`), the nav include,
  wireframes, `CONTEXT.md` and `README.md`. `archive/` is out of scope.
- `CLAUDE.md` is Matt's: never edit it. Report its drift in the pass's
  report and in `CONTEXT.md` Next Steps.
- Label every finding as one of:
  - **supported**: code on origin/main backs it as written
  - **stale**: it was true once; the code has moved on
  - **overclaimed**: no code backs it, or the code does less than the claim
  - **unverified**: no source found yet. Report it; do not edit it.
- Each finding gives the spec file and line, the claim, and the source it
  was checked against (file and function or constant, by name), with a commit
  hash for anything historical.

## Rules

- **No typed app numbers or names.** Counts, thresholds, model names and
  copy that the app owns are either a synced fact
  (`{{ site.data.facts.* }}` from `scripts/extract_facts.py`) or a pointer to
  the doc or source file that holds them. If the fact is not synced yet,
  point to the source and add it to the follow-ups list as a candidate for
  extract_facts.py. A measured result that cannot be synced (an eval run) is
  stated with its date and evidence path.
- **Check what a synced fact measures.** Read how extract_facts.py computes a
  fact before using it; a name like "summary" or "count" does not guarantee
  the value is a measured result. If a fact doesn't mean what its name says,
  don't use it, and hand it off as an app-side defect.
- **Check before changing.** Read the cited code before writing. If a claim
  can't be traced, label it unverified and ask; don't soften or delete it on a
  guess.
- **One doc per commit, shown first.** Show the diff and the commit message
  (CLAUDE.md format), wait for approval, then commit. A commit approval is not
  a push approval. Matt pushes.
- **First-person voice is Matt's.** For judgment prose (index.md, the "why"
  sections), propose the wording and let Matt approve or rewrite it.
- **Never write to the app repo.** No edits, commits, branches or stashes in
  `llm_portfolio_assistant`.
- Historical sections (build timelines, dated snapshots) stay as written if
  they are dated; mark a whole stale page as a dated snapshot rather than
  rewriting it.

## Before Matt pushes

Static checks on the touched pages:
- every `site.data.facts.*` and `site.data.page_dates[...]` key exists in `_data/`
- every relative link resolves to a file, and every `#anchor` matches a heading

## After Matt pushes

No local Jekyll build; GitHub Pages builds the site.

1. Wait for the latest "pages build and deployment" run in the spec repo's
   Actions to succeed. An earlier run that was cancelled because a newer push
   superseded it is not a failure. The push also triggers a
   "chore: sync site data" commit; remind Matt to pull before the next edit.
2. Fetch each touched page on the live site
   (https://mcpugmire1.github.io/mattgpt-design-spec/) with cache busting and
   confirm: the new wording is present, the old wording is gone, no raw
   `{{` or `{%` remains, and every internal link and anchor returns a page.
3. Report per page. A page still showing old content means the deploy has
   not landed; poll, don't re-edit.

## Hand-offs

- **App-side defects** (wrong code, a fact that isn't synced, a synced value
  that misstates what it measures): don't fix, don't work around. Write ticket
  text for the backlog session: title, what the code does (file and function,
  origin/main hash), what the spec needed, suggested fix.
- **Follow-ups waiting on app tickets**: end the pass with the list of spec
  changes blocked on open app work, each with the ticket key and the docs it
  touches. Keep the live list in `CONTEXT.md` Next Steps so the next pass
  starts from it.

## Report

End with: findings by label, commits made (hash and doc), what is waiting on
Matt's push, live-site results, ticket text for app defects, and the
follow-up list.
