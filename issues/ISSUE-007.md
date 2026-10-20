# ISSUE-007 — Template whitespace checks inspect the wrong quasi edges

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

Audit reference: `E7`, retained by the self-review with JavaScript and Angular scope.
Root-Cause [S]: `canCollapseWhitespaceIn` tests the wrong ends of quasis adjacent to interpolated expressions.
Its positional comparisons can also include a quasi from the wrong side through a negative difference.
The owning code is `src/index.ts:579-592`.

## Reach-and-Impact

Reach [O]: JavaScript configured calls and Angular class bindings with interpolated class strings are affected.
Impact [O]: Full formatting removes a necessary separator between a static prefix and an expression result.
Severity: Medium.

## Evidence

- [S] `src/index.ts:587-592` tests a quasi's beginning for the expression's leading trim decision.
- [S] `src/index.ts:42,58,610,668` shares this helper across Angular and JavaScript transformation paths.
- [O] Actual `clsx` evaluation changes from `" prefix flex block "` to `" prefixblock flex "`.
- [O] Full Angular formatting reproduces the same missing separator.

## Prior-Art

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until prior art identifies existing template-boundary work.

## Proposed-Change

Identify adjacent quasis by expression index rather than positional difference heuristics.
For leading expression whitespace, inspect the end of the previous quasi.
For trailing expression whitespace, inspect the beginning of the next quasi.

## Scope-and-Constraints

- Preserve necessary separators for all existing helper callers.
- Exclude nonadjacent quasis from each boundary decision.
- Establish this correction before the Vue visitor migration proposed in `ISSUE-004`.
- Keep the whitespace-character-domain defect in `ISSUE-008` separate.

## Verification

- Format and evaluate interpolated class strings through JavaScript and Angular paths.
- Require the static prefix and the first interpolated class to remain distinct tokens.

## Publication-Blockers

Prior art and a minimal public reproduction repository remain missing.
All existing helper callers require focused compatibility review before implementation.
The user has not selected implementation work or approved an external draft and target.

## Next-Action

Summary: Search template edge checking prior art
Action: Search upstream work for incorrect adjacent-quasi selection in class-expression whitespace trimming.
Done-When: Relevant reports are classified and the prerequisite relationship to `ISSUE-004` is recorded.

## Bug-Reproduction

Environment: Ubuntu 24.04.5, Node.js 26.12.0, Prettier 3.9.8, and Tailwind CSS 4.3.3.
Procedure: Format with parser `babel` and `tailwindFunctions: ['clsx']`.

```js
clsx(` prefix${" flex block"} `)
```

Observed output:

```js
clsx(` prefix${"block flex"} `);
```

Expected behavior: Sorting preserves the separator immediately before the first interpolated class.
The disposable source-plugin harness has been removed.
