# ISSUE-004 — Vue trims separators required by outer concatenations

State: Submitted
Authorized-Work: Pull-Request-Implementation
Publication-Target: New-pull-request
External-Reference: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/481
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

Coverage: The combined searches and open list recorded in `ISSUE-003` were checked on 2026-10-21.
Relevant concatenation, interpolation, and escape threads were read.
Gaps: Global forum and discussion coverage is not exhaustive.

- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/396 establishes the shared boundary policy.
- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/427 changes part of the existing template heuristic.
- Distinct: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/issues/328 is the closed JSX predecessor.

Contribution fit: A new PR can migrate dynamic JS attributes to the existing concatenation policy.
No competing active implementation was identified.

## Proposed-Change

Use the common `canCollapseWhitespaceIn` boundary policy instead of the nearest-concatenation decision.
The nested-concatenation fix does not require changing the helper under `ISSUE-007`.

## Scope-and-Constraints

- Preserve necessary separators across enclosing concatenations without claiming universal interpolation correctness.
- Preserve existing sorting and optional whitespace preservation.
- Leave `ISSUE-007` unimplemented; its independent quasi-edge defect is not a prerequisite for this correction.

## Verification

- Format and evaluate a nested Vue concatenation before and after the correction.
- Require unchanged token identities in nested concatenations while preserving safe inner sorting.

## Publication-Blockers

None; the exact approved draft and target were published on 2026-10-21.
Scope reuses the existing boundary policy; the independent `ISSUE-007` defect remains unresolved.
The user authorized this finding's PR implementation and a new GitHub PR on 2026-10-21.

## Next-Action

Summary: Await upstream review
Action: Review maintainer feedback when upstream responds to the submitted pull request.
Done-When: Upstream merges, closes, or requests a concrete follow-up change.

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
Commit: a762ce905d22c992fbbec116e99f47bf7fcea237
Push: MikeeI:fix/issue-004-preserve-vue-class-separators
Checks:

- `pnpm run format --check` → passed.
- `pnpm run lint` → passed with existing Knip configuration hints.
- `pnpm run build` → passed.
- `env npm_config_userconfig=/dev/null pnpm run test --run` → 112 passed, four existing skips.

## Publication-Draft

Target: https://github.com/tailwindlabs/prettier-plugin-tailwindcss
Base: `main`
Head: `MikeeI:fix/issue-004-preserve-vue-class-separators`
Title: Preserve separators in nested Vue class concatenations
Approval: Approved on 2026-10-21; user selected `Alle sechs unverändert veröffentlichen`.

```markdown
## Summary

Dynamic Vue class attributes inspect only the nearest concatenation when deciding which edges to trim.
In `('inline ' + 'sm:p-0 p-0 ') + 'block'`, formatting removes the separator before `block`.
Use the existing `canCollapseWhitespaceIn` policy to accumulate enclosing concatenation boundaries.
Keep sorting the independent classes inside the middle operand.
Correct the existing fixture expectation that encoded the lost separator.

Related boundary policy: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/396
This does not change the shared helper's interpolation heuristic.

## Verification

- Added a built-plugin regression for nested Vue concatenation.
- Existing Vue parser and plugin-compatibility coverage passed.
- `pnpm run format --check`, `pnpm run lint`, and `pnpm run build` passed.
- `pnpm run test --run`: 112 passed, four existing skips.

Tested with Node.js 26.12.0, pnpm 10.28.2, and lockfile-pinned Prettier 3.9.6.
The `src/config.ts` change is only repository-formatter normalization of an existing signature.

### Disclosure

Investigated thoroughly with `openai-codex/gpt-6.1-sol` (medium reasoning effort), using [Oh My Pi](https://github.com/can1357/oh-my-pi) as the agent framework.
I thoroughly reviewed this contribution with `openai-codex/gpt-6.1-sol` (medium reasoning effort).

This report is not generic or unreviewed AI-generated output.
Its claims were checked against the cited evidence, and it includes the relevant detail intended to help maintainers resolve the issue.

If reports like this are not useful to the project, please let me know and I will refrain from submitting similar ones.
My intent is to help without wasting maintainer time or energy or discouraging their work.

Thank you for your work.
```
