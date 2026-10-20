# ISSUE-001 — Class traversal rewrites comparison operands

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

Audit reference: `E1`, retained by the self-review.
Root-Cause [S]: Class-expression traversal treats comparison operands as output class strings.
The owning JavaScript visitor is `src/index.ts:608-621`; analogous visitors affect other template languages.

## Reach-and-Impact

Reach [O]: Full source-plugin formatting reproduced the defect in JavaScript, JSX, Vue, Glimmer, and Svelte.
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

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until prior art identifies the owning thread and an appropriate contribution.

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

Prior art, a minimal public reproduction repository, and an exact publication target remain unresolved.
The user authorized this finding's PR implementation and a new GitHub PR on 2026-10-21.
The exact external draft and target still require review before publication.

## Next-Action

Summary: Search comparison traversal prior art
Action: Search upstream work for comparison-literal rewriting in class expressions.
Done-When: Relevant matches are classified by root cause and an owning thread or evidence gap is recorded.

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
No source patch or production-state change was performed.

## Pull-Request-Implementation

Branch: fix/issue-001-preserve-class-conditions
Base: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`
Scope: Preserve comparison and condition operands across the affected class-expression visitors.
Commit: Pending.
Push: Pending.
Checks:

- Implementation and focused regression verification are pending.
