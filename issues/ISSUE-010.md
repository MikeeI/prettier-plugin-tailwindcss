# ISSUE-010 — Relative sorter paths hide an installed Tailwind package

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: API
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`

## Root-Cause

Audit reference: `E10`, narrowed by the self-review to the observed installation-selection failure.
Root-Cause [S]: Relative `base` and `filepath` values reach package resolution without normalization.
The resolver error is swallowed and the installed package is treated as unavailable.
The owning boundaries are `src/sorter.ts:108-119,223-229`.

## Reach-and-Impact

Reach [S]: `createSorter` forwards public path options directly to `getTailwindConfig`.
Impact [O]: A relative lookup fails where the equivalent absolute lookup finds the installed Tailwind package.
Impact [S]: The swallowed failure selects a fallback instead of that local installation.
No differing v3 class-order output has been observed.
Severity: Medium.

## Evidence

- [S] `src/sorter.ts:11-40` describes a directory for relative paths without requiring an absolute `base`.
- [S] `src/sorter.ts:108-112` normalizes config and stylesheet paths but not the package lookup directory.
- [O] `resolveJsFrom('.', 'tailwindcss')` throws `Cannot read properties of undefined (reading 'length')`.
- [O] The absolute counterpart resolves `/root/projects/project-oh-my-pi-fork/node_modules/tailwindcss/dist/lib.mjs`.
- [O] Public configuration loading completes through the fallback after the relative resolver failure.
- [O] The simple sampled class order was identical for both paths; no output difference is inferred from it.

## Prior-Art

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until the public path contract and prior API reports are reviewed.

## Proposed-Change

Normalize `base` to an absolute path at the public configuration boundary.
Resolve relative `filepath` values against that base before deriving the package lookup directory.

## Scope-and-Constraints

- Preserve absolute-path behavior, monorepo lookup, and intentional missing-installation fallbacks.
- Do not claim that every fallback produces a different class order.
- Confirm relative-file semantics against the public API contract before implementation.

## Verification

- Compare relative and absolute public API calls within a project containing a local Tailwind installation.
- Require identical installation and configuration selection rather than only identical ordering of default classes.
- Use a distinguishable v3 configuration before claiming an observed class-order consequence.

## Publication-Blockers

Prior art and a minimal public reproduction repository remain missing.
A distinguishing full-sorter reproduction and relative-file contract review remain incomplete.
The user has not selected implementation work or approved publication.

## Next-Action

Summary: Reproduce public path selection difference
Action: Build a minimal external reproduction with distinguishable local Tailwind configuration and relative API paths.
Done-When: Installation selection is observable through the public API and any class-order claim is measured.

## Bug-Reproduction

Environment: Ubuntu 24.04.5, Node.js 26.12.0, enhanced-resolve 5.25.1, and Tailwind CSS 4.3.3.
The full source harness also used Prettier 3.9.8.
Working directory: `/root/projects/project-oh-my-pi-fork`.
Relative lookup: `resolveJsFrom('.', 'tailwindcss')` throws the error recorded above.
Absolute lookup: The same function with that absolute working directory resolves the installed ESM entrypoint.
Public calls with `base: '.'` and the absolute base both complete, but use different selection paths.
The disposable harness has been removed.

## API-and-Compatibility

`SorterOptions.base` owns resolution of relative file paths, and `filepath` owns package-lookup location.
The correction must not accidentally reinterpret already absolute paths or replace an intentional fallback policy.
