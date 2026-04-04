# 14 Surprising Design Choices And Startup Lessons

## Purpose

Synthesize the parts of this codebase that are easy to underestimate and expensive to retrofit.

## Key Files And Symbols

This chapter is synthetic, but it is grounded in recurring patterns from:

- [`src/entrypoints/cli.tsx`](../../src/entrypoints/cli.tsx)
- [`src/main.tsx`](../../src/main.tsx)
- [`src/tools.ts`](../../src/tools.ts)
- [`src/utils/bash/ast.ts`](../../src/utils/bash/ast.ts)
- [`src/query.ts`](../../src/query.ts)
- [`src/utils/sessionStorage.ts`](../../src/utils/sessionStorage.ts)
- [`src/memdir/memoryTypes.ts`](../../src/memdir/memoryTypes.ts)
- [`src/bridge/bridgeMain.ts`](../../src/bridge/bridgeMain.ts)
- [`src/state/store.ts`](../../src/state/store.ts)

Important recurring symbols:
- `feature()`
- `PARSE_ABORTED`
- `isTranscriptMessage()`
- `QueryEngine`
- `StreamingToolExecutor`

## Core Types / Classes

This chapter is about design patterns more than one concrete type, but the repeating shape is:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Data Flow

```text
read subsystem chapters
-> identify repeated defensive patterns
-> collapse them into startup lessons
-> turn lessons into architecture heuristics for future agent products
```

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- Build-time product slicing can quietly leak code if treated as ordinary runtime config.
- Shell safety fails dangerously if uncertainty is collapsed into optimistic parsing.
- Session persistence fails subtly if transcript and progress semantics are mixed.
- Remote control becomes unmaintainable if it forks message and permission contracts.
- Memory becomes noise if it lacks a narrow ontology and size discipline.

## Non-Obvious Implementation Choices

### 1. Compile-time feature gating is architecture, not polish

Observed:
- `feature()` appears everywhere
- comments repeatedly warn to keep it inline
- tools, commands, bridge flows, UI hooks, and transports are sliced by it

Startup lesson:
- if you expect multiple editions, internal-only features, or staged rollouts, decide early whether product slicing is compile-time or runtime
- mixing both ad hoc creates a brittle codebase

Why startups waste time here:
- they start with runtime flags only
- then discover bundle size, dependency leakage, and internal-only code exposure problems
- then retrofit ugly build hacks later

### 2. Lazy imports are doing more than performance work

Observed:
- they are used to preserve DCE
- they break circular dependencies
- they prevent heavy UI/native modules from loading on fast paths

Startup lesson:
- lazy imports are often a structural tool in complex CLI/agent apps

Counterintuitive part:
- `import later` is not always technical debt; here it is often load-bearing

### 3. The shell-security layer fails closed on purpose

Observed:
- `ast.ts` explicitly says unknown structure becomes `too-complex`
- parser abort is distinct from parser absence
- comments repeatedly call out specific bypass classes

Startup lesson:
- the correct shell-security UX is often `ask more often`, not `pretend confidence`

Why startups waste time here:
- they optimize early for convenience
- they keep widening permissive heuristics
- they only discover the real complexity after the first dangerous bypass

### 4. Query recovery logic becomes a product subsystem

Observed:
- streaming fallback
- reactive compact
- max-output-token recovery
- token-budget continuation
- stop-hook interaction
- tool-result pairing repair

Startup lesson:
- once your model can call tools, recover mid-turn, and persist sessions, query execution stops being a request wrapper and becomes a state machine

### 5. Session storage is not "chat history"

Observed:
- transcript filtering
- parent UUID chain handling
- subagent transcript paths
- resume and rewind integration
- metadata and sidechain handling

Startup lesson:
- if you want resume, branch, rewind, side agents, or remote viewers, you need a conversation log model, not just append-only text

### 6. Memory needs a narrow ontology

Observed:
- typed memory taxonomy
- explicit "what not to save"
- line and byte caps for entrypoints

Startup lesson:
- memory quality depends more on exclusion rules than on storage volume

### 7. Worktrees matter earlier than most teams expect

Observed:
- setup path handles worktree creation
- bridge mode handles worktree spawning
- agents can request worktree isolation

Startup lesson:
- if your agent edits repositories seriously, file sandboxing alone will not cover branch isolation, PR workflows, or concurrent agent work

### 8. A custom store can beat heavyweight state libraries

Observed:
- tiny `createStore()`
- `useSyncExternalStore`
- broad but explicit `AppState`

Startup lesson:
- if the real problem is cross-runtime orchestration rather than reducer complexity, a light store plus good boundaries may outperform Redux-scale ceremony

### 9. Remote mode should adapt shared primitives, not fork them

Observed:
- remote permission bridge synthesizes assistant/tool context
- SDK message adapter maps remote messages into local message semantics

Startup lesson:
- reuse your core message and permission abstractions across process boundaries
- do not build a second incompatible runtime for remote control

### 10. Transformed source artifacts can mislead reverse engineering

Observed:
- many `.tsx` files include `react/compiler-runtime`
- many files include embedded source maps

Startup lesson:
- document when readers are seeing compiler output instead of author source
- otherwise teams cargo-cult the artifact instead of the design

## Agent-Builder Takeaways

- Decide your compile-time vs runtime feature strategy.
- Make transcript persistence selective and chain-aware.
- Separate tool visibility, permission, and execution.
- Build a fail-closed shell classifier before you ship broad shell autonomy.
- Plan for compaction and recovery as part of normal execution.
- Version and sanitize plugin caches.
- Build remote/hosted modes by adapting shared message/tool abstractions.
- Add worktree-aware isolation before multi-agent repo editing becomes common.

## Source Anchors

- [`src/entrypoints/cli.tsx`](../../src/entrypoints/cli.tsx)
- [`src/tools.ts`](../../src/tools.ts)
- [`src/utils/bash/ast.ts`](../../src/utils/bash/ast.ts)
- [`src/query.ts`](../../src/query.ts)
- [`src/utils/sessionStorage.ts`](../../src/utils/sessionStorage.ts)
- [`src/memdir/memoryTypes.ts`](../../src/memdir/memoryTypes.ts)
- [`src/bridge/bridgeMain.ts`](../../src/bridge/bridgeMain.ts)
- [`src/state/store.ts`](../../src/state/store.ts)

## [Inference]

The most surprising thing about this codebase is not any one feature. It is how many "later" problems were pulled into first-class architecture: startup latency, shell ambiguity, transcript integrity, remote control, memory quality, and multi-agent isolation. That is exactly the class of work many startups postpone until the cost of retrofitting it is much higher.
