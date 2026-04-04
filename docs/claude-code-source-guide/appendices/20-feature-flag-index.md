# 20 Feature Flag Index

This appendix lists compile-time feature flags observed via `feature('...')` calls in the snapshot. The count is a rough grep count, not semantic truth. It is still useful because it shows which product slices are architecturally important.

## High-Impact Flags

| Flag | Rough refs | What it appears to gate |
| --- | ---: | --- |
| `KAIROS` | 154 | Assistant/managed or specialized product mode |
| `TRANSCRIPT_CLASSIFIER` | 107 | Auto/permission classification behavior |
| `TEAMMEM` | 51 | Team memory features |
| `VOICE_MODE` | 46 | Voice interaction |
| `BASH_CLASSIFIER` | 45 | Bash/tool permission classifier |
| `KAIROS_BRIEF` | 39 | Brief/user-visible response surface |
| `PROACTIVE` | 37 | Proactive/scheduled or background behavior |
| `COORDINATOR_MODE` | 32 | Coordinator/team-style orchestration |
| `BRIDGE_MODE` | 28 | Remote-control / bridge mode |
| `EXPERIMENTAL_SKILL_SEARCH` | 21 | Skill discovery/search |
| `CONTEXT_COLLAPSE` | 20 | Context collapse behavior |
| `KAIROS_CHANNELS` | 19 | Channel integrations |
| `UDS_INBOX` | 17 | Local messaging / peer inbox |
| `BUDDY` | 16 | Companion/buddy UI |
| `CHICAGO_MCP` | 16 | Computer-use / specialized MCP flow |
| `HISTORY_SNIP` | 15 | Snip/history shortening |

## Full Extracted List

| Flag | Rough refs |
| --- | ---: |
| `ABLATION_BASELINE` | 1 |
| `AGENT_MEMORY_SNAPSHOT` | 2 |
| `AGENT_TRIGGERS` | 11 |
| `AGENT_TRIGGERS_REMOTE` | 2 |
| `ALLOW_TEST_VERSIONS` | 2 |
| `ANTI_DISTILLATION_CC` | 1 |
| `AUTO_THEME` | 2 |
| `AWAY_SUMMARY` | 2 |
| `BASH_CLASSIFIER` | 45 |
| `BG_SESSIONS` | 11 |
| `BREAK_CACHE_COMMAND` | 2 |
| `BRIDGE_MODE` | 28 |
| `BUDDY` | 16 |
| `BUILDING_CLAUDE_APPS` | 1 |
| `BUILTIN_EXPLORE_PLAN_AGENTS` | 1 |
| `BYOC_ENVIRONMENT_RUNNER` | 1 |
| `CACHED_MICROCOMPACT` | 12 |
| `CCR_AUTO_CONNECT` | 3 |
| `CCR_MIRROR` | 4 |
| `CCR_REMOTE_SETUP` | 1 |
| `CHICAGO_MCP` | 16 |
| `COMMIT_ATTRIBUTION` | 12 |
| `COMPACTION_REMINDERS` | 1 |
| `CONNECTOR_TEXT` | 7 |
| `CONTEXT_COLLAPSE` | 20 |
| `COORDINATOR_MODE` | 32 |
| `COWORKER_TYPE_TELEMETRY` | 2 |
| `DAEMON` | 3 |
| `DIRECT_CONNECT` | 5 |
| `DOWNLOAD_USER_SETTINGS` | 5 |
| `DUMP_SYSTEM_PROMPT` | 1 |
| `ENHANCED_TELEMETRY_BETA` | 2 |
| `EXPERIMENTAL_SKILL_SEARCH` | 21 |
| `EXTRACT_MEMORIES` | 7 |
| `FILE_PERSISTENCE` | 3 |
| `FORK_SUBAGENT` | 4 |
| `HARD_FAIL` | 2 |
| `HISTORY_PICKER` | 4 |
| `HISTORY_SNIP` | 15 |
| `HOOK_PROMPTS` | 1 |
| `IS_LIBC_GLIBC` | 1 |
| `IS_LIBC_MUSL` | 1 |
| `KAIROS` | 154 |
| `KAIROS_BRIEF` | 39 |
| `KAIROS_CHANNELS` | 19 |
| `KAIROS_DREAM` | 1 |
| `KAIROS_GITHUB_WEBHOOKS` | 3 |
| `KAIROS_PUSH_NOTIFICATION` | 4 |
| `LODESTONE` | 6 |
| `MCP_RICH_OUTPUT` | 3 |
| `MCP_SKILLS` | 9 |
| `MEMORY_SHAPE_TELEMETRY` | 3 |
| `MESSAGE_ACTIONS` | 5 |
| `MONITOR_TOOL` | 13 |
| `NATIVE_CLIENT_ATTESTATION` | 1 |
| `NATIVE_CLIPBOARD_IMAGE` | 2 |
| `NEW_INIT` | 2 |
| `OVERFLOW_TEST_TOOL` | 2 |
| `PERFETTO_TRACING` | 1 |
| `POWERSHELL_AUTO_MODE` | 2 |
| `PROACTIVE` | 37 |
| `PROMPT_CACHE_BREAK_DETECTION` | 9 |
| `QUICK_SEARCH` | 5 |
| `REACTIVE_COMPACT` | 4 |
| `REVIEW_ARTIFACT` | 4 |
| `RUN_SKILL_GENERATOR` | 1 |
| `SELF_HOSTED_RUNNER` | 1 |
| `SHOT_STATS` | 10 |
| `SKILL_IMPROVEMENT` | 1 |
| `SLOW_OPERATION_LOGGING` | 1 |
| `SSH_REMOTE` | 4 |
| `STREAMLINED_OUTPUT` | 1 |
| `TEAMMEM` | 51 |
| `TEMPLATES` | 6 |
| `TERMINAL_PANEL` | 4 |
| `TOKEN_BUDGET` | 9 |
| `TORCH` | 1 |
| `TRANSCRIPT_CLASSIFIER` | 107 |
| `TREE_SITTER_BASH` | 3 |
| `TREE_SITTER_BASH_SHADOW` | 5 |
| `UDS_INBOX` | 17 |
| `ULTRAPLAN` | 10 |
| `ULTRATHINK` | 1 |
| `UNATTENDED_RETRY` | 1 |
| `UPLOAD_USER_SETTINGS` | 2 |
| `VERIFICATION_AGENT` | 4 |
| `VOICE_MODE` | 46 |
| `WEB_BROWSER_TOOL` | 4 |
| `WORKFLOW_SCRIPTS` | 10 |

## Why This Appendix Matters

This product is not just `runtime flags sprinkled around`. Feature slicing is structurally important:
- it changes imports
- it changes visible tool and command surfaces
- it changes available UI flows
- it changes which subsystems even exist in a given build

That is why the main guide treats `feature()` as architecture rather than trivia.
