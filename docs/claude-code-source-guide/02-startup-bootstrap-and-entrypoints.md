# 02 Startup Bootstrap And Entrypoints

## Purpose

Explain how Claude Code gets from a raw process invocation to a fully initialized agent runtime, and why startup ordering is a first-class design concern.

## Key Files And Symbols

- [`src/entrypoints/cli.tsx`](../../src/entrypoints/cli.tsx)
- [`src/main.tsx`](../../src/main.tsx)
- [`src/setup.ts`](../../src/setup.ts)
- [`src/entrypoints/init.ts`](../../src/entrypoints/init.ts)
- [`src/bootstrap/state.ts`](../../src/bootstrap/state.ts)
- [`src/services/remoteManagedSettings/index.ts`](../../src/services/remoteManagedSettings/index.ts)

Important symbols:
- `main()` in `entrypoints/cli.tsx`
- `feature()` build-time gate
- `profileCheckpoint()`
- `setup()`
- `setCwd()`
- `initializeRemoteManagedSettingsLoadingPromise()`

## Core Types / Classes

Conceptual startup model:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Data Flow

`entrypoints/cli.tsx` is the pre-bootstrap dispatcher.

Observed behavior:
- it handles special flags before loading the full CLI
- it avoids heavy imports on fast paths like `--version`
- it uses inline `feature()` guards so Bun can dead-code-eliminate excluded branches
- it runs different boot flows for bridge, daemon, background sessions, template jobs, and other specialized modes

`main.tsx` is the full bootstrap.

Observed startup concerns there:
- early profiler markers
- parallel prefetches for MDM and keychain
- trust/config/auth/growthbook initialization
- initialize a remote-managed-settings loading promise before downstream systems can await it
- remote-managed-settings load and periodic refresh wiring
- command/tool/plugin/MCP loading
- deferred LSP manager startup after config/plugin surfaces are available
- REPL launch or alternative control surface setup

`setup.ts` is the pre-query environment normalizer.

Observed ordering constraints:
- start optional UDS messaging before hooks inherit environment
- restore terminal backups before interactive UI work
- call `setCwd()` early
- snapshot hooks configuration before hidden mutation can happen
- initialize file change watchers
- perform worktree/tmux setup before command surface assumptions get locked in
- enforce dangerous permission bypass safety checks
- start long-lived repo-scoped services only after cwd, hooks, and project context are stable

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- `feature()` is not a runtime convenience. It is a compile-time slicing boundary.
- Trust must be established before some expensive or security-sensitive behavior becomes active.
- Worktree creation changes both cwd and project/session semantics.
- Permission bypass mode is explicitly constrained by environment safety checks.
- Hook configuration is captured early because later mutation would silently alter behavior.

## Non-Obvious Implementation Choices

### 1. Startup is split into two layers

`entrypoints/cli.tsx` is intentionally tiny so trivial paths avoid full module evaluation.

That matters because this product has:
- heavy UI/runtime imports
- many feature-gated branches
- security-sensitive initialization order

### 2. `feature()` must remain inline

Multiple comments explicitly warn that a seemingly harmless refactor can break dead-code elimination. This is unusual in normal app code, but completely rational in a product with:
- internal-only features
- multiple editions
- ant-only code
- optional native integrations

### 3. Early side effects are deliberate

`main.tsx` starts MDM and keychain work before the rest of imports finish. That is not stylistic noise; it is latency hiding.

### 4. `setup()` is not just setup

It is effectively a safety and environment transaction:
- sync the working directory
- establish trust assumptions
- capture hooks
- wire message buses
- prepare worktree state
- reject unsafe bypass modes

## Remote Managed Settings As A Startup Plane

The original chapter treated managed settings as a prefetched input, but the implementation is more disciplined than that.

Observed behavior across [`src/entrypoints/init.ts`](../../src/entrypoints/init.ts), [`src/main.tsx`](../../src/main.tsx), and [`src/services/remoteManagedSettings/index.ts`](../../src/services/remoteManagedSettings/index.ts):
- startup creates a shared loading promise up front so later code can await managed-settings completion without deadlocking if the load never starts
- the full runtime kicks off `loadRemoteManagedSettings()` in the background, then schedules refreshes separately from first-load behavior
- the managed-settings service handles eligibility checks, checksum/cached-state comparison, retry policy, and a blocking security approval step before newly fetched settings are applied
- the system explicitly models both `load` and `refresh` paths, which matters because startup wants bounded blocking while long-lived sessions want eventual convergence

This is a startup plane, not a leaf service, because it shapes what config and policy later subsystems are allowed to observe.

## Startup Also Wires Hidden Long-Lived Services

Two important subsystems become active during bootstrap even though they are not part of the user-visible startup story:
- the LSP manager is initialized as a singleton service, then later backs both explicit LSP tool calls and passive diagnostics/notifications
- team-memory synchronization watchers are started after setup stabilizes cwd and trust assumptions, which keeps repo-derived sync identity from drifting under the watcher

Those choices mean startup is responsible not only for creating the first request context, but also for activating background services whose correctness depends on early ordering.

## Agent-Builder Takeaways

- Build a tiny dispatcher in front of your real runtime.
- Separate compile-time product slicing from runtime policy gates.
- Treat cwd, trust, and hook state as startup-critical, not incidental.
- If hooks or subprocesses inherit environment, bind all inherited state before they spawn.

## Source Anchors

- [`src/entrypoints/cli.tsx`](../../src/entrypoints/cli.tsx)
- [`src/main.tsx`](../../src/main.tsx)
- [`src/setup.ts`](../../src/setup.ts)
- [`src/bootstrap/state.ts`](../../src/bootstrap/state.ts)
- [`src/entrypoints/init.ts`](../../src/entrypoints/init.ts)
- [`src/services/remoteManagedSettings/index.ts`](../../src/services/remoteManagedSettings/index.ts)

## [Inference]

The amount of effort spent on fast paths, DCE comments, and startup prefetch strongly suggests startup latency is a product metric, not just an implementation concern.
