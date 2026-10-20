# ISSUE-009 — Stateful regex matching skips eligible attributes and calls

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`

## Root-Cause

Audit reference: `E9`, retained by the self-review.
Root-Cause [S]: Reused global or sticky regex objects carry `lastIndex` across independent name checks.
The matcher at `src/options.ts:166-185` preserves flags and calls `regex.test(name)` without resetting state.

## Reach-and-Impact

Reach [O]: Regex entries in `tailwindAttributes` and `tailwindFunctions` can be affected.
Impact [O]: Eligible class lists are skipped according to earlier matching attempts in the same traversal.
Severity: Medium.

## Evidence

- [S] `src/options.ts:182-185` accepts regex flags, including `g` and `y`.
- [O] Repeated checks of `data-classes` with `/data-.*/g` return `true`, `false`, `true`.
- [O] The sticky variant `/data-.*/y` produces the same alternating results for the identical name.
- [O] Full formatting sorts the first and third matching JSX attributes but leaves the second unsorted.
- [O] Three configured calls with `/clsx/g` reproduce the skipped middle result.

## Prior-Art

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until existing regex-option reports and fixes are checked.

## Proposed-Change

Reset `lastIndex` before each independent name test.
Preserve accepted flags without allowing prior names to change match eligibility.

## Scope-and-Constraints

- Preserve ordinary regex matching, literal-name matching, and supported option syntax.
- Do not reject previously accepted flags merely to hide mutable regex state.
- Keep one finding for the shared predicate used by attributes and functions.

## Verification

- Repeatedly test identical eligible names through the same matcher instance.
- Format several matching attributes or calls and require every eligible class list to be sorted.

## Publication-Blockers

Prior art and a minimal public reproduction repository remain missing.
The user has not selected implementation work or approved an external target and draft.

## Next-Action

Summary: Search stateful matcher prior art
Action: Search upstream work for global or sticky option regexes skipping matching names.
Done-When: Relevant matches are classified and the owning thread or evidence gap is recorded.

## Bug-Reproduction

Environment: Ubuntu 24.04.5, Node.js 26.12.0, Prettier 3.9.8, and Tailwind CSS 4.3.3.
Procedure: Format with parser `babel` and `tailwindAttributes: ['/data-.*/g']`.

```jsx
<div data-x="p-4 m-2" data-y="p-4 m-2" data-z="p-4 m-2" />
```

Observed output:

```jsx
<div data-x="m-2 p-4" data-y="p-4 m-2" data-z="m-2 p-4" />;
```

Expected behavior: All three eligible attributes are sorted independently.
The disposable source-plugin harness has been removed.
