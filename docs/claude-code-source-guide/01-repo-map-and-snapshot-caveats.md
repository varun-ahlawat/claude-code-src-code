# 01 Repo Map And Snapshot Caveats

## Purpose

This chapter establishes what this repo actually is, what is missing, and how to read it safely.

Important framing:
- the analyzed snapshot originally included a `src/` tree
- that source tree has since been removed from this repository
- this chapter therefore describes the analyzed snapshot, not the current checked-in file tree

## Observed

The analyzed snapshot root was minimal:
- `.git`
- `README.md`
- `src/`

There is no visible build metadata:
- no `package.json`
- no `tsconfig.json`
- no lockfile
- no visible test files

`src/` is the real product surface. A quick count shows about `1902` files under `src/`.

Top-level source density:

| Area | Approx files |
| --- | ---: |
| `src/utils` | 564 |
| `src/components` | 389 |
| `src/commands` | 207 |
| `src/tools` | 184 |
| `src/services` | 130 |
| `src/hooks` | 104 |
| `src/ink` | 96 |
| `src/bridge` | 31 |
| `src/cli` | 19 |
| `src/skills` | 20 |

This is enough to infer the product shape:
- terminal-first app
- React + Ink UI
- strong agent/tool runtime layer
- serious shell-security work
- first-party MCP client implementation
- remote-control / bridge mode
- persistent transcripts, memory, and compaction infrastructure

Small top-level roots also matter because they reveal seams that would be easy to miss in a folder-count-only pass:
- [`src/coordinator/coordinatorMode.ts`](../../src/coordinator/coordinatorMode.ts): alternate orchestrator role and worker-capability prompt
- [`src/schemas/hooks.ts`](../../src/schemas/hooks.ts): cycle-broken persisted hook DSL
- [`src/outputStyles/loadOutputStylesDir.ts`](../../src/outputStyles/loadOutputStylesDir.ts): markdown-defined output-style loading
- [`src/assistant/sessionHistory.ts`](../../src/assistant/sessionHistory.ts): assistant-mode remote history paging
- [`src/voice/voiceModeEnabled.ts`](../../src/voice/voiceModeEnabled.ts): feature-gated voice visibility + auth checks
- [`src/moreright/useMoreRight.tsx`](../../src/moreright/useMoreRight.tsx): explicit external-build stub for an internal-only hook

## Key Files And Symbols

High-gravity files by line count:
- [`src/cli/print.ts`](../../src/cli/print.ts)
- [`src/utils/messages.ts`](../../src/utils/messages.ts)
- [`src/utils/sessionStorage.ts`](../../src/utils/sessionStorage.ts)
- [`src/utils/hooks.ts`](../../src/utils/hooks.ts)
- [`src/screens/REPL.tsx`](../../src/screens/REPL.tsx)
- [`src/main.tsx`](../../src/main.tsx)
- [`src/utils/bash/bashParser.ts`](../../src/utils/bash/bashParser.ts)
- [`src/services/api/claude.ts`](../../src/services/api/claude.ts)
- [`src/services/mcp/client.ts`](../../src/services/mcp/client.ts)
- [`src/bridge/bridgeMain.ts`](../../src/bridge/bridgeMain.ts)

These files are where the architecture stops being cosmetic and becomes operational.

## Core Types / Classes

Even before diving into subsystems, the repo advertises several central abstractions:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Data Flow

At a high level:

```text
startup entrypoint
-> settings + trust + policy + auth bootstrap
-> build command/tool/skill/plugin/MCP surface
-> accept user input
-> run query loop
-> emit tool calls, UI updates, transcript writes, hooks, and session state
-> optionally bridge/remote/daemon integration
```

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- The snapshot is incomplete. Some imports reference files not present here.
- Missing build config means any runtime conclusion must be drawn from source topology and comments, not from successful local execution.
- Several `.tsx` files are compiled/transformed artifacts rather than pristine author source.
- Some files are deliberate external-build stubs. [`src/moreright/useMoreRight.tsx`](../../src/moreright/useMoreRight.tsx) explicitly says the real hook is internal only, which is strong evidence that this snapshot sits behind an overlay/build-selection system we cannot see here.

One concrete caveat:
- many modules import `src/types/message.js`, but no corresponding message-type source file exists under [`src/types`](../../src/types)

That means the guide's message chapter must combine direct evidence from:
- imports
- helper code in [`src/utils/messages.ts`](../../src/utils/messages.ts)
- transcript logic in [`src/utils/sessionStorage.ts`](../../src/utils/sessionStorage.ts)
- SDK schemas in [`src/entrypoints/sdk/coreSchemas.ts`](../../src/entrypoints/sdk/coreSchemas.ts)

## Non-Obvious Implementation Choices

- The repo is not structured like a toy CLI. It looks like a product platform with multiple execution modes sharing one core conversation runtime.
- `src/utils` is much larger than `src/services`, which is a clue that behavior is encoded in local orchestration helpers rather than purely vertical service folders.
- `src/ink` is substantial. This is not `commander + console.log`; it is a terminal application framework.
- The source snapshot appears to include some transformed UI artifacts. Reverse-engineering without acknowledging that will distort conclusions about coding style.
- Several one-file top-level roots carry strategic behavior. Coordinator mode, hook schemas, output styles, voice gating, and assistant-mode history are not large folders, but they are product-shaping surfaces.

## Agent-Builder Takeaways

- Read this repo as a runtime architecture, not just a CLI app.
- Prioritize execution seams over folder names: startup, query, tools, persistence, remote control.
- Assume missing files exist behind some imported `.js` surfaces and treat those gaps explicitly.

## Source Anchors

- [`README.md`](../../README.md)
- [`src/main.tsx`](../../src/main.tsx)
- [`src/entrypoints/cli.tsx`](../../src/entrypoints/cli.tsx)
- [`src/cli/print.ts`](../../src/cli/print.ts)
- [`src/screens/REPL.tsx`](../../src/screens/REPL.tsx)
- [`src/utils/sessionStorage.ts`](../../src/utils/sessionStorage.ts)

## [Inference]

The repo was almost certainly extracted from a larger internal monorepo with:
- generated SDK type outputs
- private packages or build macros
- multiple product editions hidden behind compile-time flags
- test and CI infrastructure stripped out of the snapshot

The evidence is strong, but the internal build pipeline itself is not present, so this remains inference rather than direct observation.
