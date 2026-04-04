# Claude Code Source Guide

This repository no longer contains the originally leaked Claude Code source tree.

The tracked `src/` snapshot has been removed. What remains here is documentation derived from a prior review of that snapshot:

- architecture notes
- subsystem walkthroughs
- appendix indexes
- an audit checklist for documentation completeness

The notes are intentionally high-level and non-reconstructive:

- no raw source files
- no source-shaped pseudocode
- no implementation-complete type sketches
- no attempt to preserve code-equivalent behavior

## What's In This Repo

The primary artifact is the guide under [`docs/claude-code-source-guide`](./docs/claude-code-source-guide).

Start here:

1. [`docs/claude-code-source-guide/README.md`](./docs/claude-code-source-guide/README.md)
2. [`docs/claude-code-source-guide/00-checklist.md`](./docs/claude-code-source-guide/00-checklist.md)
3. [`docs/claude-code-source-guide/appendices/21-third-party-audit-checklist.md`](./docs/claude-code-source-guide/appendices/21-third-party-audit-checklist.md)

## Important Note

Many guide chapters reference paths under `src/` because they were written against the removed snapshot. Those references are preserved for provenance, but the source files themselves are no longer included in this repository.

The remaining material is meant to preserve architectural understanding, not implementation exhaustiveness.
