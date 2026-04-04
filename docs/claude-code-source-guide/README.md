# Claude Code Source Guide

This guide turns a prior review of the leaked Claude Code source snapshot into an agent-first manual for people building coding agents, agent runtimes, and terminal-native AI products.

Note:
- the original `src/` snapshot analyzed by this guide has been removed from the repository
- chapter links to `src/...` are preserved as provenance from that analyzed snapshot, even though those files are no longer present locally

The snapshot is useful because it exposes the product's real control-flow seams:
- startup fast paths
- the query loop and recovery logic
- tool registration and permission handling
- shell parsing and fail-closed validation
- MCP client plumbing
- bridge and remote-session infrastructure
- memory, compaction, and transcript persistence
- agent, task, and worktree orchestration

This guide is source-grounded:
- `Observed` means directly supported by code or comments in this snapshot.
- `[Inference]` means the code strongly implies a design choice, but the snapshot is incomplete.

It is also written for coding agents first:
- chapter layouts are repetitive on purpose
- trust boundaries and failure modes are explicit
- source-shaped examples have been removed
- appendices index commands, tools, parsers, flags, and key types

Important boundary:
- this guide is intended to preserve architectural understanding only
- it is not intended to be code-equivalent or implementation-complete
- where a chapter previously used source-like pseudocode, that material has been replaced with prose

## Reading Order

1. [00-checklist.md](./00-checklist.md)
2. [01-repo-map-and-snapshot-caveats.md](./01-repo-map-and-snapshot-caveats.md)
3. [02-startup-bootstrap-and-entrypoints.md](./02-startup-bootstrap-and-entrypoints.md)
4. [03-query-engine-and-turn-lifecycle.md](./03-query-engine-and-turn-lifecycle.md)
5. [04-message-model-sdk-and-schemas.md](./04-message-model-sdk-and-schemas.md)
6. [05-tools-system-and-permissions.md](./05-tools-system-and-permissions.md)
7. [06-bash-parser-validation-and-security.md](./06-bash-parser-validation-and-security.md)
8. [07-commands-skills-and-plugins.md](./07-commands-skills-and-plugins.md)
9. [08-mcp-and-external-integrations.md](./08-mcp-and-external-integrations.md)
10. [09-bridge-remote-daemon-and-session-plumbing.md](./09-bridge-remote-daemon-and-session-plumbing.md)
11. [10-state-react-ink-and-terminal-runtime.md](./10-state-react-ink-and-terminal-runtime.md)
12. [11-memory-compaction-and-session-storage.md](./11-memory-compaction-and-session-storage.md)
13. [12-agents-tasks-worktrees-and-swarms.md](./12-agents-tasks-worktrees-and-swarms.md)
14. [13-auth-config-policy-and-analytics.md](./13-auth-config-policy-and-analytics.md)
15. [14-surprising-design-choices-and-startup-lessons.md](./14-surprising-design-choices-and-startup-lessons.md)

Appendices:
- [15-file-inventory.md](./appendices/15-file-inventory.md)
- [16-command-index.md](./appendices/16-command-index.md)
- [17-tool-index.md](./appendices/17-tool-index.md)
- [18-type-class-and-schema-index.md](./appendices/18-type-class-and-schema-index.md)
- [19-parser-index.md](./appendices/19-parser-index.md)
- [20-feature-flag-index.md](./appendices/20-feature-flag-index.md)
- [21-third-party-audit-checklist.md](./appendices/21-third-party-audit-checklist.md)

## How To Use This Guide

If you are building a similar product:
- read chapters 02, 03, 05, 06, 08, 09, and 11 first
- treat chapter 14 as a list of architecture traps to avoid
- use the appendices as a lookup layer once you know the big picture

If you are building agent tooling against this snapshot:
- start with the query loop, tool system, and session storage
- then read MCP, bridge, and agent/task orchestration
- use the file inventory to jump directly into specific areas

## Snapshot Caveats

This repo is not a complete standalone build:
- there is no `package.json`
- there is no `tsconfig.json`
- there is no lockfile
- there are no visible test files
- several imports point at files that are not present in the snapshot

Example:
- many modules imported `src/types/message.js`, but the underlying message type source file was not present under `src/types` even in the analyzed snapshot

That does not make the snapshot useless. It means we should treat it like an extracted internal monorepo slice and keep the boundary between `Observed` and `[Inference]` clean.
