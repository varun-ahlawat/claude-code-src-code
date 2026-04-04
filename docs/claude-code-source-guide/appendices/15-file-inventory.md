# 15 File Inventory

This appendix accounts for every major top-level source area in `src/` and points to the highest-value files to read inside each one.

## Top-Level Source Areas

| Path | Approx files | What it appears to own |
| --- | ---: | --- |
| [`src/utils`](../../../src/utils) | 564 | Core runtime helpers: config, auth, session storage, shell parsing, permissions, prompt plumbing, computer-use integration |
| [`src/components`](../../../src/components) | 389 | REPL and terminal UI components, transcript renderers, MCP settings, permission flows, background-task dialogs |
| [`src/commands`](../../../src/commands) | 207 | Slash command surface and command-specific UIs |
| [`src/tools`](../../../src/tools) | 184 | Model-callable tool implementations |
| [`src/services`](../../../src/services) | 130 | API, MCP, analytics, compact, policy, OAuth, remote-managed settings, LSP, team-memory sync, voice |
| [`src/hooks`](../../../src/hooks) | 104 | React hooks and orchestration glue |
| [`src/ink`](../../../src/ink) | 96 | Terminal rendering/runtime stack |
| [`src/bridge`](../../../src/bridge) | 31 | Remote-control / bridge subsystem |
| [`src/bootstrap`](../../../src/bootstrap) | 1 | Early bootstrap-state helpers before full runtime is available |
| [`src/constants`](../../../src/constants) | 21 | Prompts, flags, string constants, product constants |
| [`src/skills`](../../../src/skills) | 20 | Bundled skills and skill loading helpers |
| [`src/cli`](../../../src/cli) | 19 | Headless / SDK-facing CLI orchestration |
| [`src/keybindings`](../../../src/keybindings) | 14 | Keybinding model and parsing |
| [`src/tasks`](../../../src/tasks) | 12 | Task runtime and task kinds |
| [`src/migrations`](../../../src/migrations) | 11 | Config and behavior migrations |
| [`src/types`](../../../src/types) | 7 visible | Shared TS types in the snapshot |
| [`src/context`](../../../src/context) | 9 | React contexts and providers |
| [`src/memdir`](../../../src/memdir) | 8 | Memory system |
| [`src/entrypoints`](../../../src/entrypoints) | 8 | CLI, SDK, MCP, init entry surfaces |
| [`src/assistant`](../../../src/assistant) | 1 | Assistant-mode helpers such as remote session history paging |
| [`src/buddy`](../../../src/buddy) | 6 | Companion/buddy UI behavior |
| [`src/coordinator`](../../../src/coordinator) | 1 | Coordinator-mode prompt and worker-contract logic |
| [`src/outputStyles`](../../../src/outputStyles) | 1 | Markdown-defined output-style loading |
| [`src/schemas`](../../../src/schemas) | 1 | Shared schema modules extracted to break import cycles |
| [`src/state`](../../../src/state) | 6 | Shared store and app state |
| [`src/moreright`](../../../src/moreright) | 1 | External-build stub overlay hook in this snapshot |
| [`src/vim`](../../../src/vim) | 5 | Vim interaction support |
| [`src/voice`](../../../src/voice) | 1 | Voice-mode feature gating and auth checks |
| [`src/query`](../../../src/query) | 4 | Query-loop helpers and budgeting |
| [`src/native-ts`](../../../src/native-ts) | 4 | Native TS wrappers and embedded runtime pieces |
| [`src/remote`](../../../src/remote) | 4 | Remote session viewer/adapters |
| [`src/server`](../../../src/server) | 3 | Direct connect / server types |
| [`src/screens`](../../../src/screens) | 3 | Top-level screen components |
| [`src/upstreamproxy`](../../../src/upstreamproxy) | 2 | Proxy-related code in snapshot |
| [`src/plugins`](../../../src/plugins) | 2 | Builtin plugin declarations |
| root singleton files | several | `main.tsx`, `QueryEngine.ts`, `query.ts`, `tools.ts`, `commands.ts`, etc. |

## Highest-Gravity Files

These are the best "read next" files if you want real architecture rather than leaf behavior:

| File | Why it matters |
| --- | --- |
| [`src/cli/print.ts`](../../../src/cli/print.ts) | Headless/SDK/runtime orchestration |
| [`src/utils/messages.ts`](../../../src/utils/messages.ts) | Message normalization, system messages, tool-result pairing |
| [`src/utils/sessionStorage.ts`](../../../src/utils/sessionStorage.ts) | Transcript persistence, resume, session metadata |
| [`src/screens/REPL.tsx`](../../../src/screens/REPL.tsx) | Interactive runtime hub |
| [`src/main.tsx`](../../../src/main.tsx) | Full startup bootstrap |
| [`src/utils/bash/bashParser.ts`](../../../src/utils/bash/bashParser.ts) | Tree-sitter bash parser integration |
| [`src/services/api/claude.ts`](../../../src/services/api/claude.ts) | Model request/stream/fallback plumbing |
| [`src/services/mcp/client.ts`](../../../src/services/mcp/client.ts) | MCP connection and tool wrapping |
| [`src/services/remoteManagedSettings/index.ts`](../../../src/services/remoteManagedSettings/index.ts) | Managed-settings fetch/apply/retry/security lifecycle |
| [`src/services/oauth/index.ts`](../../../src/services/oauth/index.ts) | First-party OAuth PKCE orchestration |
| [`src/services/teamMemorySync/index.ts`](../../../src/services/teamMemorySync/index.ts) | Repo-scoped shared-memory sync engine |
| [`src/services/lsp/manager.ts`](../../../src/services/lsp/manager.ts) | Startup-managed LSP singleton and diagnostics wiring |
| [`src/bridge/bridgeMain.ts`](../../../src/bridge/bridgeMain.ts) | Bridge poll/spawn/heartbeat loop |
| [`src/QueryEngine.ts`](../../../src/QueryEngine.ts) | Conversation lifecycle owner |
| [`src/query.ts`](../../../src/query.ts) | Core query state machine |
| [`src/services/compact/compact.ts`](../../../src/services/compact/compact.ts) | Context compaction logic |
| [`src/components/Message.tsx`](../../../src/components/Message.tsx) | Transcript render dispatch and message taxonomy hub |
| [`src/components/mcp/MCPSettings.tsx`](../../../src/components/mcp/MCPSettings.tsx) | Operator-facing MCP management surface |
| [`src/components/PromptInput/PromptInput.tsx`](../../../src/components/PromptInput/PromptInput.tsx) | Prompt/input orchestration and submission control plane |
| [`src/tasks/LocalAgentTask/LocalAgentTask.tsx`](../../../src/tasks/LocalAgentTask/LocalAgentTask.tsx) | Concrete local subagent runtime and progress tracking |
| [`src/coordinator/coordinatorMode.ts`](../../../src/coordinator/coordinatorMode.ts) | Distinct coordinator-mode prompt/runtime contract |

## High-Value Subroots Worth Reading

These subdirectories have outsized architectural value relative to their size:

| Path | Why it is worth reading |
| --- | --- |
| [`src/components/messages`](../../../src/components/messages) | Operational/system message render taxonomy |
| [`src/components/tasks`](../../../src/components/tasks) | Background-task review, local-agent detail, remote-session supervision |
| [`src/components/mcp`](../../../src/components/mcp) | MCP management UI, auth actions, elicitation dialog |
| [`src/components/permissions/rules`](../../../src/components/permissions/rules) | Editable vs managed permission-rule surfaces |
| [`src/services/oauth`](../../../src/services/oauth) | PKCE OAuth flow and callback listener |
| [`src/services/remoteManagedSettings`](../../../src/services/remoteManagedSettings) | Managed-settings lifecycle, cache, security review |
| [`src/services/teamMemorySync`](../../../src/services/teamMemorySync) | Repo-scoped shared memory sync and watcher logic |
| [`src/services/lsp`](../../../src/services/lsp) | LSP manager and passive diagnostics |
| [`src/utils/computerUse`](../../../src/utils/computerUse) | Built-in computer-use MCP server and lock/approval wrapper |

## Snapshot Gaps Worth Noting

- `src/types/message.ts` is not present even though many imports reference `src/types/message.js`.
- No visible tests exist under `src/`.
- No visible build metadata exists at the repo root.

These are not minor gaps. They should shape how confidently you generalize from the snapshot.
