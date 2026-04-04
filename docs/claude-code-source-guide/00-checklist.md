# 00 Checklist

This is the master coverage tracker for the guide. Items are checked only when the guide tree contains a corresponding chapter section or appendix entry.

## Baseline Inventory

- [x] Record snapshot caveats: no `package.json`, no `tsconfig`, no lockfile, no tests, extracted-from-monorepo state.
- [x] Capture the top-level repo map and explain why `src/` is effectively the whole product.
- [x] Record subsystem density so readers know where the architectural weight lives (`utils`, `components`, `commands`, `tools`, `services`, `hooks`).
- [x] Record that the repo contains about `1902` files under `src/`.
- [x] Record the largest/highest-gravity files and ensure each is covered somewhere.
- [x] Note that many `.tsx` files are React-compiler-transformed artifacts with embedded source maps.
- [x] Define glossary terms used throughout the guide: REPL, MCP, bridge, CCR, worktree, plan mode, compact, snip, session, task, swarm, teammate.

## Repo Map / Snapshot Caveats

- [x] Explain what this codebase is: CLI app, agent runtime, terminal UI, tool host, remote-control client/server hybrid.
- [x] Explain what is missing from the snapshot and how that affects confidence.
- [x] Separate `Observed` code from likely internal monorepo dependencies.
- [x] Document major source roots and what each one owns.
- [x] Provide a `where to start reading` path for new engineers and for coding agents.
- [x] Call out explicit external-build stubs and internal-only overlay files such as `src/moreright/useMoreRight.tsx`.

## Startup / Bootstrap / Entrypoints

- [x] Trace `src/entrypoints/cli.tsx` fast paths and why so much logic is gated before full CLI boot.
- [x] Explain build-time `feature()` gating as a first-class architectural mechanism.
- [x] Trace `src/main.tsx` startup sequence and early side effects.
- [x] Explain `src/setup.ts` ordering constraints, especially cwd, hooks, worktree, trust, and policy interactions.
- [x] Document interactive vs non-interactive vs SDK vs remote boot paths.
- [x] Document lazy imports, DCE boundaries, and circular-dependency avoidance patterns.

## Query Engine / Turn Lifecycle

- [x] Trace `QueryEngine.submitMessage()` from input to persisted state.
- [x] Trace `query()` / `queryLoop()` as the real conversation state machine.
- [x] Explain how system prompt, user context, tool context, and memory prompt are assembled.
- [x] Explain streaming vs non-streaming fallback behavior.
- [x] Explain tool-use streaming, buffering, ordering, and cancellation behavior.
- [x] Explain max-output-token recovery, token-budget continuation, and stop-hook handling.
- [x] Explain compaction triggers, reactive compact, snip handling, and history preservation rules.
- [x] Provide pseudocode for the end-to-end turn loop.

## Message Model / SDK / Schemas

- [x] Document the internal message model and how it differs from SDK-facing types.
- [x] Explain the `entrypoints/sdk/*` layering: core types, runtime types, control types, schemas.
- [x] Document transcript/session-chain concepts and parent UUID linkage.
- [x] Explain system/user/assistant/tool/progress/tombstone/result message variants.
- [x] Document normalization boundaries between UI-only messages and API-safe messages.
- [x] Summarize important Zod schema families and what stability guarantees they imply.
- [x] Provide pseudocode for message normalization and transcript reconstruction.

## Tool System / Permissions

- [x] Explain `Tool.ts` as the contract surface for tool execution.
- [x] Explain `tools.ts` as the source of truth for available tools.
- [x] Document tool registration, enablement, presets, deferred tools, and DCE-driven tool inclusion.
- [x] Explain permission context, permission modes, decision sources, and denial tracking.
- [x] Explain progress messages, tool result storage, and collapsed/expanded rendering behavior.
- [x] Explain concurrency-safe vs exclusive tools and the `StreamingToolExecutor` contract.
- [x] Explain synthetic tools and why they exist.
- [x] Provide pseudocode for a full tool call lifecycle, including permission, execution, progress, result mapping, and telemetry.

## Bash Parser / Validation / Security

- [x] Treat this as a standalone major chapter, not a subsection.
- [x] Explain why the repo has both legacy shell-quote logic and newer tree-sitter-based parsing.
- [x] Explain the fail-closed AST design and why `too complex` is a feature, not a bug.
- [x] Explain `parseCommandRaw`, `PARSE_ABORTED`, and the security significance of distinguishing parse failure from parser absence.
- [x] Explain `ast.ts` node allowlists, dangerous node families, placeholders, and command reconstruction.
- [x] Explain path validation, read-only validation, mode validation, destructive-command warnings, and wrapper stripping.
- [x] Explain `sedEditParser.ts` and why a special educational path exists for in-place edits.
- [x] Call out the `not a sandbox` distinction very explicitly.
- [x] Provide pseudocode for the bash security pipeline from raw command to permission decision.
- [x] Explain PowerShell parity and the shared cross-shell read-only validation tables in `src/utils/shell/readOnlyCommandValidation.ts`.

## Commands / Skills / Plugins

- [x] Explain slash commands as a second execution plane beside tool use.
- [x] Document command registry construction in `commands.ts`.
- [x] Distinguish prompt commands, local commands, local JSX commands, and lazy-shim commands.
- [x] Explain skills loading, frontmatter parsing, token estimation, path scoping, hooks, execution context, and argument substitution.
- [x] Explain bundled skills vs file-based skills vs MCP skills.
- [x] Explain plugin discovery, validation, enablement, marketplaces, session-only plugins, and cache/version layout.
- [x] Provide pseudocode for skill loading and plugin loading.
- [x] Include a startup lesson on why command/skill/plugin boundaries are worth keeping separate.
- [x] Document plugin marketplace management, source trust, install/update flows, and management UI surfaces.

## MCP / External Integrations

- [x] Explain MCP as a first-class extension mechanism in this codebase.
- [x] Document config parsing, scope resolution, deduplication, policy filtering, and enable/disable state.
- [x] Explain client transports: stdio, SSE, HTTP streamable, SDK control transport, in-process transport.
- [x] Explain auth flows, OAuth handling, token refresh, and auth error surfaces.
- [x] Explain tool/resource wrapping, description truncation, result transformation, and content persistence.
- [x] Explain elicitation handling and channel permissions.
- [x] Provide pseudocode for `connect config -> create client -> expose tools/resources -> call tool -> normalize output`.

## Bridge / Remote / Daemon / Session Plumbing

- [x] Explain bridge mode as a separate architecture, not a UI add-on.
- [x] Trace `src/bridge/bridgeMain.ts` poll/spawn/heartbeat/backoff loop.
- [x] Explain session IDs, compat shims, ingress tokens, and session spawners.
- [x] Explain worktree creation and tmux coordination in bridge flows.
- [x] Explain remote session adapters, websocket flows, permission bridging, and SDK-message adaptation.
- [x] Explain daemon/background session modes where visible in the snapshot.
- [x] Provide pseudocode for remote-control lifecycle from poll to child session to heartbeat to cleanup.
- [x] Explain `src/bridge/replBridge.ts` and `src/hooks/useReplBridge.tsx` as the always-on REPL sync path distinct from `bridgeMain.ts`.
- [x] Explain paginated assistant-mode remote history retrieval in `src/assistant/sessionHistory.ts`.

## State / React / Ink / Terminal Runtime

- [x] Explain the custom store in `src/state/store.ts` and why `useSyncExternalStore` is used instead of Redux.
- [x] Explain `AppStateStore` as the real app-wide contract.
- [x] Explain the REPL screen as the interactive hub and `src/cli/print.ts` as the headless/structured output hub.
- [x] Explain the custom Ink runtime and terminal parser stack.
- [x] Explain keyboard parsing, focus handling, scroll/layout machinery, and terminal semantic-action parsing.
- [x] Explain compiled React-compiler output in this snapshot and how to read through it.
- [x] Provide pseudocode for state updates across non-React and React consumers.
- [x] Explain `PromptInput`, `useTypeahead`, attachment paste flows, and slash-command submission as a major orchestration surface.
- [x] Explain `Messages`, `VirtualMessageList`, transcript search, and virtualization/performance guardrails.
- [x] Explain tool-specific permission request UIs and why `PermissionRequest.tsx` dispatches to per-tool components.

## Memory / Compaction / Session Storage

- [x] Explain memdir as an intentional memory subsystem, not `just files`.
- [x] Explain typed memory taxonomy and what is intentionally excluded from memory.
- [x] Explain entrypoint truncation limits and why they exist.
- [x] Explain relevant-memory lookup, extraction, auto-memory, and team-memory distinctions.
- [x] Explain session storage as a critical source of truth for transcripts and recovery.
- [x] Explain compact, microcompact, cached microcompact, snip, and post-compact cleanup.
- [x] Provide pseudocode for memory prompt assembly and compaction/recovery flow.

## Agents / Tasks / Worktrees / Swarms

- [x] Explain the Agent tool contract and subagent lifecycle.
- [x] Explain task abstractions and task state models.
- [x] Explain agent name registry, send-message routing, and foreground/background task viewing.
- [x] Explain forked subagents, worktree-backed agents, and in-process teammate flows where present.
- [x] Explain how swarms/teams are threaded through state, tools, and prompts.
- [x] Provide pseudocode for agent spawn, message routing, and result propagation.
- [x] Explain coordinator mode and its dedicated orchestrator system prompt / worker contract.
- [x] Explain swarm backend detection (tmux, iTerm2, in-process), teammate layout, and permission sync.
- [x] Explain `LocalAgentTask` and `RemoteAgentTask` as concrete execution state machines.
- [x] Explain agent authoring/editing UI and agent source precedence.

## Auth / Config / Policy / Analytics

- [x] Explain config layering and managed settings.
- [x] Explain trust, safe environment checks, permission bypass restrictions, and policy limits.
- [x] Explain auth surfaces: OAuth, API keys, cloud provider prefetch, secure storage.
- [x] Explain analytics/growthbook as both telemetry and runtime-behavior infrastructure.
- [x] Explain why policy and feature gates must be documented separately from static code paths.
- [x] Document the persisted hook matcher DSL and cycle-breaking extraction in `src/schemas/hooks.ts`.
- [x] Document markdown-defined output styles and precedence rules.
- [x] Document voice-mode visibility/auth gating as a compact feature-policy example.

## Surprising / Non-Obvious / Counterintuitive Design Choices

- [x] Write a dedicated chapter for startup lessons and `don't reinvent this badly` guidance.
- [x] Explain compile-time feature gating as an architecture-shaping choice.
- [x] Explain why lazy imports are used for more than performance: DCE, trust boundaries, cycle breaking, build slicing.
- [x] Explain why the bash pipeline intentionally fails closed and asks the user more often.
- [x] Explain why query recovery logic is so branchy: context exhaustion, streaming failures, tool/result integrity, stop hooks.
- [x] Explain why custom store + `useSyncExternalStore` is enough here and avoids heavyweight state machinery.
- [x] Explain why remote/bridge flows reuse the same message model via adapters.
- [x] Explain why session storage and compaction are core infrastructure, not optional polish.
- [x] Explain why worktree/project-root/session-root distinctions matter.
- [x] Explain why transformed source artifacts in the snapshot can mislead reverse-engineering if not called out.

## Appendices / Coverage Completeness

- [x] Build a file inventory appendix that accounts for every top-level source area.
- [x] Build a command index appendix.
- [x] Build a tool index appendix.
- [x] Build a type/class/schema index appendix.
- [x] Build a parser index appendix.
- [x] Build a feature-flag index appendix.
- [x] Ensure every major file family has at least one inbound link from a narrative chapter.
