# ISSUE-005 — Glimmer class composition loses separators and token bindings

State: Submitted
Authorized-Work: Pull-Request-Implementation
Publication-Target: New-pull-request
External-Reference: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/482
Contribution-Priority: High
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`

## Root-Cause

Audit reference: `E5`, retained by the self-review.
Root-Cause [S]: Glimmer string-literal handling does not preserve both composition boundaries.
It recognizes `concat` only as `SubExpression` and lacks protection for an attached first token.
The owning visitor is `src/index.ts:282-302`.

## Reach-and-Impact

Reach [O]: Dynamic Glimmer class values using `concat` or conditional string fragments are affected.
Impact [O]: Full formatting merges classes or attaches a prefix to a different token.
Severity: High.

## Evidence

- [S] `src/index.ts:287-301` misses `MustacheStatement` concatenations and conditionally trims string ends.
- [O] Full formatting removes the separator from `class={{concat "flex " "block"}}`.
- [O] The resulting composed value is `"flexblock"` instead of `"flex block"`.
- [O] Nested `concat` moves `"underline"` away from `"inline-"`, producing `"inline-block"` instead.
- [O] Whitespace preservation does not prevent the observed nested prefix-token movement.

## Prior-Art

Coverage: Combined issue/PR searches covered Glimmer, Handlebars, concat, Svelte, and partial-token terms.
All three Glimmer and 62 Svelte results were retrieved with `incomplete_results: false` on 2026-10-21.
Relevant bodies, threads, reviews, and diffs were read.
Gaps: Global discussion, release, and forum coverage is not exhaustive.

- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/119 introduces Glimmer concat edge protection.
- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/396 preserves nested-expression boundaries.

Contribution fit: A new Glimmer PR extends the existing concat policy to both edges and AST forms.
No identical active implementation was identified.

## Proposed-Change

Handle both `MustacheStatement` and `SubExpression` forms of `concat` and protect both attached token edges.
Leave string fragments unchanged when their class boundaries cannot be established safely.

## Scope-and-Constraints

- Preserve class-result sorting when boundaries are known.
- Preserve separators and token identity in concatenated and conditional fragments.
- Keep comparison-operand traversal in `ISSUE-001` separate.
- Do not introduce a general template evaluator.

## Verification

- Format representative `concat` values and compare their composed class tokens before and after formatting.
- Include a prefix-attached fragment so whitespace-only preservation cannot hide token movement.

## Publication-Blockers

None; the exact approved draft and target were published on 2026-10-21.
The user authorized this finding's PR implementation and a new GitHub PR on 2026-10-21.

## Next-Action

Summary: Await upstream review
Action: Review maintainer feedback when upstream responds to the submitted pull request.
Done-When: Upstream merges, closes, or requests a concrete follow-up change.

## Bug-Reproduction

Environment: Ubuntu 24.04.5, Node.js 26.12.0, Prettier 3.9.8, and Tailwind CSS 4.3.3.
Procedure: Format with the original source plugin and parser `glimmer`.

```hbs
<div class={{concat "flex " "block"}}></div>
```

Observed output:

```hbs
<div class={{concat "flex" "block"}}></div>
```

Expected behavior: Composition retains the separator between the two classes.
A second observed input exposes prefix movement:

```hbs
<div class={{if ok (concat "inline-" "underline flex block ") ""}}></div>
```

The sorted fragment becomes `"block flex underline "` and changes the attached class identity.
The disposable source-plugin harness has been removed.

## Pull-Request-Implementation

Branch: fix/issue-005-preserve-glimmer-class-boundaries
Base: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`
Scope: Preserve separators and attached class-token fragments in Glimmer composition.
Commit: 52be5ac5bc82c8c44d84c16ca06425108cf1c39e
Push: MikeeI:fix/issue-005-preserve-glimmer-class-boundaries
Checks:

- `pnpm run format --check` → passed.
- `pnpm run lint` → passed with existing Knip configuration hints.
- `pnpm run build` → passed.
- `env npm_config_userconfig=/dev/null pnpm run test --run` → 114 passed, four existing skips.

## Publication-Draft

Target: https://github.com/tailwindlabs/prettier-plugin-tailwindcss
Base: `main`
Head: `MikeeI:fix/issue-005-preserve-glimmer-class-boundaries`
Title: Preserve token boundaries in Glimmer class composition
Approval: Approved on 2026-10-21; user selected `Alle sechs unverändert veröffentlichen`.

```markdown
## Summary

Glimmer class fragments can lose separators or move a token attached to another expression.
For example, `class={{concat "flex " "block"}}` becomes a different class value after trimming.
Handle both `MustacheStatement` and `SubExpression` forms of `concat`.
Track enclosing composition on both edges and preserve externally attached first or last tokens.
Known literal separators still allow independent inner classes to sort.

Previous concat handling: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/119
This extends that boundary policy without introducing a template evaluator.

## Verification

- Added built-plugin regressions for concat separators, nested prefix attachment, and conditional fragments.
- Existing Glimmer parser and plugin-compatibility coverage passed.
- `pnpm run format --check`, `pnpm run lint`, and `pnpm run build` passed.
- `pnpm run test --run`: 114 passed, four existing skips.

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
