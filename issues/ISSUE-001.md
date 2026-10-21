# ISSUE-001 — Class traversal rewrites comparison operands

State: Submitted
Authorized-Work: Pull-Request-Implementation
Publication-Target: New-pull-request
External-Reference: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/478
Contribution-Priority: High
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`

## Root-Cause

Audit reference: `E1`, retained by the self-review.
Root-Cause [S]: Class-expression traversal treats comparison operands as output class strings.
The owning JavaScript visitor is `src/index.ts:608-621`; analogous visitors affect other template languages.

## Reach-and-Impact

Reach [O]: Formatting reproduced the defect across JavaScript/JSX, Vue, Angular, Glimmer, and Svelte.
Reach [S]: Angular also visits comparison literals through `src/index.ts:38-52`.
Impact [O]: Formatting changes a true condition into a false condition and removes the selected classes.
Severity: High.

## Evidence

- [S] `src/index.ts:609-613` sorts every visited string literal without excluding comparison operands.
- [S] `src/index.ts:645-658` reaches dynamic class attributes and configured function arguments.
- [O] With `tailwindFunctions: ['clsx']`, `state === "flex block"` becomes `state === "block flex"`.
- [O] With `state = 'flex block'`, actual `clsx` evaluation changes from `"flex block"` to `""`.
- [S] The intended operation sorts emitted classes, not the data used to select them.

## Prior-Art

Coverage: Six repository-scoped combined issue/PR searches and all open PRs were checked on 2026-10-21.
The searches returned 14, 36, 44, 54, 40, and 5 results with `incomplete_results: false`.
Relevant threads, implementation links, and discussion 10452 were read.
Gaps: Global forum, release, and discussion coverage is not exhaustive.

- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/issues/426 reports condition-string mutation.
- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/155 introduced configurable calls and class keys.
- Distinct: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/440 changed visitor mechanics without this guard.

Contribution fit: A new narrowly scoped PR can preserve syntactic condition inputs without disabling class-result sorting.
No direct active implementation was identified in the checked open PRs.

## Proposed-Change

Exclude comparison and condition subtrees from class sorting while preserving traversal of class-result branches.
Apply the same semantic boundary to the affected language visitors.
Do not disable documented class-object key sorting as a substitute.

## Scope-and-Constraints

- Preserve sorting of actual conditional results and documented class-object keys.
- Exclude unrelated visitor refactoring and any general-purpose expression evaluator.
- Keep the independent key-collision defect in `ISSUE-002` separate.

## Verification

- Reproduce the conditional call through full formatting and evaluate it with actual `clsx`.
- Require unchanged condition behavior and activated class tokens, with the output classes still sorted.

## Publication-Blockers

None; the exact approved draft and target were published on 2026-10-21.
The user authorized this finding's PR implementation and a new GitHub PR on 2026-10-21.

## Next-Action

Summary: Await upstream review
Action: Review maintainer feedback when upstream responds to the submitted pull request.
Done-When: Upstream merges, closes, or requests a concrete follow-up change.

## Bug-Reproduction

Environment: Ubuntu 24.04.5, Node.js 26.12.0, Prettier 3.9.8, and Tailwind CSS 4.3.3.
Procedure: Format this input with the original source plugin, parser `babel`, and `tailwindFunctions: ['clsx']`.

```js
clsx(state === "flex block" && "flex block")
```

Observed output:

```js
clsx(state === "block flex" && "block flex");
```

Expected behavior: The comparison operand remains `"flex block"`, while the result string may be sorted.
The disposable harness used real dependencies with module-loading adaptations and has been removed.
That initial reproduction did not patch source or change production state.

## Pull-Request-Implementation

Branch: fix/issue-001-preserve-class-conditions
Base: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`
Scope: Preserve comparison and condition operands across the affected class-expression visitors.
Commit: 6c826fc9fafc1d04af841cacf1fda0a852222c95
Push: MikeeI:fix/issue-001-preserve-class-conditions
Checks:

- `pnpm run format --check` → passed.
- `pnpm run lint` → passed with existing Knip configuration hints.
- `pnpm run build` → passed.
- `env npm_config_userconfig=/dev/null pnpm run test --run` → 118 passed, four existing skips.

## Publication-Draft

Target: https://github.com/tailwindlabs/prettier-plugin-tailwindcss
Base: `main`
Head: `MikeeI:fix/issue-001-preserve-class-conditions`
Title: Preserve condition operands when sorting class expressions
Approval: Approved on 2026-10-21; user selected `Alle sechs unverändert veröffentlichen`.

```markdown
## Summary

Class sorting currently rewrites comparison/test inputs instead of only the class results they select.
For example, `clsx(status === 'underline flex' ? 'sm:p-0 p-0' : '')` changes the compared value.
Preserve syntactic condition subtrees across JavaScript/TypeScript, Vue, Angular, Glimmer, and Svelte.
Result strings remain sortable, and global configured-call, tag, and JSX entries respect the same boundary.

Related report: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/issues/426
This does not infer arbitrary helper semantics or claim to resolve every case in that report.

## Verification

- Added built-plugin regressions for six parser boundaries and nested global reentry.
- `pnpm run format --check`, `pnpm run lint`, and `pnpm run build` passed.
- `pnpm run test --run`: 118 passed, four existing skips.

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
