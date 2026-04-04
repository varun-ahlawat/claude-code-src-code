# 07 Commands Skills And Plugins

## Purpose

Explain the product's other extension planes besides raw tool use: slash commands, markdown-defined skills, and plugins.

## Key Files And Symbols

- [`src/commands.ts`](../../src/commands.ts)
- [`src/skills/loadSkillsDir.ts`](../../src/skills/loadSkillsDir.ts)
- [`src/utils/plugins/pluginLoader.ts`](../../src/utils/plugins/pluginLoader.ts)
- [`src/utils/plugins/marketplaceManager.ts`](../../src/utils/plugins/marketplaceManager.ts)
- [`src/commands/plugin/ManagePlugins.tsx`](../../src/commands/plugin/ManagePlugins.tsx)
- [`src/components/mcp/MCPRemoteServerMenu.tsx`](../../src/components/mcp/MCPRemoteServerMenu.tsx)
- [`src/components/mcp/MCPStdioServerMenu.tsx`](../../src/components/mcp/MCPStdioServerMenu.tsx)
- [`src/tools/AgentTool/loadAgentsDir.ts`](../../src/tools/AgentTool/loadAgentsDir.ts)
- [`src/plugins/builtinPlugins.ts`](../../src/plugins/builtinPlugins.ts)

Important symbols:
- `COMMANDS`
- `getCommands()`
- `parseSkillFrontmatterFields()`
- `getSkillsPath()`
- `getVersionedCachePath()`
- `getMarketplace()`

## Core Types / Classes

Observed command model:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Observed skill-loading concepts from [`src/skills/loadSkillsDir.ts`](../../src/skills/loadSkillsDir.ts):


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Observed plugin-loading concepts:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Data Flow

Commands:

```text
command modules
-> registry assembly in commands.ts
-> feature-gated and source-aware filtering
-> interactive or headless invocation path
```

Skills:

```text
markdown files + frontmatter
-> path resolution by source
-> frontmatter parse
-> dedupe by canonical file identity
-> convert into command-like invocable surfaces
```

Plugins:

```text
plugin reference or session dir
-> resolve marketplace / trust policy / scope
-> fetch / clone / cache
-> manifest validation
-> hook / command / agent load
-> enable/disable and policy checks
```

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- Commands are product-authored runtime behavior.
- Skills are markdown-authored prompt behavior.
- Plugins are external code/content surfaces and therefore need validation, policy filtering, and cache isolation.
- MCP skills are called out as remote and untrusted; the skill loader explicitly avoids executing them inline.

## Non-Obvious Implementation Choices

### 1. Commands, skills, and plugins are intentionally separate layers

This avoids collapsing three different things into one abstraction:
- first-party executable behavior
- prompt-time task scaffolding
- externally sourced extension bundles

### 2. Skills are token-budget-aware

The loader estimates frontmatter token cost and delays full content loading until needed. That is a practical design for agent systems, not something generic markdown loaders usually care about.

### 3. Plugins have versioned cache paths

The plugin loader treats plugin fetch/install as a supply and cache problem:
- sanitize IDs
- derive versioned directories
- probe seed caches
- validate manifests

This is much more mature than `git clone somewhere and hope`.

### 4. Agents are also markdown-configured content

`loadAgentsDir.ts` shows that agent definitions are structurally similar to skills/plugins but add:
- tool restrictions
- MCP server specs
- hooks
- memory scope
- isolation mode

### 5. Plugin management is a product surface, not just a loader

[`src/utils/plugins/marketplaceManager.ts`](../../src/utils/plugins/marketplaceManager.ts) manages known marketplaces, cached manifests, fetch telemetry, allowlists/blocklists, and official marketplace sources. [`src/commands/plugin/ManagePlugins.tsx`](../../src/commands/plugin/ManagePlugins.tsx) then turns that machinery into a real management UI with enable/disable, update, uninstall, MCP inspection, and options flows.

## Plugin-Embedded MCP Management

One underdocumented detail is that plugin management reuses the MCP operator surface instead of inventing a separate plugin-specific inspector.

Observed behavior in [`src/commands/plugin/ManagePlugins.tsx`](../../src/commands/plugin/ManagePlugins.tsx):
- plugin detail state includes dedicated `mcp-detail`, `mcp-tools`, and `mcp-tool-detail` views
- those views reuse [`src/components/mcp/MCPStdioServerMenu.tsx`](../../src/components/mcp/MCPStdioServerMenu.tsx) and [`src/components/mcp/MCPRemoteServerMenu.tsx`](../../src/components/mcp/MCPRemoteServerMenu.tsx), which means plugin-owned servers participate in the same runtime management flows as first-class configured servers
- the UI exposes live discovered tools and tool details, so plugin MCP entries are not just manifest metadata; they are active runtime objects backed by actual server connection state

That is a useful design choice: plugins extend the same capability fabric rather than introducing a parallel one-off integration path.

## Agent-Builder Takeaways

- Do not merge commands, skills, and plugins into one vague extension model.
- Treat markdown-defined agent behavior as configuration with real validation.
- Version and sanitize plugin caches.
- Make token cost and invocation context first-class in any skill system.
- If you ship plugins broadly, design marketplace trust, cache cleanup, and management UI at the same time as the loader.

## Source Anchors

- [`src/commands.ts`](../../src/commands.ts)
- [`src/skills/loadSkillsDir.ts`](../../src/skills/loadSkillsDir.ts)
- [`src/utils/plugins/pluginLoader.ts`](../../src/utils/plugins/pluginLoader.ts)
- [`src/utils/plugins/marketplaceManager.ts`](../../src/utils/plugins/marketplaceManager.ts)
- [`src/commands/plugin/ManagePlugins.tsx`](../../src/commands/plugin/ManagePlugins.tsx)
- [`src/components/mcp/MCPRemoteServerMenu.tsx`](../../src/components/mcp/MCPRemoteServerMenu.tsx)
- [`src/components/mcp/MCPStdioServerMenu.tsx`](../../src/components/mcp/MCPStdioServerMenu.tsx)
- [`src/tools/AgentTool/loadAgentsDir.ts`](../../src/tools/AgentTool/loadAgentsDir.ts)

## [Inference]

This repo's extension model appears to have evolved in layers rather than being designed all at once. The code keeps those layers separate instead of papering over their differences, which is probably one reason the system remained extensible.
