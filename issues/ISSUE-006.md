# ISSUE-006 — Svelte sorts token fragments across concatenation boundaries

State: PR-Ready
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

Audit reference: `E6`, retained by the self-review.
Root-Cause [S]: Svelte expression literals are sorted without protecting tokens attached to adjacent operands.
The caller at `src/index.ts:973-981` disables whitespace reduction but supplies no concatenation-edge protection.

## Reach-and-Impact

Reach [O]: Svelte class expressions that build a class from a prefix and a string operand are affected.
Impact [O]: Full formatting changes the attached class and introduces a different standalone class.
Severity: High.

## Evidence

- [S] `src/index.ts:977-980` passes no operand-boundary policy to `sortStringLiteral`.
- [S] `src/index.ts:1208-1222` writes the resulting source changes into the Svelte printer's original text.
- [O] Full formatting changes `"inline-" + "underline flex block"` to `"inline-" + "block flex underline"`.
- [S] Evaluation therefore replaces `"inline-underline"` with `"inline-block"` and activates standalone `"underline"`.

## Prior-Art

Coverage: The Glimmer/Svelte searches recorded in `ISSUE-005` were checked on 2026-10-21.
Relevant complete threads and the active Svelte diff were read.
Gaps: Global discussion, release, and forum coverage is not exhaustive.

- Distinct overlap: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/475 changes whitespace and offsets.
- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/issues/474 is owned by that active whitespace PR.
- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/332 protects Svelte source-length invariants.
- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/359 prevents length-changing deduplication.

Contribution fit: A separate length-preserving token-attachment PR addresses a different root cause from PR 475.
The overlapping handlers and existing active owner must be disclosed rather than treated as a duplicate.

## Proposed-Change

Sort Svelte string literals under `+` only when their class-token edges are established safely.
Preserve uncertain partial concatenations instead of sorting attached token fragments.

## Scope-and-Constraints

- Preserve sorting of standalone complete class strings.
- Preserve the value and token bindings of concatenated class expressions.
- Do not treat `collapseWhitespace: false` as token-boundary protection.
- Keep comparison traversal in `ISSUE-001` separate.

## Verification

- Format a prefix-concatenated Svelte class expression and compare its evaluated class tokens.
- Require the prefix to remain attached to the original token.

## Publication-Blockers

Publication requires approval of the exact Publication-Draft and recorded target.
The draft distinguishes token attachment from the overlapping whitespace/offset PR 475.
The user authorized this finding's PR implementation and a new GitHub PR on 2026-10-21.

## Next-Action

Summary: Approve exact pull request draft
Action: Review and approve the exact Publication-Draft and its upstream/base/head target.
Done-When: The user explicitly approves the unchanged full draft and exact target.

## Bug-Reproduction

Environment: Ubuntu 24.04.5, Node.js 26.12.0, Prettier 3.9.8, and Tailwind CSS 4.3.3.
The specialist also reproduced the path with `prettier-plugin-svelte` 4.1.1 and Svelte 5.57.2.
Procedure: Format with the original source plugin and parser `svelte` using the installed Svelte plugin.

```svelte
<div class={"inline-" + "underline flex block"}></div>
```

Observed full-pipeline output:

```svelte
<div class={"inline-" + "block flex underline"}></div>
```

Expected behavior: The original class-token identities remain unchanged.
The disposable source-plugin harness has been removed.

## Pull-Request-Implementation

Branch: fix/issue-006-preserve-svelte-class-fragments
Base: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`
Scope: Preserve class-token attachment across Svelte string concatenation boundaries.
Commit: b9745429c57ea750f0d6d4f6c9040982fb7f1aa9
Push: MikeeI:fix/issue-006-preserve-svelte-class-fragments
Checks:

- `pnpm run format --check` → passed.
- `pnpm run lint` → passed with existing Knip configuration hints.
- `pnpm run build` → passed.
- `env npm_config_userconfig=/dev/null pnpm run test --run` → 112 passed, four existing skips.

## Publication-Draft

Target: https://github.com/tailwindlabs/prettier-plugin-tailwindcss
Base: `main`
Head: `MikeeI:fix/issue-006-preserve-svelte-class-fragments`
Title: Preserve attached tokens in Svelte class concatenations
Approval: Pending.

```markdown
## Summary

Disabling whitespace collapse does not prevent sorting from moving an externally attached class token.
In `"inline-" + "underline flex block"`, the prefix can become attached to `block` instead of `underline`.
Accumulate concatenation edges through the complete Svelte expression path.
Protect attached first and last tokens in both string literals and template quasis.
Independent middle classes remain sortable.

Related work: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/475
This targets token attachment, not that PR's whitespace-collapse and source-offset changes.
Existing whitespace, deduplication, and source-length policies remain unchanged.

## Verification

- Added a built-plugin regression with outer concatenations, nested conditionals, and both literal kinds.
- Existing Svelte plugin-compatibility coverage passed after correcting unsafe historical expectations.
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
