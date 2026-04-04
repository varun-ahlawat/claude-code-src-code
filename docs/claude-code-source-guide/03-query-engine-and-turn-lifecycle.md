# 03 Query Engine And Turn Lifecycle

## Purpose

Document the real center of gravity in the product: how a user turn becomes model calls, tool calls, recovery loops, transcript updates, and final output.

## Key Files And Symbols

- [`src/QueryEngine.ts`](../../src/QueryEngine.ts)
- [`src/query.ts`](../../src/query.ts)
- [`src/services/api/claude.ts`](../../src/services/api/claude.ts)
- [`src/services/tools/StreamingToolExecutor.ts`](../../src/services/tools/StreamingToolExecutor.ts)
- [`src/services/tools/toolExecution.ts`](../../src/services/tools/toolExecution.ts)

Important symbols:
- `QueryEngine`
- `submitMessage()`
- `query()`
- `queryLoop()`
- `StreamingToolExecutor`

## Core Types / Classes

Observed query configuration shape from [`src/QueryEngine.ts`](../../src/QueryEngine.ts):


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Mutable loop state from `query.ts` conceptually looks like:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Data Flow

Observed turn lifecycle:

1. `QueryEngine.submitMessage()` resets turn-scoped bookkeeping.
2. It establishes cwd and tool-permission wrappers.
3. It constructs prompt context: commands, tools, MCP, memory, system prompt, settings-derived behavior.
4. It yields into `query()`.
5. `query()` delegates to `queryLoop()`.
6. `queryLoop()` normalizes messages for the API, manages streaming, and reacts to tool-use blocks.
7. Tools run either eagerly while streaming or through normal orchestration.
8. Recovery branches may fire:
   - prompt-too-long
   - reactive compact
   - max-output-token escalation
   - token-budget continuation
   - fallback from streaming to non-streaming
9. Results and transcript state are persisted.

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- Message normalization is a trust boundary. UI-only state must not leak into API-bound payloads.
- Tool-result pairing is a trust boundary. Synthetic placeholders exist to preserve structural integrity when internal errors occur.
- Recovery loops can easily become infinite if guards are wrong. The source contains explicit comments about prior loop bugs and API-burn risks.
- Streaming and non-streaming paths must remain behaviorally compatible enough to share transcript semantics.

## Non-Obvious Implementation Choices

### 1. The query loop is a state machine, not a single request wrapper

This code is not `send prompt -> await answer`. It handles:
- streaming tool-use interleaving
- structured output enforcement
- multiple retry classes
- compact/replay behavior
- budget continuation
- stop hooks

### 2. Recovery logic is deliberately branchy

The branches exist because each failure mode threatens a different invariant:
- prompt too long threatens context admission
- max output tokens threatens completion progress
- missing tool results threaten structural validity
- stop-hook injection can threaten retry convergence

### 3. Tool execution is partly online

`StreamingToolExecutor` starts tools as the assistant streams them, but still preserves output order. That is a careful compromise between latency and transcript determinism.

### 4. Turn-scoped caches matter

`QueryEngine` clears discovered skill names and other turn-scoped tracking every turn. That reduces unbounded growth in long-lived headless sessions.

## Agent-Builder Takeaways

- Model loops need explicit continuation/recovery state, not just retries.
- Preserve transcript invariants even when errors happen.
- If you stream tool-use, separate execution concurrency from result ordering.
- Compaction is part of the normal runtime, not a maintenance task.

## Source Anchors

- [`src/QueryEngine.ts`](../../src/QueryEngine.ts)
- [`src/query.ts`](../../src/query.ts)
- [`src/services/api/claude.ts`](../../src/services/api/claude.ts)
- [`src/services/tools/StreamingToolExecutor.ts`](../../src/services/tools/StreamingToolExecutor.ts)
- [`src/services/tools/toolExecution.ts`](../../src/services/tools/toolExecution.ts)

## [Inference]

The query subsystem appears to be the product's hardest-won operational layer. The density of comments about retries, pairing, compaction, and error recovery suggests years of production incidents shaped this design.
