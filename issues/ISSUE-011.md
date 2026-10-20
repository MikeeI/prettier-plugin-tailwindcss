# ISSUE-011 — Config discovery permanently caches stale paths and misses

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Reliability
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`

## Root-Cause

Audit reference: `E11`, narrowed by the self-review to observed discovery staleness without a promised refresh deadline.
Root-Cause [S]: `findClosestJsConfig` stores positive paths and negative results in an unbounded-lifetime `Map`.
The owning code at `src/sorter.ts:253-278` never rechecks a cached lookup directory.

## Reach-and-Impact

Reach [S]: Repeated automatic v3-config discovery in long-lived formatter processes is affected.
Impact [O]: A newly created configuration remains invisible to the existing finder cache.
Impact [S]: A cached removed path can also keep selecting a nonexistent configuration.
No full v3 sorting change or removed-path load failure was reproduced.
Severity: Medium.

## Evidence

- [S] `src/sorter.ts:257-259` returns a cached path or `null` before inspecting the filesystem again.
- [S] The nearby API and package caches expire after 10,000 ms, unlike `configPathCache`.
- [O] The original finder initially returns `null` for a real temporary project directory.
- [O] After creating valid `tailwind.config.js` and waiting over 11 seconds, that instance still returns `null`.
- [O] A fresh instance on the same filesystem returns the newly created configuration's exact path.
- [S] Nearby cache durations do not establish a public ten-second refresh guarantee.

## Prior-Art

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until existing editor lifecycle and cache-invalidation reports are reviewed.

## Proposed-Change

Bound the lifetime of both positive and negative config-discovery results with the existing `expiringMap`.
Do not present the chosen duration as an existing public service-level guarantee.

## Scope-and-Constraints

- Preserve automatic nearest-config lookup and its file-name precedence.
- Do not confuse changes inside an unchanged config path with discovery of a different path.
- Keep the discovery observation separate from unexecuted full-v3 formatting consequences.

## Verification

- In one process, perform discovery, create a valid config, and wait for the chosen cache duration.
- Require the original process to discover the new path after expiration.
- Verify moved or removed positive paths separately before claiming their load-error behavior as observed.

## Publication-Blockers

Prior art and a minimal public reproduction repository remain missing.
Full v3 formatting impact and the editor lifecycle contract remain to be verified.
The user has not selected implementation work or approved an external target and draft.

## Next-Action

Summary: Search config discovery lifecycle prior art
Action: Search upstream work for editor cache invalidation after creating or moving v3 configuration files.
Done-When: Lifecycle expectations and matching reports are classified without inventing a refresh guarantee.

## Bug-Reproduction

Environment: Ubuntu 24.04.5, Node.js 26.12.0, and escalade 3.2.0.
The original function was exposed by temporary export instrumentation without altering its implementation.
The real filesystem fixture initially contained only an empty `src` directory.
The old finder returned `null` before configuration creation and after more than 11 seconds following creation.
A fresh module instance returned `/tmp/tailwind-semantic-audit.oHLmvN/discovery-project/tailwind.config.js`.
The valid temporary config exported a theme extension containing the color `audit: '#123456'`.
Both the fixture and instrumentation were removed after verification.
No Tailwind-v3 runtime or sorting result is claimed as observed.
