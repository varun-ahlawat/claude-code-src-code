# 08 MCP And External Integrations

## Purpose

Explain how Claude Code treats MCP as a first-class extension and transport layer rather than a bolt-on integration.

## Key Files And Symbols

- [`src/services/mcp/client.ts`](../../src/services/mcp/client.ts)
- [`src/services/mcp/config.ts`](../../src/services/mcp/config.ts)
- [`src/services/mcp/auth.ts`](../../src/services/mcp/auth.ts)
- [`src/services/mcp/SdkControlTransport.ts`](../../src/services/mcp/SdkControlTransport.ts)
- [`src/services/mcp/InProcessTransport.ts`](../../src/services/mcp/InProcessTransport.ts)
- [`src/services/mcp/elicitationHandler.ts`](../../src/services/mcp/elicitationHandler.ts)
- [`src/components/mcp/MCPSettings.tsx`](../../src/components/mcp/MCPSettings.tsx)
- [`src/components/mcp/MCPListPanel.tsx`](../../src/components/mcp/MCPListPanel.tsx)
- [`src/components/mcp/MCPStdioServerMenu.tsx`](../../src/components/mcp/MCPStdioServerMenu.tsx)
- [`src/components/mcp/MCPRemoteServerMenu.tsx`](../../src/components/mcp/MCPRemoteServerMenu.tsx)
- [`src/components/mcp/ElicitationDialog.tsx`](../../src/components/mcp/ElicitationDialog.tsx)

Important symbols:
- `McpAuthError`
- `McpToolCallError`
- `connectToServer()`
- `fetchToolsForClient()`
- `parseMcpConfig()`
- `SdkControlClientTransport`

## Core Types / Classes

Observed MCP transport surface in this snapshot:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Observed product role of MCP:
- load configs from multiple scopes
- connect through multiple transports
- expose tools, prompts, and resources to the model
- bridge auth and elicitation flows
- normalize server output into transcript-safe content

## Data Flow

```text
settings / project / managed / plugin MCP configs
-> parse + dedupe + policy filter
-> connect using stdio / SSE / HTTP / SDK transport
-> discover tools / prompts / resources
-> wrap as Claude Code tools
-> call through transport
-> normalize result content
-> persist or truncate large output if needed
```

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- MCP config parsing is a trust boundary because configs can come from multiple scopes with different trust levels.
- Auth errors are promoted into explicit `needs-auth` style flows instead of being treated as generic tool failure.
- Server descriptions are truncated before being shown to the model, which protects context budget from generated or overly verbose tool docs.
- Tool results may need truncation or persistence when content is too large or binary.

## Non-Obvious Implementation Choices

### 1. MCP is woven into the tool system, not layered beside it

The client turns MCP tools into native tool objects instead of making the model learn a second invocation mechanism.

### 2. Multiple transports are treated as peers

This codebase supports:
- stdio subprocess servers
- SSE
- streamable HTTP
- in-process linked transports
- SDK control transport

That is a significant architecture choice. It means `MCP` here really means `capability fabric`, not just `launch one local server`.

### 3. Auth is productized

The MCP auth layer is large because connector auth is not a one-shot concern. It includes:
- stored client config
- token refresh
- step-up detection
- auth cache
- auth-specific error classes

### 4. Resources and prompts matter too

The presence of `ListMcpResourcesTool` and `ReadMcpResourceTool` shows MCP is not just about tools. The product exposes non-tool context surfaces as first-class model-readable capabilities.

## MCP Management Surfaces And Operator Flows

The transport/auth description only tells half the story. The repo also has a full operator-facing MCP management workflow.

Observed behavior across [`src/components/mcp/MCPSettings.tsx`](../../src/components/mcp/MCPSettings.tsx), [`src/components/mcp/MCPListPanel.tsx`](../../src/components/mcp/MCPListPanel.tsx), [`src/components/mcp/MCPStdioServerMenu.tsx`](../../src/components/mcp/MCPStdioServerMenu.tsx), and [`src/components/mcp/MCPRemoteServerMenu.tsx`](../../src/components/mcp/MCPRemoteServerMenu.tsx):
- the UI groups servers by source and status rather than showing a flat list
- stdio servers and remote servers have distinct management surfaces because their actionable operations differ
- remote flows include reconnect, enable/disable, authenticate, clear-auth, and tool inspection behavior that depends on current connection state
- agent-scoped MCP servers are surfaced alongside persistent servers, which makes temporary subagent capability injection inspectable instead of invisible

That means MCP here is not just a config format plus transport code. It is an actively managed runtime surface.

## Elicitation And Retry Workflow

[`src/services/mcp/elicitationHandler.ts`](../../src/services/mcp/elicitationHandler.ts) and [`src/components/mcp/ElicitationDialog.tsx`](../../src/components/mcp/ElicitationDialog.tsx) show another important plane: server-driven user input collection.

Observed behavior:
- MCP servers can request structured follow-up input that gets queued into app state instead of being improvised through generic tool errors
- the UI presents those requests in a dedicated elicitation dialog, then routes the user's answer back through the MCP client layer
- the flow includes retry and failure handling, which keeps long-lived MCP interactions resumable instead of turning every interrupted elicitation into a dead-end

This is one of the clearest signs that MCP is treated as an application protocol, not merely a subprocess transport.

## Agent-Builder Takeaways

- If you support MCP seriously, treat transport, auth, tool wrapping, and result normalization as separate subproblems.
- Do not let external tool descriptions or binary content blow up prompt budgets.
- Model-visible capabilities should feel native even if they come from heterogeneous backends.

## Source Anchors

- [`src/services/mcp/client.ts`](../../src/services/mcp/client.ts)
- [`src/services/mcp/config.ts`](../../src/services/mcp/config.ts)
- [`src/services/mcp/auth.ts`](../../src/services/mcp/auth.ts)
- [`src/services/mcp/SdkControlTransport.ts`](../../src/services/mcp/SdkControlTransport.ts)
- [`src/services/mcp/InProcessTransport.ts`](../../src/services/mcp/InProcessTransport.ts)
- [`src/services/mcp/elicitationHandler.ts`](../../src/services/mcp/elicitationHandler.ts)
- [`src/components/mcp/MCPSettings.tsx`](../../src/components/mcp/MCPSettings.tsx)
- [`src/components/mcp/MCPListPanel.tsx`](../../src/components/mcp/MCPListPanel.tsx)
- [`src/components/mcp/MCPStdioServerMenu.tsx`](../../src/components/mcp/MCPStdioServerMenu.tsx)
- [`src/components/mcp/MCPRemoteServerMenu.tsx`](../../src/components/mcp/MCPRemoteServerMenu.tsx)
- [`src/components/mcp/ElicitationDialog.tsx`](../../src/components/mcp/ElicitationDialog.tsx)
- [`src/tools/MCPTool/MCPTool.ts`](../../src/tools/MCPTool/MCPTool.ts)

## [Inference]

The product likely uses the same MCP substrate to power local extensions, internal connector surfaces, and SDK-integrated tools. The transport diversity and auth complexity strongly point that way.
