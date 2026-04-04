# 09 Bridge Remote Daemon And Session Plumbing

## Purpose

Explain the remote-control side of the product: bridge mode, remote session management, websocket transport, heartbeat loops, and session compatibility shims.

## Key Files And Symbols

- [`src/bridge/bridgeMain.ts`](../../src/bridge/bridgeMain.ts)
- [`src/bridge/replBridge.ts`](../../src/bridge/replBridge.ts)
- [`src/bridge/bridgeMessaging.ts`](../../src/bridge/bridgeMessaging.ts)
- [`src/bridge/sessionIdCompat.ts`](../../src/bridge/sessionIdCompat.ts)
- [`src/bridge/jwtUtils.ts`](../../src/bridge/jwtUtils.ts)
- [`src/hooks/useReplBridge.tsx`](../../src/hooks/useReplBridge.tsx)
- [`src/assistant/sessionHistory.ts`](../../src/assistant/sessionHistory.ts)
- [`src/remote/RemoteSessionManager.ts`](../../src/remote/RemoteSessionManager.ts)
- [`src/remote/SessionsWebSocket.ts`](../../src/remote/SessionsWebSocket.ts)
- [`src/remote/sdkMessageAdapter.ts`](../../src/remote/sdkMessageAdapter.ts)

Important symbols:
- `runBridgeLoop()`
- `ReplBridgeHandle`
- `createSessionSpawner()`
- `RemoteSessionManager`
- `SessionsWebSocket`
- `toCompatSessionId()`
- `toInfraSessionId()`
- `fetchLatestEvents()`

## Core Types / Classes

Observed bridge runtime concepts:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Bridge responsibilities visible in source:
- poll for work
- spawn local session processes
- maintain heartbeats
- manage ingress tokens and compat session IDs
- cleanup child sessions and optional worktrees
- keep an always-on REPL bridge connection when enabled in app state
- page older assistant-mode remote history instead of assuming full transcript download

## Data Flow

```text
bridge CLI path
-> auth + gate checks
-> register environment
-> poll bridge API for work
-> spawn or reuse local session
-> connect session ingress / websocket state
-> forward permission requests and SDK messages
-> heartbeat active work items
-> cleanup timed out or completed sessions
```

Remote viewer flow:

```text
server events
-> SessionsWebSocket
-> RemoteSessionManager
-> sdkMessageAdapter
-> renderable internal messages

REPL bridge flow:

```text
AppState.replBridgeEnabled
-> useReplBridge()
-> initReplBridge()
-> register or reconnect bridge environment
-> flush initial transcript history
-> stream outbound local messages + inbound SDK/control messages
-> update REPL state and permission UI
```
```

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- Bridge auth must be checked before gate evaluation because user context affects gate freshness.
- Session ID compatibility is treated explicitly, which suggests server/client versions may differ.
- Permission prompts are bridged across processes, so the code synthesizes assistant/tool context when the local viewer does not have the original assistant message object.
- Heartbeat failure classification distinguishes auth expiry from fatal environment deletion.

## Non-Obvious Implementation Choices

### 1. Bridge mode is its own architecture

The code is too large and operationally rich to treat as a feature toggle on the normal REPL. It has its own:
- polling lifecycle
- backoff logic
- session spawning
- compat shims
- JWT refresh scheduling

### 2. Worktrees show up in remote infrastructure too

The bridge can create and clean up agent worktrees, which means session isolation is coupled to remote orchestration, not only local UI flows.

### 3. Remote permission handling is shape-preserving

`remotePermissionBridge.ts` builds synthetic assistant messages so downstream permission UIs can keep using the same tooling contract. That reuse is elegant and easy to miss.

### 4. Adapters protect the core runtime from transport-specific messiness

The remote viewer does not reimplement the entire message model. It adapts SDK/control/websocket events back into the local runtime's message semantics.

### 5. `bridgeMain` and `replBridge` are related but different runtimes

`bridgeMain.ts` is the background poll/spawn/heartbeat loop. [`src/bridge/replBridge.ts`](../../src/bridge/replBridge.ts) plus [`src/hooks/useReplBridge.tsx`](../../src/hooks/useReplBridge.tsx) are the live REPL-side connection layer that keeps a local interactive session mirrored to a remote environment. Treating them as one thing hides an important architecture split.

### 6. Assistant-mode history is paginated on purpose

[`src/assistant/sessionHistory.ts`](../../src/assistant/sessionHistory.ts) exposes `fetchLatestEvents()` and `fetchOlderEvents()` against remote session event APIs. That is a signal that long-lived remote sessions are expected to outgrow naive `download full transcript` assumptions.

## Agent-Builder Takeaways

- If you add remote control later, isolate it as a separate subsystem.
- Preserve core message and permission contracts across process boundaries by adapting into shared shapes.
- Explicitly plan for session-ID migrations and transport churn.
- Separate `background work acquisition` from `live mirrored session sync`.
- Design paginated session-history retrieval before remote sessions get large.

## Source Anchors

- [`src/bridge/bridgeMain.ts`](../../src/bridge/bridgeMain.ts)
- [`src/bridge/replBridge.ts`](../../src/bridge/replBridge.ts)
- [`src/bridge/bridgeMessaging.ts`](../../src/bridge/bridgeMessaging.ts)
- [`src/hooks/useReplBridge.tsx`](../../src/hooks/useReplBridge.tsx)
- [`src/assistant/sessionHistory.ts`](../../src/assistant/sessionHistory.ts)
- [`src/remote/RemoteSessionManager.ts`](../../src/remote/RemoteSessionManager.ts)
- [`src/remote/SessionsWebSocket.ts`](../../src/remote/SessionsWebSocket.ts)
- [`src/remote/sdkMessageAdapter.ts`](../../src/remote/sdkMessageAdapter.ts)

## [Inference]

This subsystem looks like the product had to grow from `local CLI` into `local CLI plus hosted/remote operator surface` without rewriting the core conversation runtime. The adapters and compat shims are exactly what you build under that pressure.
