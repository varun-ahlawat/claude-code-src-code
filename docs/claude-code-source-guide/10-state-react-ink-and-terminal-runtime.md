# 10 State React Ink And Terminal Runtime

## Purpose

Explain how the UI state model works, why it does not use Redux, and how terminal rendering/parsing is integrated into the app runtime.

## Key Files And Symbols

- [`src/state/store.ts`](../../src/state/store.ts)
- [`src/state/AppStateStore.ts`](../../src/state/AppStateStore.ts)
- [`src/state/AppState.tsx`](../../src/state/AppState.tsx)
- [`src/screens/REPL.tsx`](../../src/screens/REPL.tsx)
- [`src/cli/print.ts`](../../src/cli/print.ts)
- [`src/components/PromptInput/PromptInput.tsx`](../../src/components/PromptInput/PromptInput.tsx)
- [`src/hooks/useTypeahead.tsx`](../../src/hooks/useTypeahead.tsx)
- [`src/components/Messages.tsx`](../../src/components/Messages.tsx)
- [`src/components/VirtualMessageList.tsx`](../../src/components/VirtualMessageList.tsx)
- [`src/components/Message.tsx`](../../src/components/Message.tsx)
- [`src/components/messages/SystemTextMessage.tsx`](../../src/components/messages/SystemTextMessage.tsx)
- [`src/components/MessageSelector.tsx`](../../src/components/MessageSelector.tsx)
- [`src/components/LogSelector.tsx`](../../src/components/LogSelector.tsx)
- [`src/components/permissions/PermissionRequest.tsx`](../../src/components/permissions/PermissionRequest.tsx)
- [`src/ink/termio/parser.ts`](../../src/ink/termio/parser.ts)
- [`src/keybindings/parser.ts`](../../src/keybindings/parser.ts)

Important symbols:
- `createStore()`
- `AppState`
- `useAppState()`
- `useSyncExternalStore`
- terminal `Action` parsing

## Core Types / Classes

Observed store contract from [`src/state/store.ts`](../../src/state/store.ts):


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Observed app-state scope from [`src/state/AppStateStore.ts`](../../src/state/AppStateStore.ts):
- settings
- model and effort state
- tool permission context
- tasks and agent registries
- MCP clients, tools, commands, resources
- plugin state
- notifications and elicitation queues
- thinking state
- bridge and remote session state
- companion / buddy state
- browser and terminal panel state
- prompt input, queued command, and prompt-overlay state
- permission-request queues and tool-confirm UI state

## Data Flow

Interactive path:

```text
AppStateStore
-> AppStateProvider
-> REPL screen + component tree
-> tools / hooks / bridge events mutate store
-> selective subscriptions rerender specific UI slices
```

Input path:

```text
raw keypresses
-> keybindings + input mode detection
-> PromptInput local state
-> useTypeahead / history / slash-command / attachment helpers
-> queued command or user message submission
```

Transcript path:

```text
normalized renderable messages
-> Messages.tsx grouping + lookup building
-> VirtualMessageList virtualization + search indexing
-> MessageRow / message renderers
-> scroll handlers + selection + permission dialogs
```

Headless path:

```text
query + structured IO
-> cli/print.ts orchestration
-> output transport / session state / hook callbacks
```

Terminal runtime path:

```text
raw terminal bytes
-> tokenizer
-> ANSI semantic parser
-> cursor/style/mode actions
-> Ink-compatible rendering state
```

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- `useAppState()` relies on selector stability. Returning fresh objects defeats render optimization.
- The REPL has both local React state and store-backed state; bugs can appear if one is treated as fresher than the other.
- Terminal parsing is incremental and semantic. Incorrect ANSI handling can corrupt layout or cursor behavior in a terminal UI, which is more fragile than browser rendering.
- Prompt input is a mixed local/store system too. Typeahead, prompt overlays, queued slash commands, pasted attachments, and submission helpers all touch different state layers.
- Permission requests are UI state with behavioral consequences. If the assistant message, tool input, or permission context goes stale, the approval UI becomes misleading instead of merely ugly.

## Non-Obvious Implementation Choices

### 1. Custom store plus `useSyncExternalStore` is enough

This codebase does not need Redux because:
- state is local to one product runtime
- updates are explicit function updaters
- React only needs subscription semantics
- non-React code can still call `getState()` / `setState()`

### 2. `cli/print.ts` is as important as `REPL.tsx`

The interactive and headless paths are peers. The headless side is not an afterthought; it is a large orchestration layer for structured output, remote IO, hooks, and session state.

### 3. The terminal parser is semantic, not just lexical

[`src/ink/termio/parser.ts`](../../src/ink/termio/parser.ts) does not simply tokenize escapes. It emits actions like:
- cursor movement
- erase commands
- mode toggles
- style updates

### 4. The snapshot includes compiled React output

Many `.tsx` files start with `react/compiler-runtime` imports and embedded source maps. That means:
- the artifact is still readable
- but some code shape reflects compiler transforms rather than original handwritten structure

### 5. `PromptInput` is a control plane, not a text box

[`src/components/PromptInput/PromptInput.tsx`](../../src/components/PromptInput/PromptInput.tsx) pulls together slash commands, history search, teammate routing, prompt suggestions, image paste, IDE mentions, permission mode toggles, and model/effort controls. For agent products, this is where a large share of “how the system feels” actually lives.

### 6. Transcript rendering is explicitly performance-engineered

[`src/components/Messages.tsx`](../../src/components/Messages.tsx), [`src/components/VirtualMessageList.tsx`](../../src/components/VirtualMessageList.tsx), and [`src/components/MessageRow.tsx`](../../src/components/MessageRow.tsx) show careful work around grouping, collapse state, search indexing, scroll anchoring, and avoiding pathological rerenders in long transcripts.

### 7. Permission UIs are specialized by tool

[`src/components/permissions/PermissionRequest.tsx`](../../src/components/permissions/PermissionRequest.tsx) dispatches to tool-specific components such as Bash, PowerShell, AskUserQuestion, EnterPlanMode, and ExitPlanMode request views. That means the human approval surface is not one generic modal but a family of domain-specific workflows.

## Transcript Taxonomy And Operator Restore Workflows

The transcript layer is richer than `Messages.tsx` plus virtualization.

Observed behavior across [`src/components/Message.tsx`](../../src/components/Message.tsx), [`src/components/messages/SystemTextMessage.tsx`](../../src/components/messages/SystemTextMessage.tsx), [`src/components/MessageSelector.tsx`](../../src/components/MessageSelector.tsx), and [`src/components/LogSelector.tsx`](../../src/components/LogSelector.tsx):
- `Message.tsx` is the render dispatcher for many distinct message kinds, including assistant output, tool traffic, system events, and operational messages that do not fit a simple `chat bubble` model
- `SystemTextMessage.tsx` contains a taxonomy of operational subtypes, which is where restore/summarize/compact/system notices become legible to the operator
- `MessageSelector.tsx` is not just “pick a message”; it supports restoring conversation, restoring code, doing both together, and summarizing either from or up to a selected point, with diff-aware feedback about what code would actually change
- `LogSelector.tsx` is a session browser with grouped histories, sidechain/session filtering, preview state, search, and rename flows, which makes transcript history an inspectable workspace object instead of a hidden log directory

This is the operator workflow layer for transcript recovery and summarization, and it belongs in the architecture story.

## Agent-Builder Takeaways

- A lightweight store with explicit selectors can be enough for complex agent UIs.
- Treat headless and interactive runtimes as equal design targets.
- If you build a terminal app, invest in semantic terminal parsing instead of assuming text-only output.
- Document transformed-source artifacts so future readers do not misattribute compiler output to coding style.
- Treat prompt input, transcript virtualization, and approval dialogs as core runtime architecture, not UI polish.

## Source Anchors

- [`src/state/store.ts`](../../src/state/store.ts)
- [`src/state/AppStateStore.ts`](../../src/state/AppStateStore.ts)
- [`src/state/AppState.tsx`](../../src/state/AppState.tsx)
- [`src/screens/REPL.tsx`](../../src/screens/REPL.tsx)
- [`src/cli/print.ts`](../../src/cli/print.ts)
- [`src/components/PromptInput/PromptInput.tsx`](../../src/components/PromptInput/PromptInput.tsx)
- [`src/hooks/useTypeahead.tsx`](../../src/hooks/useTypeahead.tsx)
- [`src/components/Messages.tsx`](../../src/components/Messages.tsx)
- [`src/components/VirtualMessageList.tsx`](../../src/components/VirtualMessageList.tsx)
- [`src/components/Message.tsx`](../../src/components/Message.tsx)
- [`src/components/messages/SystemTextMessage.tsx`](../../src/components/messages/SystemTextMessage.tsx)
- [`src/components/MessageSelector.tsx`](../../src/components/MessageSelector.tsx)
- [`src/components/LogSelector.tsx`](../../src/components/LogSelector.tsx)
- [`src/components/permissions/PermissionRequest.tsx`](../../src/components/permissions/PermissionRequest.tsx)
- [`src/ink/termio/parser.ts`](../../src/ink/termio/parser.ts)
- [`src/keybindings/parser.ts`](../../src/keybindings/parser.ts)

## [Inference]

The combination of a custom store, custom Ink fork/runtime, and terminal parser stack suggests the team hit scaling limits with stock terminal UI assumptions and progressively internalized more of the terminal runtime.
