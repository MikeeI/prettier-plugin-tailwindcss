# ISSUE-004 — Vue trims separators required by outer concatenations

State: Investigating
Authorized-Work: Pull-Request-Implementation
Publication-Target: New-pull-request
External-Reference: Not published.
Contribution-Priority: High
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`

## Root-Cause

Audit reference: `E4`, retained by the self-review.
Root-Cause [S]: The dynamic JavaScript attribute visitor considers only the nearest concatenation ancestor.
Outer concatenations and interpolation boundaries do not participate in its trim decision.
The owning paths are `src/index.ts:102-105,130-138,170-178`.

## Reach-and-Impact

Reach [O]: Vue `:class` expressions with nested concatenations or interpolated strings are affected.
Impact [O]: Formatting removes a necessary separator and merges previously distinct class tokens.
Severity: High.

## Evidence

- [S] `findConcatEntry` selects one `BinaryExpression` ancestor rather than accumulating enclosing boundaries.
- [O] Full Vue formatting changes `('inline ' + 'flex ') + 'block'` to `'inline ' + 'flex' + 'block'`.
- [O] The evaluated class value changes from `"inline flex block"` to `"inline flexblock"`.
- [O] An interpolated expression also loses its leading separator in the full Vue pipeline.

## Prior-Art

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until prior art identifies whether an existing thread owns nested expression trimming.

## Proposed-Change

Use the common `canCollapseWhitespaceIn` boundary policy instead of the nearest-concatenation decision.
Correct that helper under `ISSUE-007` before relying on it for this visitor.

## Scope-and-Constraints

- Preserve necessary separators across all enclosing concatenations and interpolations.
- Preserve existing sorting and optional whitespace preservation.
- Keep the helper's wrong-quasi-edge defect in `ISSUE-007` as a distinct root cause and prerequisite.

## Verification

- Format and evaluate a nested Vue concatenation before and after the correction.
- Require identical class tokens for the nested concatenation and the interpolation boundary scenario.

## Publication-Blockers

Prior art and a minimal public reproduction repository remain missing.
The proposed dependency on `ISSUE-007` must be resolved without silently expanding implementation scope.
The user authorized this finding's PR implementation and a new GitHub PR on 2026-10-21.
The exact external draft and target still require review before publication.

## Next-Action

Summary: Search Vue concatenation prior art
Action: Search upstream work for Vue class expressions losing separators in nested concatenations.
Done-When: Relevant reports are classified and the relationship to `ISSUE-007` is documented.

## Bug-Reproduction

Environment: Ubuntu 24.04.5, Node.js 26.12.0, Prettier 3.9.8, and Tailwind CSS 4.3.3.
Procedure: Format with the original source plugin and parser `vue`.

```html
<div :class="('inline ' + 'flex ') + 'block'"></div>
```

Observed output:

```html
<div :class="'inline ' + 'flex' + 'block'"></div>
```

Expected behavior: The resulting class value remains `"inline flex block"`.
The disposable source-plugin harness has been removed.

## Pull-Request-Implementation

Branch: fix/issue-004-preserve-vue-class-separators
Base: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`
Scope: Preserve necessary separators in nested dynamic Vue class expressions.
Commit: Pending.
Push: Pending.
Checks:

- Implementation and focused regression verification are pending.
