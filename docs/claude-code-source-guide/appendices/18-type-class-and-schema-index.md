# 18 Type Class And Schema Index

This appendix lists the most important structural types, classes, and schema families visible in the snapshot.

## Runtime Classes

| Symbol | File | Role |
| --- | --- | --- |
| `QueryEngine` | [`src/QueryEngine.ts`](../../../src/QueryEngine.ts) | Owns conversation/session query lifecycle |
| `StreamingToolExecutor` | [`src/services/tools/StreamingToolExecutor.ts`](../../../src/services/tools/StreamingToolExecutor.ts) | Concurrent streaming tool execution with ordered output |
| `RemoteSessionManager` | [`src/remote/RemoteSessionManager.ts`](../../../src/remote/RemoteSessionManager.ts) | Remote session viewer controller |
| `SessionsWebSocket` | [`src/remote/SessionsWebSocket.ts`](../../../src/remote/SessionsWebSocket.ts) | Remote event websocket wrapper |
| `SdkControlClientTransport` | [`src/services/mcp/SdkControlTransport.ts`](../../../src/services/mcp/SdkControlTransport.ts) | MCP transport over SDK control channel |
| `SdkControlServerTransport` | [`src/services/mcp/SdkControlTransport.ts`](../../../src/services/mcp/SdkControlTransport.ts) | Server-side counterpart transport |
| `FlushGate` | [`src/bridge/flushGate.ts`](../../../src/bridge/flushGate.ts) | Bridge flush coordination |
| `BoundedUUIDSet` | [`src/bridge/bridgeMessaging.ts`](../../../src/bridge/bridgeMessaging.ts) | Bounded bridge message tracking |
| `ClaudeAuthProvider` | [`src/services/mcp/auth.ts`](../../../src/services/mcp/auth.ts) | MCP auth provider implementation |
| `McpAuthError` | [`src/services/mcp/client.ts`](../../../src/services/mcp/client.ts) | Auth-specific MCP tool error |
| `OAuthService` | [`src/services/oauth/index.ts`](../../../src/services/oauth/index.ts) | First-party OAuth PKCE orchestration |
| `AuthCodeListener` | [`src/services/oauth/auth-code-listener.ts`](../../../src/services/oauth/auth-code-listener.ts) | Localhost callback listener for OAuth handoff |

## High-Value Types

| Symbol | File | Notes |
| --- | --- | --- |
| `QueryEngineConfig` | [`src/QueryEngine.ts`](../../../src/QueryEngine.ts) | Per-session query runtime config |
| `ToolUseContext` | [`src/Tool.ts`](../../../src/Tool.ts) | Central tool execution context |
| `ToolPermissionContext` | [`src/Tool.ts`](../../../src/Tool.ts) | Permission rules and mode |
| `AppState` | [`src/state/AppStateStore.ts`](../../../src/state/AppStateStore.ts) | Large shared runtime/UI state shape |
| `AgentDefinition` | [`src/tools/AgentTool/loadAgentsDir.ts`](../../../src/tools/AgentTool/loadAgentsDir.ts) | Agent configuration surface |
| `BaseAgentDefinition` | [`src/tools/AgentTool/loadAgentsDir.ts`](../../../src/tools/AgentTool/loadAgentsDir.ts) | Common agent fields |
| `AgentProgress` | [`src/tasks/LocalAgentTask/LocalAgentTask.tsx`](../../../src/tasks/LocalAgentTask/LocalAgentTask.tsx) | Local agent progress and recent activity summary |
| `RemoteAgentTaskState` | [`src/tasks/RemoteAgentTask/RemoteAgentTask.tsx`](../../../src/tasks/RemoteAgentTask/RemoteAgentTask.tsx) | Remote/background task state including review metadata |
| `SwarmPermissionRequest` | [`src/utils/swarm/permissionSync.ts`](../../../src/utils/swarm/permissionSync.ts) | Worker-to-leader permission envelope |
| `SimpleCommand` | [`src/utils/bash/ast.ts`](../../../src/utils/bash/ast.ts) | Security-checked shell command shape |
| `ParseForSecurityResult` | [`src/utils/bash/ast.ts`](../../../src/utils/bash/ast.ts) | Shell parse result classification |
| `SedEditInfo` | [`src/tools/BashTool/sedEditParser.ts`](../../../src/tools/BashTool/sedEditParser.ts) | Parsed `sed -i` edit shape |
| `BackoffConfig` | [`src/bridge/bridgeMain.ts`](../../../src/bridge/bridgeMain.ts) | Bridge reconnect timing |
| `RemoteSessionConfig` | [`src/remote/RemoteSessionManager.ts`](../../../src/remote/RemoteSessionManager.ts) | Remote viewer session config |
| `ProjectConfig` | [`src/utils/config.ts`](../../../src/utils/config.ts) | Per-project persisted config |
| `GlobalConfig` | [`src/utils/config.ts`](../../../src/utils/config.ts) | Global persisted config |
| `MemoryType` | [`src/memdir/memoryTypes.ts`](../../../src/memdir/memoryTypes.ts) | Typed memory taxonomy |
| `DangerousPermissionInfo` | [`src/utils/permissions/permissionSetup.ts`](../../../src/utils/permissions/permissionSetup.ts) | Captures risky allow-rules for stripping and later restoration |
| `AutoModeRules` | [`src/utils/permissions/yoloClassifier.ts`](../../../src/utils/permissions/yoloClassifier.ts) | Default classifier-backed auto-mode rule families |
| `SyncState` | [`src/services/teamMemorySync/index.ts`](../../../src/services/teamMemorySync/index.ts) | Long-lived shared-memory sync state threaded through sync calls |
| `HandlerRegistrationResult` | [`src/services/lsp/passiveFeedback.ts`](../../../src/services/lsp/passiveFeedback.ts) | Return type for passive LSP notification wiring |
| `SecurityCheckResult` | [`src/services/remoteManagedSettings/securityCheck.tsx`](../../../src/services/remoteManagedSettings/securityCheck.tsx) | Outcome of managed-settings approval gate |

## SDK And Schema Families

| Family | File | Notes |
| --- | --- | --- |
| Core SDK schemas | [`src/entrypoints/sdk/coreSchemas.ts`](../../../src/entrypoints/sdk/coreSchemas.ts) | Main serializable SDK source of truth |
| Core SDK types | [`src/entrypoints/sdk/coreTypes.ts`](../../../src/entrypoints/sdk/coreTypes.ts) | Generated/public type re-exports |
| Control schemas | [`src/entrypoints/sdk/controlSchemas.ts`](../../../src/entrypoints/sdk/controlSchemas.ts) | SDK control channel validation |
| Agent SDK facade | [`src/entrypoints/agentSdkTypes.ts`](../../../src/entrypoints/agentSdkTypes.ts) | Public SDK exports |
| Settings schema/types | [`src/utils/settings/types.ts`](../../../src/utils/settings/types.ts) | Settings validation |
| Hook command / matcher / hooks schemas | [`src/schemas/hooks.ts`](../../../src/schemas/hooks.ts) | Persisted hook matcher DSL and cycle-broken schema extraction |
| Agent JSON schema | [`src/tools/AgentTool/loadAgentsDir.ts`](../../../src/tools/AgentTool/loadAgentsDir.ts) | Agent definition validation |
| MCP config schema | [`src/services/mcp/types.ts`](../../../src/services/mcp/types.ts) | MCP server config validation |
| Policy limits schema | [`src/services/policyLimits/types.ts`](../../../src/services/policyLimits/types.ts) | Org policy restrictions |
| Permission prompt/update schemas | [`src/utils/permissions/PermissionPromptToolResultSchema.ts`](../../../src/utils/permissions/PermissionPromptToolResultSchema.ts), [`src/utils/permissions/PermissionUpdateSchema.ts`](../../../src/utils/permissions/PermissionUpdateSchema.ts) | Permission wire/model surfaces |
| Remote managed settings response schema | [`src/services/remoteManagedSettings/types.ts`](../../../src/services/remoteManagedSettings/types.ts) | Managed-settings payload and cache validation |
| Team memory sync schemas | [`src/services/teamMemorySync/types.ts`](../../../src/services/teamMemorySync/types.ts) | Shared-memory request/response validation |

## Snapshot Gap

The message-type source file is missing from the visible `src/types` directory even though it is imported widely. That gap is central enough to mention again:

- expected but absent: `src/types/message.ts`
- reconstructed indirectly via [`src/utils/messages.ts`](../../../src/utils/messages.ts), [`src/utils/sessionStorage.ts`](../../../src/utils/sessionStorage.ts), and SDK schema files
