---
title: Keeping the spec in sync
---

# Keeping the spec in sync

Documentation drifts from code in every real system. The cause is
duplication: the same fact declared in a doc, a rules file, and a test
expectation. When it changes in the code, the copies lag, and a reader
who spot-checks two of them finds a contradiction in the first five
minutes.

The correction is to stop declaring facts and start deriving them.

## The pipeline

A GitHub Actions job runs on every push to the source repository. It
derives volatile facts from the running code and writes them to
`_data/facts.yml`; the docs consume them through Jekyll Liquid
references. Currently synced: model name, confidence thresholds, intent
family count, BDD scenario and feature-file counts, and story count.

A synced fact changes in exactly one place, the code. The docs that read
it stay current with no review queue.

## What the pipeline does not cover

Behavioral prose and claims about code paths. "Agy asks a clarifying
question when confidence is low" is not derivable from a constant, and
no pipeline will tell you when it stops being true. That layer needs
periodic human review.

## The review layer

A full pass over all 13 docs against the running implementation, most
recently June 2026. That audit found the same number living in five
docs, a stale headline metric, and brand vocabulary the product retired
in March still sitting in test expectations. It also found `CLAUDE.md`
declaring `DEFAULT_CHAT_MODEL = "gpt-4o-mini"` and
`CONFIDENCE_LOW = 0.15` when `config/constants.py`, the declared single
source of truth, had `gpt-4o` and `0.20`. The spec was right and the
rules file was stale, which is the case for deriving rather than
restating. Fixed in the source repo.

Everything the June pass corrected by hand is now either synced or
recorded here. The next pass covers prose only.
