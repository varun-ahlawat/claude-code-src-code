# 21 Third-Party Audit Checklist

Use this appendix as the review brief for an independent auditor of this guide tree.

The goal is not to praise the documentation. The goal is to find what it still misses, what it overstates, and what it treats as fully covered when the source suggests otherwise.

## Audit Message

Please audit this guide as if you were trying to break confidence in it.

Assume the current guide is useful but incomplete. Your job is to find:
- source areas that are materially underdocumented
- concepts that are named but not actually explained
- incorrect or overconfident claims
- places where `[Inference]` should replace direct prose
- areas where the correction pass still left major blind spots

Do not optimize for cosmetic feedback. Optimize for high-signal omissions that would matter to:
- a startup trying to build a similar product
- an agent trying to navigate this codebase safely
- a reviewer trying to understand the system's trust boundaries

## Required Output

Return findings in this format:

```md
## Findings

### 1. [Severity: high|medium|low] Missing or underdocumented topic
- Why it matters
- Which guide chapter or appendix should cover it
- Which source files prove it matters
- Whether the gap is:
  - completely missing
  - mentioned but underexplained
  - explained incorrectly
  - inferred too confidently

### 2. ...

## Coverage Verdict
- strongest-covered areas
- weakest-covered areas
- whether the guide is currently safe to call "exhaustive at the subsystem level"

## Suggested Doc Patches
- chapter to amend
- exact section to add or expand
- source anchors to include
```

Use file paths and concrete symbols. Do not give generic advice.

## Audit Standard

Only report a miss if at least one of these is true:
- the source contains a distinct subsystem that the guide does not explain
- the guide cites a file but does not explain the behavior that makes it important
- a large or high-gravity file contains architecture-significant logic absent from the guide
- a small file carries surprising product semantics that the guide currently hides
- the guide compresses multiple different execution planes into one description
- the guide implies certainty where the snapshot only supports inference

Do not report:
- style preferences
- formatting preferences
- requests for more examples unless the missing example hides a real concept

## Method

### 1. Validate the guide's claims against the repo

- Start with [`../00-checklist.md`](../00-checklist.md) and the narrative chapters.
- For each chapter, sample both:
  - files it cites directly
  - adjacent sibling files in the same subsystem that it does not cite
- Look for `high-gravity file omitted`, `important sibling omitted`, and `second-order concept omitted`.

### 2. Hunt for "mentioned but not covered"

The guide may mention a file family without actually documenting the architecture behind it.

For each chapter, ask:
- does it describe the control flow
- does it describe the data model
- does it describe the trust boundary
- does it describe why this design exists
- does it distinguish local runtime vs remote/runtime UI vs config/runtime side effects

If the answer is "no" for a materially important source area, that is a real finding.

### 3. Hunt for second-order misses from the correction pass

The correction pass intentionally strengthened:
- coordinator mode
- swarm backends
- prompt input and typeahead
- permission UI
- repl bridge
- plugin marketplaces
- hook schemas
- output styles
- voice gating
- PowerShell parity

The likely remaining misses are therefore one layer below those topics:
- concrete subcomponents and flows inside those areas
- adjacent UI surfaces
- supporting services that make those features operational

### 4. Test whether appendices overstate completeness

Check whether the appendices actually support the narrative claim of subsystem-level exhaustiveness.

Specifically test:
- whether top-level roots with product significance appear in [`15-file-inventory.md`](./15-file-inventory.md)
- whether important schema surfaces appear in [`18-type-class-and-schema-index.md`](./18-type-class-and-schema-index.md)
- whether the guide still has clusters of uncited or underdescribed files in dense areas like `components`, `hooks`, `services`, and `utils`

## High-Suspicion Areas

These are the places most likely to still contain meaningful misses after the correction run.

### UI and transcript behavior

- [`src/components/messages`](../../../src/components/messages)
- [`src/components/Message.tsx`](../../../src/components/Message.tsx)
- [`src/components/MessageRow.tsx`](../../../src/components/MessageRow.tsx)
- [`src/components/MessageSelector.tsx`](../../../src/components/MessageSelector.tsx)
- [`src/components/LogSelector.tsx`](../../../src/components/LogSelector.tsx)
- [`src/components/tasks`](../../../src/components/tasks)
- [`src/components/design-system`](../../../src/components/design-system)

Audit questions:
- Does the guide explain the actual message rendering taxonomy, or only the list-level virtualization?
- Does it explain rewind/restore/summarize UI surfaces well enough?
- Does it explain task-detail and background-task operator surfaces, or only task state?

### Permission and operator UX

- [`src/components/permissions`](../../../src/components/permissions)
- [`src/components/permissions/rules`](../../../src/components/permissions/rules)
- [`src/components/permissions/BashPermissionRequest/BashPermissionRequest.tsx`](../../../src/components/permissions/BashPermissionRequest/BashPermissionRequest.tsx)
- [`src/components/permissions/ExitPlanModePermissionRequest/ExitPlanModePermissionRequest.tsx`](../../../src/components/permissions/ExitPlanModePermissionRequest/ExitPlanModePermissionRequest.tsx)

Audit questions:
- Does the guide distinguish policy logic from operator workflow?
- Does it explain permission rule editing and persistence, not just request dialogs?
- Does it explain classifier-assisted approval behavior accurately?

### MCP and plugin-adjacent UI

- [`src/components/mcp`](../../../src/components/mcp)
- [`src/commands/plugin`](../../../src/commands/plugin)
- [`src/services/mcp`](../../../src/services/mcp)

Audit questions:
- Does the guide explain the MCP operator UI surfaces, or only the transport/client layer?
- Does it cover remote vs stdio MCP management workflows?
- Does it explain plugin-management UI as a workflow, not just marketplace caching?

### Hooks, notifications, and side-effect surfaces

- [`src/utils/hooks.ts`](../../../src/utils/hooks.ts)
- [`src/hooks/notifs`](../../../src/hooks/notifs)
- [`src/services/oauth`](../../../src/services/oauth)
- [`src/services/remoteManagedSettings`](../../../src/services/remoteManagedSettings)

Audit questions:
- Does the guide cover notification infrastructure deeply enough?
- Does it cover hook execution orchestration, or mostly hook schema shape?
- Does it explain managed-settings propagation and OAuth operational flows well enough?

### Suggestions, input support, and agent ergonomics

- [`src/utils/suggestions`](../../../src/utils/suggestions)
- [`src/utils/attachments.ts`](../../../src/utils/attachments.ts)
- [`src/hooks/useTypeahead.tsx`](../../../src/hooks/useTypeahead.tsx)

Audit questions:
- Does the guide explain suggestion sources and background indexing architecture?
- Does it document attachment ingestion and persisted pasted content behavior enough for an agent-builder?

### Tooling and side services

- [`src/services/lsp`](../../../src/services/lsp)
- [`src/utils/computerUse`](../../../src/utils/computerUse)
- [`src/services/teamMemorySync`](../../../src/services/teamMemorySync)
- [`src/tools/LSPTool`](../../../src/tools/LSPTool)

Audit questions:
- Are these just mentioned in appendices, or actually explained where needed?
- Do they materially affect agent capability modeling or trust boundaries?

## Concrete Checklist

### Coverage Validation

- [ ] Verify every top-level `src/*` root with product significance is either narratively documented or explicitly called out as a gap.
- [ ] Verify every chapter documents control flow, not just file names.
- [ ] Verify every chapter with strong claims also documents trust boundaries or failure modes.
- [ ] Verify every appendix entry that implies completeness is backed by narrative coverage somewhere.

### Source-Grounding Validation

- [ ] Find every place where the guide sounds certain and test whether the source really proves it.
- [ ] Check that `[Inference]` blocks are used where the snapshot is incomplete.
- [ ] Flag any place where the guide reconstructs missing source too confidently.

### Omission Hunting

- [ ] Sample at least five large files not directly cited in the chapters and test whether they expose missing subsystem behavior.
- [ ] Sample at least five small but semantically heavy files and test whether they reveal product seams the guide hides.
- [ ] Check whether the guide undercovers operator-facing UI relative to backend/runtime logic.
- [ ] Check whether the guide undercovers management surfaces relative to loader/registry surfaces.

### Correction-Pass Regression Checks

- [ ] Confirm the correction pass really covered coordinator mode beyond naming it.
- [ ] Confirm the correction pass really covered swarm backend detection and permission sync, not just "swarms exist".
- [ ] Confirm the correction pass really covered `PromptInput` and `useTypeahead` as a control plane, not just as input widgets.
- [ ] Confirm the correction pass really covered `replBridge` as distinct from `bridgeMain`.
- [ ] Confirm the correction pass really covered PowerShell as part of the shell-safety story, not as a footnote.
- [ ] Confirm the correction pass really covered plugin marketplaces as product operations, not just as cache directories.

### Strong Candidates For New Findings

- [ ] `components/messages` subtree may still be underexplained relative to `Messages.tsx`.
- [ ] `components/tasks` subtree may still be underexplained relative to `LocalAgentTask` and `RemoteAgentTask`.
- [ ] `components/mcp` subtree may still be underexplained relative to `services/mcp/client.ts`.
- [ ] `components/permissions/rules` may still be underexplained relative to permission request dialogs.
- [ ] `services/oauth`, `services/remoteManagedSettings`, and `services/teamMemorySync` may still be underrepresented in config/auth chapters.
- [ ] `services/lsp` and `tools/LSPTool` may still be underrepresented in tool-capability modeling.
- [ ] `utils/suggestions` and `hooks/notifs` may still hide user-facing orchestration that the guide compresses away.
- [ ] `utils/computerUse` may still represent an unmodeled capability seam.

## Verdict Threshold

You should only mark this guide as "exhaustive at the subsystem level" if:
- no major execution plane appears materially underdocumented
- no major trust boundary is missing from the relevant chapter
- no materially important top-level root is absent from the repo map and file inventory
- the remaining issues are leaf-level or example-level, not subsystem-level

If any of those fail, explicitly say the guide is valuable but not yet exhaustive.
