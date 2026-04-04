# 19 Parser Index

This appendix lists parser-related modules in the snapshot and what each appears to parse.

## Shell And Command Parsers

| File | Scope |
| --- | --- |
| [`src/utils/bash/parser.ts`](../../../src/utils/bash/parser.ts) | Tree-sitter bash entry layer and parse result shaping |
| [`src/utils/bash/bashParser.ts`](../../../src/utils/bash/bashParser.ts) | Lower-level tree-sitter bash integration |
| [`src/utils/bash/ast.ts`](../../../src/utils/bash/ast.ts) | Fail-closed AST walk for security/argv extraction |
| [`src/tools/BashTool/sedEditParser.ts`](../../../src/tools/BashTool/sedEditParser.ts) | Simple `sed -i` edit parsing |
| [`src/utils/powershell/parser.ts`](../../../src/utils/powershell/parser.ts) | PowerShell parsing/analysis support |

## Terminal And Input Parsers

| File | Scope |
| --- | --- |
| [`src/ink/termio/parser.ts`](../../../src/ink/termio/parser.ts) | Streaming ANSI semantic parser |
| [`src/ink/parse-keypress.ts`](../../../src/ink/parse-keypress.ts) | Keypress parsing in terminal runtime |
| [`src/keybindings/parser.ts`](../../../src/keybindings/parser.ts) | Human-readable keybinding chord parsing |

## Config / Metadata Parsers

| File | Scope |
| --- | --- |
| [`src/utils/frontmatterParser.ts`](../../../src/utils/frontmatterParser.ts) | Markdown frontmatter for skills/agents/config |
| [`src/utils/permissions/permissionRuleParser.ts`](../../../src/utils/permissions/permissionRuleParser.ts) | Permission rule strings |
| [`src/utils/git/gitConfigParser.ts`](../../../src/utils/git/gitConfigParser.ts) | Git config parsing |
| [`src/utils/mcp/dateTimeParser.ts`](../../../src/utils/mcp/dateTimeParser.ts) | MCP date/time parsing helper |
| [`src/utils/deepLink/parseDeepLink.ts`](../../../src/utils/deepLink/parseDeepLink.ts) | Deep link parsing |
| [`src/commands/plugin/parseArgs.ts`](../../../src/commands/plugin/parseArgs.ts) | Plugin command argument parsing |
| [`src/utils/plugins/parseMarketplaceInput.ts`](../../../src/utils/plugins/parseMarketplaceInput.ts) | Marketplace/plugin identifier parsing |

## Why This Matters

The repo does not rely on one generic parsing strategy. It uses:
- AST parsing where trust matters
- simple token/chord parsing where UX matters
- frontmatter parsing where markdown becomes runtime configuration

That is a recurring theme in the codebase: parse at the level of certainty the product actually needs.
