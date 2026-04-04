# 06 Bash Parser Validation And Security

## Purpose

Document one of the most educational parts of the codebase: how the product tries to make shell execution useful without pretending it is safe by default. The chapter is bash-first because the parser work is deepest there, but the source also shows a shared cross-shell validation layer and a PowerShell analogue.

## Key Files And Symbols

- [`src/utils/bash/parser.ts`](../../src/utils/bash/parser.ts)
- [`src/utils/bash/ast.ts`](../../src/utils/bash/ast.ts)
- [`src/utils/bash/bashParser.ts`](../../src/utils/bash/bashParser.ts)
- [`src/tools/BashTool/bashPermissions.ts`](../../src/tools/BashTool/bashPermissions.ts)
- [`src/tools/BashTool/bashSecurity.ts`](../../src/tools/BashTool/bashSecurity.ts)
- [`src/tools/BashTool/pathValidation.ts`](../../src/tools/BashTool/pathValidation.ts)
- [`src/tools/BashTool/readOnlyValidation.ts`](../../src/tools/BashTool/readOnlyValidation.ts)
- [`src/tools/BashTool/sedEditParser.ts`](../../src/tools/BashTool/sedEditParser.ts)
- [`src/tools/PowerShellTool/PowerShellTool.tsx`](../../src/tools/PowerShellTool/PowerShellTool.tsx)
- [`src/utils/shell/readOnlyCommandValidation.ts`](../../src/utils/shell/readOnlyCommandValidation.ts)
- [`src/utils/powershell/parser.ts`](../../src/utils/powershell/parser.ts)

Important symbols:
- `parseCommand()`
- `parseCommandRaw()`
- `PARSE_ABORTED`
- `ParseForSecurityResult`
- `SimpleCommand`
- `ExternalCommandConfig`

## Core Types / Classes

Observed AST-based security model from [`src/utils/bash/ast.ts`](../../src/utils/bash/ast.ts):


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Observed parser result distinction from [`src/utils/bash/parser.ts`](../../src/utils/bash/parser.ts):


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


That distinction matters.

## Data Flow

Observed shell-safety pipeline:

```text
raw bash command
-> parser availability check
-> tree-sitter parse or legacy fallback
-> AST walk with explicit allowlist
-> classify as simple / too-complex / parse-unavailable
-> path and read-only validation
-> wrapper stripping and semantics checks
-> permission decision
-> actual bash execution

Cross-shell path visible in the source:

```text
shared safe-command maps
-> bash or powershell-specific parser / argument extraction
-> shell-specific path and read-only validators
-> common permission-decision model
```
```

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

### Not A Sandbox

Observed directly in source comments:
- the AST analyzer is not a sandbox
- it answers whether trustworthy `argv[]` extraction is possible
- it does not make dangerous commands harmless

### Fail-Closed Parsing

The AST walker allowlists only understood node types. Unknown or dangerous structure becomes `too-complex`, which routes the user toward explicit approval instead of silently pretending analysis succeeded.

### `PARSE_ABORTED` Matters

This is a subtle but extremely important design choice.

`null` means:
- parser unavailable
- feature off
- no parse attempted

`PARSE_ABORTED` means:
- parser loaded
- parse was attempted
- the parser hit timeout / panic / node-budget abort

If those cases are collapsed together, an attacker can trigger parser abort and fall back into a weaker legacy path.

## Non-Obvious Implementation Choices

### 1. "Too complex" is a security success case

Most startups would call this a parser failure and try harder to recover. This repo instead treats uncertainty as a reason to reintroduce human approval.

### 2. Placeholder-based substitution modeling

`ast.ts` uses placeholders like `__CMDSUB_OUTPUT__` and `__TRACKED_VAR__` to preserve shape without lying about runtime-determined expansion values.

### 3. There are multiple validators because no one validator is enough

The bash path combines:
- structural parse
- shell-operator detection
- heredoc edge cases
- path constraints
- read-only checks
- destructive command checks

That layered approach is more robust than any single parser or regex.

### 4. `sed` receives a special educational fast path

[`src/tools/BashTool/sedEditParser.ts`](../../src/tools/BashTool/sedEditParser.ts) recognizes simple in-place `sed -i 's/.../.../' file` edits so the UI can treat them more like file edits than opaque shell blobs.

### 5. The safety story is broader than bash

[`src/tools/PowerShellTool/PowerShellTool.tsx`](../../src/tools/PowerShellTool/PowerShellTool.tsx) is not a thin wrapper. It has its own parsing, permission UI, and read-only checks, while [`src/utils/shell/readOnlyCommandValidation.ts`](../../src/utils/shell/readOnlyCommandValidation.ts) provides shared command maps and UNC-path-risk helpers used across shell tools.

## Agent-Builder Takeaways

- Do not call your shell safety layer a sandbox unless it actually is one.
- Distinguish parser absence from parser failure.
- Fail closed on unknown structure.
- Preserve user approval as the escape hatch for uncertain cases.
- If shell edits are common, special-case high-value patterns like simple `sed` edits for better UX.
- If you support multiple shells, share the command-knowledge tables but keep parsing and path semantics shell-specific.

## Source Anchors

- [`src/utils/bash/parser.ts`](../../src/utils/bash/parser.ts)
- [`src/utils/bash/ast.ts`](../../src/utils/bash/ast.ts)
- [`src/tools/BashTool/bashPermissions.ts`](../../src/tools/BashTool/bashPermissions.ts)
- [`src/tools/BashTool/bashSecurity.ts`](../../src/tools/BashTool/bashSecurity.ts)
- [`src/tools/BashTool/pathValidation.ts`](../../src/tools/BashTool/pathValidation.ts)
- [`src/tools/BashTool/readOnlyValidation.ts`](../../src/tools/BashTool/readOnlyValidation.ts)
- [`src/tools/BashTool/sedEditParser.ts`](../../src/tools/BashTool/sedEditParser.ts)
- [`src/tools/PowerShellTool/PowerShellTool.tsx`](../../src/tools/PowerShellTool/PowerShellTool.tsx)
- [`src/utils/shell/readOnlyCommandValidation.ts`](../../src/utils/shell/readOnlyCommandValidation.ts)
- [`src/utils/powershell/parser.ts`](../../src/utils/powershell/parser.ts)

## [Inference]

The density of comments marked `SECURITY:` strongly suggests the shell layer was shaped by real bypasses and incident-driven hardening, not by greenfield design alone.
