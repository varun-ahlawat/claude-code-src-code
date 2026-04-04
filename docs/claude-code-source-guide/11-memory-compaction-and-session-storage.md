# 11 Memory Compaction And Session Storage

## Purpose

Explain the persistence and context-management layer: memory directories, transcript storage, compaction, and recovery.

## Key Files And Symbols

- [`src/memdir/memdir.ts`](../../src/memdir/memdir.ts)
- [`src/memdir/memoryTypes.ts`](../../src/memdir/memoryTypes.ts)
- [`src/memdir/findRelevantMemories.ts`](../../src/memdir/findRelevantMemories.ts)
- [`src/services/compact/compact.ts`](../../src/services/compact/compact.ts)
- [`src/services/compact/microCompact.ts`](../../src/services/compact/microCompact.ts)
- [`src/utils/sessionStorage.ts`](../../src/utils/sessionStorage.ts)
- [`src/services/teamMemorySync/index.ts`](../../src/services/teamMemorySync/index.ts)
- [`src/services/teamMemorySync/secretScanner.ts`](../../src/services/teamMemorySync/secretScanner.ts)
- [`src/services/teamMemorySync/watcher.ts`](../../src/services/teamMemorySync/watcher.ts)

Important symbols:
- `truncateEntrypointContent()`
- `buildMemoryLines()`
- `isTranscriptMessage()`
- `stripImagesFromMessages()`
- `truncateHeadForPTLRetry()`

## Core Types / Classes

Observed memory structure:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Observed session-storage concept:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Observed compaction role:
- summarize old context
- strip irrelevant high-cost payloads like images before compaction
- preserve post-compact affordances like file restore hints, skill re-injection, and compact boundary markers

## Data Flow

Memory:

```text
memory dir exists
-> load entrypoint and memory typing instructions
-> optionally retrieve relevant memories
-> inject memory context into prompt
-> later extract and save new memories
```

Session storage:

```text
runtime messages
-> transcript filter
-> JSONL append
-> resume / rewind / load chain reconstruction
```

Compaction:

```text
token pressure / explicit compact / retry conditions
-> summarize or trim old conversation
-> emit compact boundary messages
-> keep transcript continuity and reload critical deltas
```

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- Memory is intentionally typed. Not everything worth remembering belongs in memory.
- The memory instructions explicitly exclude facts derivable from the current repo state. That is a very important anti-bloat rule.
- Transcript persistence is a source-of-truth boundary. Persisting ephemeral progress breaks session chains.
- Compaction itself can hit prompt-too-long failures, so the compaction subsystem contains its own retry logic.

## Non-Obvious Implementation Choices

### 1. Memory is not free-form notes

The product encodes a closed taxonomy with behavioral guidance. That keeps memory semantically useful instead of becoming a dump of arbitrary reminders.

### 2. Memory entrypoints have byte and line caps

This is subtle and valuable. The source explicitly defends against giant index files and long-line blowups.

### 3. Compaction is part of the online request loop

It is not a maintenance operation or post-processing step. The query loop expects compaction and recovery to be routine.

### 4. Session storage has chain semantics, not just append-only logs

The code cares about:
- parent UUID linkage
- sidechains
- subagent transcript paths
- resume visibility
- metadata re-append behavior

This is much more than `save chat history`.

## Team Memory Sync

The guide previously treated memory as local-only, but the repo contains a real shared-memory synchronization subsystem.

Observed behavior across [`src/services/teamMemorySync/index.ts`](../../src/services/teamMemorySync/index.ts), [`src/services/teamMemorySync/secretScanner.ts`](../../src/services/teamMemorySync/secretScanner.ts), and [`src/services/teamMemorySync/watcher.ts`](../../src/services/teamMemorySync/watcher.ts):
- sync identity is derived from repository remote metadata, so shared memory is repo-scoped rather than workspace-global
- the sync engine maintains explicit state across calls, batches uploads/downloads, and retries on conflict instead of assuming single-shot success
- pull is effectively server-wins while delete propagation is intentionally conservative, which reduces destructive drift
- uploads are secret-scanned before they leave disk
- the watcher is debounced and suppression-aware so local writes caused by sync itself do not recursively trigger infinite resync loops
- startup wires this watcher in once cwd and trust state are stable, which makes it part of the long-lived session model rather than a utility command

This subsystem materially changes how “memory” behaves in collaborative repos and deserves to be documented alongside compaction and transcript storage.

## Agent-Builder Takeaways

- Give memory a type system and exclusion rules.
- Put hard caps on user-editable context entrypoints.
- Treat transcript persistence as relational state, not just logging.
- Expect your compaction path to fail and need retries too.

## Source Anchors

- [`src/memdir/memdir.ts`](../../src/memdir/memdir.ts)
- [`src/memdir/memoryTypes.ts`](../../src/memdir/memoryTypes.ts)
- [`src/memdir/findRelevantMemories.ts`](../../src/memdir/findRelevantMemories.ts)
- [`src/services/compact/compact.ts`](../../src/services/compact/compact.ts)
- [`src/utils/sessionStorage.ts`](../../src/utils/sessionStorage.ts)
- [`src/services/teamMemorySync/index.ts`](../../src/services/teamMemorySync/index.ts)
- [`src/services/teamMemorySync/secretScanner.ts`](../../src/services/teamMemorySync/secretScanner.ts)
- [`src/services/teamMemorySync/watcher.ts`](../../src/services/teamMemorySync/watcher.ts)

## [Inference]

The amount of infrastructure around compaction and transcript recovery suggests long-running sessions are a core product case, not an edge case.
