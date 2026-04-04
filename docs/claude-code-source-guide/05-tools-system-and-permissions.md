# 05 Tools System And Permissions

## Purpose

Explain how the product exposes capabilities to the model, decides whether they are callable, executes them, and maps results back into the conversation.

## Key Files And Symbols

- [`src/Tool.ts`](../../src/Tool.ts)
- [`src/tools.ts`](../../src/tools.ts)
- [`src/services/tools/toolExecution.ts`](../../src/services/tools/toolExecution.ts)
- [`src/services/tools/StreamingToolExecutor.ts`](../../src/services/tools/StreamingToolExecutor.ts)
- [`src/hooks/useCanUseTool.tsx`](../../src/hooks/useCanUseTool.tsx)
- [`src/utils/permissions/permissions.ts`](../../src/utils/permissions/permissions.ts)
- [`src/hooks/toolPermission/PermissionContext.ts`](../../src/hooks/toolPermission/PermissionContext.ts)
- [`src/utils/permissions/permissionSetup.ts`](../../src/utils/permissions/permissionSetup.ts)
- [`src/utils/permissions/PermissionUpdate.ts`](../../src/utils/permissions/PermissionUpdate.ts)
- [`src/utils/permissions/permissionsLoader.ts`](../../src/utils/permissions/permissionsLoader.ts)
- [`src/services/lsp/manager.ts`](../../src/services/lsp/manager.ts)
- [`src/utils/computerUse/wrapper.tsx`](../../src/utils/computerUse/wrapper.tsx)

Important symbols:
- `Tool`
- `ToolUseContext`
- `ToolPermissionContext`
- `getAllBaseTools()`
- `runToolUse()`
- `applyPermissionUpdates()`
- `stripDangerousPermissionsForAutoMode()`
- `awaitClassifierAutoApproval()`

## Core Types / Classes

Observed contract surface from [`src/Tool.ts`](../../src/Tool.ts):


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Observed registry behavior from [`src/tools.ts`](../../src/tools.ts):
- `getAllBaseTools()` is the source of truth for available tools
- compile-time flags and environment checks remove tools before the model sees them
- some tool requires are lazy to break circular dependencies

## Data Flow

Observed lifecycle:

```text
tool registry construction
-> active tool filtering
-> tool schema shown to model
-> assistant emits tool_use block
-> permission system evaluates request
-> tool executes
-> progress and telemetry emitted
-> result normalized into tool_result message
-> query loop continues
```

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- Tool filtering before the model sees schemas is a trust boundary.
- Permission evaluation is not just UI gating; it changes which actions are allowed to exist in-session.
- Tool result pairing is mandatory for transcript integrity.
- Telemetry-safe error classification matters because external/minified builds may mangle constructor names.

## Non-Obvious Implementation Choices

### 1. Availability is decided before prompting, not only at call time

Blanket-denied tools can be removed from the visible tool pool entirely. That reduces useless tool-use attempts.

### 2. Tool execution and tool permission are separate layers

This lets the product:
- distinguish config-based deny from user reject from hook-based deny
- emit richer telemetry
- support headless and interactive approval paths

### 3. Streaming execution is concurrency-aware

`StreamingToolExecutor` tracks:
- queued vs executing vs completed status
- concurrency-safe vs exclusive tools
- sibling abort behavior
- result ordering

This is more disciplined than naive `Promise.all` tool execution.

### 4. Tools are product-sliced by compile-time flags

This repo is full of optional tools:
- bridge/remote tools
- worktree tools
- cron/trigger tools
- browser/terminal capture tools
- ant-only tools

The model sees a tool surface that is already edition-specific.

## Permission Rule Lifecycle And Classifier-Assisted Approval

The existing explanation is directionally right, but the actual permission system is more than `allow / ask / deny`.

Observed behavior across [`src/hooks/toolPermission/PermissionContext.ts`](../../src/hooks/toolPermission/PermissionContext.ts), [`src/utils/permissions/permissionSetup.ts`](../../src/utils/permissions/permissionSetup.ts), [`src/utils/permissions/PermissionUpdate.ts`](../../src/utils/permissions/PermissionUpdate.ts), and [`src/utils/permissions/permissionsLoader.ts`](../../src/utils/permissions/permissionsLoader.ts):
- rule sets are loaded from multiple sources, then split into `alwaysAllow`, `alwaysAsk`, and `alwaysDeny` maps scoped by source
- some rules are managed or policy-controlled and therefore intentionally non-editable from normal interactive flows
- before a prompt is shown, rule visibility is filtered again so the human only sees actionable state, not every raw backing rule
- approval is a pipeline: static rule match, hook decision, classifier auto-approval for some Bash requests, and finally explicit user review when required
- accepted or rejected decisions are converted into `PermissionUpdate` objects so the runtime can mutate both in-memory session context and persisted config coherently
- entering or leaving auto mode is not a cosmetic toggle: dangerous broad allow-rules are stripped for the duration of auto mode and later restored when leaving it
- exiting plan mode can itself generate permission mutations, because the plan-exit surface allows prompt-scoped rule changes alongside the mode transition

This matters because permissions in Claude Code are runtime state with lifecycle and rollback semantics, not just modal UI decisions.

## LSP And Computer-Use Capability Planes

Two important capability planes sit partly outside the obvious `src/tools` story:

- [`src/tools/LSPTool/LSPTool.ts`](../../src/tools/LSPTool/LSPTool.ts) is the explicit model-callable LSP surface, but it depends on a startup-managed singleton in [`src/services/lsp/manager.ts`](../../src/services/lsp/manager.ts) and feeds passive notifications through [`src/services/lsp/passiveFeedback.ts`](../../src/services/lsp/passiveFeedback.ts). In other words, the LSP subsystem has both active and ambient behavior.
- Computer use is implemented through a built-in MCP server plus a wrapper layer in [`src/utils/computerUse/wrapper.tsx`](../../src/utils/computerUse/wrapper.tsx) and [`src/utils/computerUse/mcpServer.ts`](../../src/utils/computerUse/mcpServer.ts). That layer handles global locking, `Esc` abort semantics, allowed-app persistence, and explicit approval UI rather than behaving like an ordinary stateless tool.

If you only read `src/tools.ts`, both of these look smaller than they really are.

## Agent-Builder Takeaways

- Keep the tool contract centralized.
- Separate tool visibility, tool permission, and tool execution.
- Treat partial tool execution as a transcript problem, not just an exception path.
- If tools can stream, explicitly model concurrency class and interruption behavior.

## Source Anchors

- [`src/Tool.ts`](../../src/Tool.ts)
- [`src/tools.ts`](../../src/tools.ts)
- [`src/services/tools/toolExecution.ts`](../../src/services/tools/toolExecution.ts)
- [`src/services/tools/StreamingToolExecutor.ts`](../../src/services/tools/StreamingToolExecutor.ts)
- [`src/utils/permissions/permissions.ts`](../../src/utils/permissions/permissions.ts)
- [`src/hooks/toolPermission/PermissionContext.ts`](../../src/hooks/toolPermission/PermissionContext.ts)
- [`src/utils/permissions/permissionSetup.ts`](../../src/utils/permissions/permissionSetup.ts)
- [`src/utils/permissions/PermissionUpdate.ts`](../../src/utils/permissions/PermissionUpdate.ts)
- [`src/utils/permissions/permissionsLoader.ts`](../../src/utils/permissions/permissionsLoader.ts)
- [`src/services/lsp/manager.ts`](../../src/services/lsp/manager.ts)
- [`src/services/lsp/passiveFeedback.ts`](../../src/services/lsp/passiveFeedback.ts)
- [`src/utils/computerUse/wrapper.tsx`](../../src/utils/computerUse/wrapper.tsx)
- [`src/utils/computerUse/mcpServer.ts`](../../src/utils/computerUse/mcpServer.ts)

## [Inference]

This tool system was designed to support multiple hosts: REPL, structured SDK, remote viewers, and background agents. The richness of `ToolUseContext` strongly implies that tool execution is one of the product's most reused internal interfaces.
