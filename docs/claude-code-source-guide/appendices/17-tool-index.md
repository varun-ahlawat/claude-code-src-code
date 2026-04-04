# 17 Tool Index

This appendix lists the major tool families visible in `src/tools` and `src/tools.ts`.

## Core Tool Families

| Tool family | Canonical module | Notes |
| --- | --- | --- |
| Agent tool | [`src/tools/AgentTool`](../../../src/tools/AgentTool) | Subagents, agent definitions, UI, routing |
| Ask user question | [`src/tools/AskUserQuestionTool`](../../../src/tools/AskUserQuestionTool) | Clarification tool, especially important in plan flows |
| Bash tool | [`src/tools/BashTool`](../../../src/tools/BashTool) | Shell execution plus extensive validation/security |
| Brief tool | [`src/tools/BriefTool`](../../../src/tools/BriefTool) | User-visible reply surface in some product modes |
| Config tool | [`src/tools/ConfigTool`](../../../src/tools/ConfigTool) | Config mutation/introspection |
| Enter / exit plan mode | [`src/tools/EnterPlanModeTool`](../../../src/tools/EnterPlanModeTool), [`src/tools/ExitPlanModeTool`](../../../src/tools/ExitPlanModeTool) | Mode-switch tools |
| Enter / exit worktree | [`src/tools/EnterWorktreeTool`](../../../src/tools/EnterWorktreeTool), [`src/tools/ExitWorktreeTool`](../../../src/tools/ExitWorktreeTool) | Worktree context switching |
| File edit | [`src/tools/FileEditTool`](../../../src/tools/FileEditTool) | Structured file mutation |
| File read | [`src/tools/FileReadTool`](../../../src/tools/FileReadTool) | Structured read path with limits |
| File write | [`src/tools/FileWriteTool`](../../../src/tools/FileWriteTool) | Structured write path |
| Glob | [`src/tools/GlobTool`](../../../src/tools/GlobTool) | File search |
| Grep | [`src/tools/GrepTool`](../../../src/tools/GrepTool) | Content search |
| LSP | [`src/tools/LSPTool`](../../../src/tools/LSPTool) | Symbol/query tooling backed by a startup-managed singleton; passive diagnostics live outside explicit tool calls |
| MCP tool wrapper | [`src/tools/MCPTool`](../../../src/tools/MCPTool) | Wraps MCP server tools |
| MCP auth tool | [`src/tools/McpAuthTool`](../../../src/tools/McpAuthTool) | Auth handoff helper |
| List/read MCP resources | [`src/tools/ListMcpResourcesTool`](../../../src/tools/ListMcpResourcesTool), [`src/tools/ReadMcpResourceTool`](../../../src/tools/ReadMcpResourceTool) | Resource-oriented MCP access |
| Notebook edit | [`src/tools/NotebookEditTool`](../../../src/tools/NotebookEditTool) | Notebook mutation |
| PowerShell | [`src/tools/PowerShellTool`](../../../src/tools/PowerShellTool) | Windows shell analogue to BashTool |
| Remote trigger | [`src/tools/RemoteTriggerTool`](../../../src/tools/RemoteTriggerTool) | Remote scheduled/task triggers |
| Send message | [`src/tools/SendMessageTool`](../../../src/tools/SendMessageTool) | Agent/task routing |
| Skill tool | [`src/tools/SkillTool`](../../../src/tools/SkillTool) | Skill invocation |
| Synthetic output | [`src/tools/SyntheticOutputTool`](../../../src/tools/SyntheticOutputTool) | Product-specific synthetic surface |
| Task tools | [`src/tools/TaskCreateTool`](../../../src/tools/TaskCreateTool), [`src/tools/TaskGetTool`](../../../src/tools/TaskGetTool), [`src/tools/TaskListTool`](../../../src/tools/TaskListTool), [`src/tools/TaskUpdateTool`](../../../src/tools/TaskUpdateTool), [`src/tools/TaskOutputTool`](../../../src/tools/TaskOutputTool), [`src/tools/TaskStopTool`](../../../src/tools/TaskStopTool) | Task lifecycle |
| Team tools | [`src/tools/TeamCreateTool`](../../../src/tools/TeamCreateTool), [`src/tools/TeamDeleteTool`](../../../src/tools/TeamDeleteTool) | Team/swarm orchestration |
| Todo write | [`src/tools/TodoWriteTool`](../../../src/tools/TodoWriteTool) | Todo tracking |
| Tool search | [`src/tools/ToolSearchTool`](../../../src/tools/ToolSearchTool) | Deferred tool discovery / reference loading |
| Web fetch | [`src/tools/WebFetchTool`](../../../src/tools/WebFetchTool) | URL fetch |
| Web search | [`src/tools/WebSearchTool`](../../../src/tools/WebSearchTool) | Search |

## Conditionally Included Tool Families

These are visible in `tools.ts` but feature-gated, env-gated, or ant-only:

| Tool / family | Gating clues |
| --- | --- |
| `REPLTool` | ant-only / REPL mode |
| `SleepTool` | `PROACTIVE` or `KAIROS` |
| cron tools | `AGENT_TRIGGERS` |
| `RemoteTriggerTool` | `AGENT_TRIGGERS_REMOTE` |
| `MonitorTool` | `MONITOR_TOOL` |
| `SendUserFileTool` | `KAIROS` |
| `PushNotificationTool` | `KAIROS` or push flag |
| `SubscribePRTool` | `KAIROS_GITHUB_WEBHOOKS` |
| `WorkflowTool` | `WORKFLOW_SCRIPTS` |
| `WebBrowserTool` | `WEB_BROWSER_TOOL` |
| `TerminalCaptureTool` | `TERMINAL_PANEL` |
| `CtxInspectTool` | `CONTEXT_COLLAPSE` |
| `SnipTool` | `HISTORY_SNIP` |
| `VerifyPlanExecutionTool` | env-gated verify flow |
| `ListPeersTool` | `UDS_INBOX` |
| `TungstenTool` | ant-only |
| `ConfigTool` | ant-only |
| `TestingPermissionTool` | test-only |

## Important Takeaway

`src/tools.ts` is the registry source of truth, but the actual visible tool surface is the product of:
- compile-time feature gating
- environment checks
- permission-mode filtering
- MCP availability
- sometimes optimistic deferred discovery
- startup-managed capability planes such as LSP initialization and built-in MCP servers

## Runtime Capability Planes Outside `src/tools`

Some important callable or semi-callable capability surfaces are not best understood as plain tool modules:

| Capability plane | Canonical modules | Why it matters |
| --- | --- | --- |
| Computer use MCP | [`src/utils/computerUse/mcpServer.ts`](../../../src/utils/computerUse/mcpServer.ts), [`src/utils/computerUse/wrapper.tsx`](../../../src/utils/computerUse/wrapper.tsx) | Built-in MCP server with lock, approval, app allowlist, and abort semantics |
| LSP passive diagnostics | [`src/services/lsp/passiveFeedback.ts`](../../../src/services/lsp/passiveFeedback.ts) | Emits ambient diagnostics/notifications outside direct LSP tool calls |
