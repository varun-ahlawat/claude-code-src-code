# 04 Message Model SDK And Schemas

## Purpose

Map the product's message universe: internal conversation messages, transcript persistence, SDK-facing serializable types, and schema-generation strategy.

## Key Files And Symbols

- [`src/utils/messages.ts`](../../src/utils/messages.ts)
- [`src/utils/sessionStorage.ts`](../../src/utils/sessionStorage.ts)
- [`src/entrypoints/agentSdkTypes.ts`](../../src/entrypoints/agentSdkTypes.ts)
- [`src/entrypoints/sdk/coreTypes.ts`](../../src/entrypoints/sdk/coreTypes.ts)
- [`src/entrypoints/sdk/coreSchemas.ts`](../../src/entrypoints/sdk/coreSchemas.ts)
- [`src/entrypoints/sdk/controlSchemas.ts`](../../src/entrypoints/sdk/controlSchemas.ts)

Important symbols:
- `SDKMessage`
- `NormalizedMessage`
- `isTranscriptMessage()`
- `isChainParticipant()`
- `HOOK_EVENTS`

## Core Types / Classes

Observed SDK layering:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Observed schema philosophy in [`src/entrypoints/sdk/coreSchemas.ts`](../../src/entrypoints/sdk/coreSchemas.ts):
- Zod schemas are the source of truth
- generated types are committed for IDE support
- public types are split from internal runtime-only helpers

Useful conceptual message sketch:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Data Flow

Internal lifecycle:

```text
UI / slash command / bridge / remote input
-> internal Message objects
-> normalization for API
-> transcript persistence as JSONL entries
-> SDK/structured output adaptation for external consumers
```

Observed persistence rule from [`src/utils/sessionStorage.ts`](../../src/utils/sessionStorage.ts):
- transcript messages are the source of truth
- progress messages are explicitly excluded from transcript chains

Observed SDK rule from [`src/entrypoints/sdk/coreTypes.ts`](../../src/entrypoints/sdk/coreTypes.ts):
- generated serializable types are preferred for public stability
- hook events and exit reasons are exported as runtime constants

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- Missing message-type source file in this snapshot means some message shapes are only indirectly recoverable.
- Transcript inclusion rules are critical. Persisting progress messages forks parent chains and breaks resume logic.
- SDK control messages are structurally separate from ordinary assistant/user messages.
- Thinking/tool-use blocks have strict structural rules, and message helpers explicitly protect them.

## Non-Obvious Implementation Choices

### 1. Schemas generate types, not the other way around

That is a runtime-contract decision. The team wants validation-first public types.

### 2. Internal messages are richer than SDK messages

The UI and runtime need:
- progress
- attachments
- compact boundary markers
- telemetry and denial detail

The SDK needs stable serialized surfaces. Adapters bridge the two.

### 3. Transcript persistence is selective

The source is explicit: not every runtime message deserves to be persisted. This is a subtle but crucial distinction.

### 4. The snapshot is partially missing type source

The missing `src/types/message.ts` file is itself an architectural clue: some internal type surfaces were stripped from the leaked snapshot, but the surrounding utilities still reveal the operational model.

## Agent-Builder Takeaways

- Make transcript persistence rules explicit and testable.
- Use a single validation source of truth for public SDK surfaces.
- Keep UI/event richness separate from stable wire shapes.
- Preserve parent-chain semantics if you ever want reliable resume, rewind, or session forking.

## Source Anchors

- [`src/utils/messages.ts`](../../src/utils/messages.ts)
- [`src/utils/sessionStorage.ts`](../../src/utils/sessionStorage.ts)
- [`src/entrypoints/agentSdkTypes.ts`](../../src/entrypoints/agentSdkTypes.ts)
- [`src/entrypoints/sdk/coreTypes.ts`](../../src/entrypoints/sdk/coreTypes.ts)
- [`src/entrypoints/sdk/coreSchemas.ts`](../../src/entrypoints/sdk/coreSchemas.ts)

## [Inference]

The combination of generated SDK types plus explicit transcript-chain handling suggests the team treats the CLI, desktop, remote viewers, and SDK consumers as different frontends over one shared conversation log model.
