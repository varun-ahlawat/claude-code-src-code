# 13 Auth Config Policy And Analytics

## Purpose

Explain the cross-cutting infrastructure that shapes runtime behavior before any user turn happens: config, auth, policy, and analytics/feature delivery.

## Key Files And Symbols

- [`src/utils/config.ts`](../../src/utils/config.ts)
- [`src/utils/auth.ts`](../../src/utils/auth.ts)
- [`src/schemas/hooks.ts`](../../src/schemas/hooks.ts)
- [`src/outputStyles/loadOutputStylesDir.ts`](../../src/outputStyles/loadOutputStylesDir.ts)
- [`src/voice/voiceModeEnabled.ts`](../../src/voice/voiceModeEnabled.ts)
- [`src/services/policyLimits/index.ts`](../../src/services/policyLimits/index.ts)
- [`src/services/analytics/growthbook.ts`](../../src/services/analytics/growthbook.ts)
- [`src/services/analytics/index.ts`](../../src/services/analytics/index.ts)
- [`src/services/oauth/index.ts`](../../src/services/oauth/index.ts)
- [`src/services/oauth/client.ts`](../../src/services/oauth/client.ts)
- [`src/services/oauth/auth-code-listener.ts`](../../src/services/oauth/auth-code-listener.ts)
- [`src/services/remoteManagedSettings/index.ts`](../../src/services/remoteManagedSettings/index.ts)
- [`src/services/remoteManagedSettings/securityCheck.tsx`](../../src/services/remoteManagedSettings/securityCheck.tsx)

Important symbols:
- `GlobalConfig`
- `ProjectConfig`
- `HookCommandSchema`
- `HooksSchema`
- `isAnthropicAuthEnabled()`
- `getAuthTokenSource()`
- `isPolicyLimitsEligible()`
- `onGrowthBookRefresh()`
- `getOutputStyleDirStyles()`
- `isVoiceModeEnabled()`
- `OAuthService`
- `AuthCodeListener`
- `loadRemoteManagedSettings()`

## Core Types / Classes

Observed config layering:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


Observed auth model:
- first-party OAuth path
- direct API key path
- 3P provider paths
- managed OAuth contexts for remote/desktop
- secure storage and helper-based retrieval

Observed policy model:
- org-level restrictions fetched from API
- fail-open on fetch failure
- cached and background-polled

Observed analytics/gating model:
- GrowthBook remote evaluation
- local and env overrides
- refresh listeners for long-lived components

Observed hook/output-style model:


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Data Flow

```text
startup
-> read config layers
-> determine auth source
-> parse persisted hooks / output styles / feature-dependent UX visibility
-> fetch policy limits if eligible
-> initialize GrowthBook and experiment cache
-> expose gates to runtime decisions
```

## Pseudocode


> Source-like pseudocode and type sketches have been removed. The surrounding prose keeps only high-level architectural notes.


## Trust Boundaries / Failure Modes

- Config loading must avoid recursive dependency loops. The source has explicit re-entrancy guards.
- Managed OAuth contexts must not silently fall back to user terminal auth sources.
- Policy limits intentionally fail open if fetch fails, which avoids bricking the CLI on control-plane issues.
- GrowthBook refresh timing matters because long-lived objects may bake gate decisions at construction time.
- Hook schemas are persisted and round-tripped through settings. `src/schemas/hooks.ts` explicitly warns against transforms that would disappear during JSON serialization.
- Output styles are prompt-shaping inputs. Project-local markdown can override user-level styles, so precedence and trust scope matter.
- Voice visibility and voice usability are separate questions. [`src/voice/voiceModeEnabled.ts`](../../src/voice/voiceModeEnabled.ts) combines feature gating and Anthropic OAuth checks instead of treating UI visibility as proof of capability.

## Non-Obvious Implementation Choices

### 1. Auth source selection is environment-sensitive

`utils/auth.ts` contains nuanced logic to avoid cross-contaminating:
- managed remote sessions
- user terminal sessions
- 3P provider flows
- SSH/proxy flows

This is a lot more subtle than `if env var else oauth`.

### 2. Config and policy are different layers

Local config expresses user/project preferences. Policy limits express org restrictions. The product keeps them distinct.

### 3. Analytics also powers runtime behavior

GrowthBook is not just telemetry. It drives:
- feature availability
- refresh behavior
- long-lived runtime adaptation

### 4. Fail-open is a product stance

Policy limits and some feature fetches prefer degraded capability over total startup failure. That is a deliberate resilience tradeoff.

### 5. Hook schemas were extracted to break cycles on purpose

[`src/schemas/hooks.ts`](../../src/schemas/hooks.ts) exists because hook-related schema definitions were pulled out of settings types to break circular dependencies. The file comments also show a second design goal: preserve clean JSON round-tripping for persisted hook config.

### 6. Output styles are configuration-driven prompt shaping

[`src/outputStyles/loadOutputStylesDir.ts`](../../src/outputStyles/loadOutputStylesDir.ts) loads markdown from project and user `.claude/output-styles/` directories, extracts frontmatter, and builds runtime output-style configs. This is a quiet but powerful extension plane for response shape.

### 7. Voice mode is a compact example of auth plus kill-switch policy

[`src/voice/voiceModeEnabled.ts`](../../src/voice/voiceModeEnabled.ts) separates `isVoiceGrowthBookEnabled()` from `hasVoiceAuth()`. That is exactly the kind of subtle availability logic product teams often bury in ad hoc UI conditionals.

## OAuth PKCE Flow And Managed OAuth Provenance

The auth chapter needs to cover the concrete first-party OAuth execution path, not just say that one exists.

Observed behavior across [`src/services/oauth/index.ts`](../../src/services/oauth/index.ts), [`src/services/oauth/client.ts`](../../src/services/oauth/client.ts), and [`src/services/oauth/auth-code-listener.ts`](../../src/services/oauth/auth-code-listener.ts):
- `OAuthService` orchestrates a PKCE flow rather than a static token exchange
- the CLI tries a localhost callback listener via `AuthCodeListener`, but also supports manual/browser fallback flows when the listener cannot be used
- token exchange, refresh, and profile/account preservation are handled in a dedicated service layer rather than being scattered through `utils/auth.ts`
- this runtime is kept distinct from managed OAuth contexts, which is important because desktop/remote-managed sessions must not silently inherit the wrong credential provenance

That distinction is a real trust boundary in a multi-host product.

## Remote Managed Settings Lifecycle

Remote managed settings are also underdescribed if they only appear as “config”.

Observed behavior across [`src/services/remoteManagedSettings/index.ts`](../../src/services/remoteManagedSettings/index.ts) and [`src/services/remoteManagedSettings/securityCheck.tsx`](../../src/services/remoteManagedSettings/securityCheck.tsx):
- the subsystem has explicit eligibility checks before attempting network fetches
- startup initializes a loading promise so other subsystems can await managed-settings readiness without depending on who triggered the fetch
- load and refresh paths compare checksums, reuse cache when possible, and retry transient failures
- newly fetched managed settings can trigger a blocking security review before being written into active config
- long-lived sessions poll or refresh in the background instead of assuming startup-time state is final

That is a lifecycle subsystem with policy implications, not just a settings reader.

## Agent-Builder Takeaways

- Separate user preference, managed config, and org policy.
- Be explicit about auth provenance in multi-host systems.
- If feature delivery affects live runtime objects, add refresh notifications.
- Decide consciously where you want fail-open vs fail-closed behavior.
- Treat persisted hook DSLs and prompt-shaping config as first-class schema surfaces, not incidental settings blobs.

## Source Anchors

- [`src/utils/config.ts`](../../src/utils/config.ts)
- [`src/utils/auth.ts`](../../src/utils/auth.ts)
- [`src/schemas/hooks.ts`](../../src/schemas/hooks.ts)
- [`src/outputStyles/loadOutputStylesDir.ts`](../../src/outputStyles/loadOutputStylesDir.ts)
- [`src/voice/voiceModeEnabled.ts`](../../src/voice/voiceModeEnabled.ts)
- [`src/services/policyLimits/index.ts`](../../src/services/policyLimits/index.ts)
- [`src/services/analytics/growthbook.ts`](../../src/services/analytics/growthbook.ts)
- [`src/services/oauth/index.ts`](../../src/services/oauth/index.ts)
- [`src/services/oauth/client.ts`](../../src/services/oauth/client.ts)
- [`src/services/oauth/auth-code-listener.ts`](../../src/services/oauth/auth-code-listener.ts)
- [`src/services/remoteManagedSettings/index.ts`](../../src/services/remoteManagedSettings/index.ts)
- [`src/services/remoteManagedSettings/securityCheck.tsx`](../../src/services/remoteManagedSettings/securityCheck.tsx)

## [Inference]

The product likely needed to support both individual users and centrally managed enterprise environments, which is why config, policy, auth, and feature delivery are so intertwined.
