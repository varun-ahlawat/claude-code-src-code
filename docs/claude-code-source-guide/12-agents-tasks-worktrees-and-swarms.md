# 12 Agents Tasks Worktrees And Swarms

## Purpose

Explain how Claude Code composes multiple agent threads, background tasks, isolated worktrees, and teammate-like orchestration.

## Key Files And Symbols

- [`src/tools/AgentTool/AgentTool.tsx`](../../src/tools/AgentTool/AgentTool.tsx)
- [`src/tools/AgentTool/runAgent.ts`](../../src/tools/AgentTool/runAgent.ts)
- [`src/tools/AgentTool/loadAgentsDir.ts`](../../src/tools/AgentTool/loadAgentsDir.ts)
- [`src/components/agents/AgentsMenu.tsx`](../../src/components/agents/AgentsMenu.tsx)
- [`src/tasks`](../../src/tasks)
- [`src/tasks/LocalAgentTask/LocalAgentTask.tsx`](../../src/tasks/LocalAgentTask/LocalAgentTask.tsx)
- [`src/tasks/RemoteAgentTask/RemoteAgentTask.tsx`](../../src/tasks/RemoteAgentTask/RemoteAgentTask.tsx)
- [`src/components/tasks/BackgroundTasksDialog.tsx`](../../src/components/tasks/BackgroundTasksDialog.tsx)
- [`src/components/tasks/AsyncAgentDetailDialog.tsx`](../../src/components/tasks/AsyncAgentDetailDialog.tsx)
- [`src/components/tasks/RemoteSessionDetailDialog.tsx`](../../src/components/tasks/RemoteSessionDetailDialog.tsx)
- [`src/tools/shared/spawnMultiAgent.ts`](../../src/tools/shared/spawnMultiAgent.ts)
- [`src/coordinator/coordinatorMode.ts`](../../src/coordinator/coordinatorMode.ts)
- [`src/utils/swarm/backends/registry.ts`](../../src/utils/swarm/backends/registry.ts)
- [`src/utils/swarm/permissionSync.ts`](../../src/utils/swarm/permissionSync.ts)
- [`src/utils/swarm/teammateLayoutManager.ts`](../../src/utils/swarm/teammateLayoutManager.ts)
- [`src/utils/worktree.ts`](../../src/utils/worktree.ts)
- [`src/utils/swarm/inProcessRunner.ts`](../../src/utils/swarm/inProcessRunner.ts)

Important symbols:
- `AgentDefinition`
- `runAgent()`
- `isCoordinatorMode()`
- `detectAndGetBackend()`
- `initializeAgentMcpServers()`
- task state maps in `AppState`
- `setAgentTranscriptSubdir()`

## Core Types / Classes

Observed agent-definition shape:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Observed app-state support for orchestration:
- task registry
- agent-name registry
- foregrounded task ID
- viewed teammate task ID
- remote background-task count

Observed task/runtime shapes:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Data Flow

```text
user or model invokes Agent tool
-> resolve agent definition
-> derive tool restrictions and inherited context
-> optionally attach agent-specific MCP servers
-> run agent query loop
-> record sidechain transcript + metadata
-> surface progress/result into parent thread
```

Worktree-backed flow:

```text
request isolation
-> create worktree
-> switch cwd / session state
-> run agent in isolated repo view
-> cleanup or persist worktree metadata
```

Coordinator / swarm flow:

```text
coordinator mode enabled
-> system prompt advertises worker contract
-> spawn workers via Agent tool
-> choose swarm backend (tmux / iTerm2 / in-process)
-> route teammate messages and permission requests
-> fold task notifications back into leader conversation
```

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- Agent definitions are configuration surfaces with real power: tools, MCP, hooks, and isolation mode.
- Plugin-only MCP restrictions still apply to user-controlled agents.
- Shared vs newly created MCP clients matter during cleanup.
- Task state is not just UI garnish; it determines routing, transcript grouping, and visibility.
- Coordinator mode changes the system prompt and user-context contract. Resuming a stored session in the wrong mode would corrupt expectations, which is why `matchSessionMode()` flips env state to match the persisted session.

## Non-Obvious Implementation Choices

### 1. Agents are not just prompts

An agent definition can shape:
- tools
- MCP servers
- hooks
- memory
- isolation
- max turns

That makes agent definitions operational bundles, not simple persona files.

### 2. Agent-specific MCP is additive

`runAgent.ts` shows subagents can bring their own MCP servers, merged with the parent context. That is a powerful extensibility move.

### 3. Sidechain transcripts are first-class

Subagents write their own transcript and metadata paths. This is a strong signal that multi-agent work is expected to be inspectable and resumable.

### 4. Worktrees are an orchestration primitive

Isolation is not only about shell sandboxing. The product can isolate agents by repository view.

### 5. Coordinator mode is a distinct runtime role

[`src/coordinator/coordinatorMode.ts`](../../src/coordinator/coordinatorMode.ts) is not merely a feature flag. It defines a different system prompt, different worker-tool context, and resume-time mode reconciliation. That is the codebase explicitly modeling `leader agent` and `worker agent` as separate roles.

### 6. Swarm execution is backend-adaptive

[`src/utils/swarm/backends/registry.ts`](../../src/utils/swarm/backends/registry.ts) chooses tmux, iTerm2, or in-process execution based on environment and cached detection. [`src/utils/swarm/teammateLayoutManager.ts`](../../src/utils/swarm/teammateLayoutManager.ts) then turns that into pane creation, color assignment, and layout behavior.

### 7. Swarm permissions are synchronized, not improvised

[`src/utils/swarm/permissionSync.ts`](../../src/utils/swarm/permissionSync.ts) and [`src/utils/swarm/leaderPermissionBridge.ts`](../../src/utils/swarm/leaderPermissionBridge.ts) show two strategies for leader-mediated permission handling: mailbox-backed worker/leader coordination and in-process queue bridging for teammates that share a process.

### 8. `LocalAgentTask` and `RemoteAgentTask` are concrete runtimes

[`src/tasks/LocalAgentTask/LocalAgentTask.tsx`](../../src/tasks/LocalAgentTask/LocalAgentTask.tsx) tracks token usage, recent activities, and task output paths for local subagents. [`src/tasks/RemoteAgentTask/RemoteAgentTask.tsx`](../../src/tasks/RemoteAgentTask/RemoteAgentTask.tsx) adds remote session polling, review progress tags, todo lists, and remote metadata persistence. These are not generic “task wrappers.”

### 9. Agents are editable product objects

[`src/components/agents/AgentsMenu.tsx`](../../src/components/agents/AgentsMenu.tsx) and related agent editor utilities show that agent definitions are expected to be created, validated, filtered by source, and edited interactively. This matters for anyone treating `agents/` as purely static config.

## Background Task And Remote Review Workflows

The task subsystem also has a substantial operator-facing review surface.

Observed behavior across [`src/components/tasks/BackgroundTasksDialog.tsx`](../../src/components/tasks/BackgroundTasksDialog.tsx), [`src/components/tasks/AsyncAgentDetailDialog.tsx`](../../src/components/tasks/AsyncAgentDetailDialog.tsx), and [`src/components/tasks/RemoteSessionDetailDialog.tsx`](../../src/components/tasks/RemoteSessionDetailDialog.tsx):
- `BackgroundTasksDialog.tsx` categorizes local agents, remote sessions, and foreground/leader state, then switches between list and detail views instead of treating background work as a notification badge
- `AsyncAgentDetailDialog.tsx` exposes progress summaries, recent activity, plan/prompt/error context, and explicit kill/back controls for local async agents
- `RemoteSessionDetailDialog.tsx` models remote review phases such as `needs_input` and `plan_ready`, shows tool-use summaries and review metadata, and provides termination/resume flows for long-lived remote sessions

This is the human inspection and control plane for multi-agent work. Without it, the architecture description overstates orchestration logic and understates how users actually supervise it.

## Agent-Builder Takeaways

- Treat agent definitions as executable configuration, not pure prompt text.
- Give subagents their own transcript identity.
- Isolate agents at the repository/worktree layer when collaboration or risky edits matter.
- Separate shared child resources from child-owned resources so cleanup is correct.
- Model coordinator/leader behavior explicitly if you want reliable multi-agent delegation.
- Decide early whether multi-agent execution uses panes, subprocesses, or in-process tasks, and how permission prompts cross that boundary.

## Source Anchors

- [`src/tools/AgentTool/loadAgentsDir.ts`](../../src/tools/AgentTool/loadAgentsDir.ts)
- [`src/tools/AgentTool/runAgent.ts`](../../src/tools/AgentTool/runAgent.ts)
- [`src/tools/AgentTool/AgentTool.tsx`](../../src/tools/AgentTool/AgentTool.tsx)
- [`src/components/agents/AgentsMenu.tsx`](../../src/components/agents/AgentsMenu.tsx)
- [`src/components/tasks/BackgroundTasksDialog.tsx`](../../src/components/tasks/BackgroundTasksDialog.tsx)
- [`src/components/tasks/AsyncAgentDetailDialog.tsx`](../../src/components/tasks/AsyncAgentDetailDialog.tsx)
- [`src/components/tasks/RemoteSessionDetailDialog.tsx`](../../src/components/tasks/RemoteSessionDetailDialog.tsx)
- [`src/coordinator/coordinatorMode.ts`](../../src/coordinator/coordinatorMode.ts)
- [`src/tasks/LocalAgentTask/LocalAgentTask.tsx`](../../src/tasks/LocalAgentTask/LocalAgentTask.tsx)
- [`src/tasks/RemoteAgentTask/RemoteAgentTask.tsx`](../../src/tasks/RemoteAgentTask/RemoteAgentTask.tsx)
- [`src/utils/swarm/backends/registry.ts`](../../src/utils/swarm/backends/registry.ts)
- [`src/utils/swarm/permissionSync.ts`](../../src/utils/swarm/permissionSync.ts)
- [`src/utils/swarm/teammateLayoutManager.ts`](../../src/utils/swarm/teammateLayoutManager.ts)
- [`src/utils/worktree.ts`](../../src/utils/worktree.ts)
- [`src/utils/swarm/inProcessRunner.ts`](../../src/utils/swarm/inProcessRunner.ts)

## [Inference]

The repo appears to be converging on a general `agent platform` inside the CLI, where the human-visible REPL is only one of several orchestration surfaces.
