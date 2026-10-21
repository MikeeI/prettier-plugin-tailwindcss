# ISSUE-003 — Angular class rewriting drops required string escapes

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

Audit reference: `E3`, retained by the self-review.
Root-Cause [S]: The Angular visitor inserts cooked `node.value` into a raw string span without encoding it.
The replacement is assembled at `src/index.ts:38-52` and applied at `src/index.ts:89`.

## Reach-and-Impact

Reach [O]: Angular dynamic class bindings containing escaped quotes are affected.
Impact [O]: Full formatting removes required escapes and changes the binding's lexical structure.
Impact [S]: Embedded apostrophes can terminate the surrounding string instead of remaining string content.
Severity: High.

## Evidence

- [S] `src/index.ts:45-48` retains the original string boundaries while replacing their content with cooked text.
- [O] The escaped apostrophes in `before:content-[\'x\']` lose their backslashes during full Angular formatting.
- [O] The defect occurs even when no class changes position.
- [O] The permissive directive parser accepts a different structure after the corruption.

## Prior-Art

Coverage: Combined issue/PR searches returned 98 and 20 results, both complete, on 2026-10-21.
The open list contained 18 entries; relevant bodies, comments, and reviews were read.
Gaps: Global forum and discussion coverage is not exhaustive.

- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/461 fixes raw escaping in JavaScript/Vue.
- Distinct: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/issues/460 is the resolved Vue escaping report.
- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/377 adds Angular template-literal support.

Contribution fit: A local Angular cooked-to-raw encoding correction is distinct from those merged changes.
No competing active Angular string-encoding implementation was identified.

## Proposed-Change

Encode the sorted cooked Angular string for its original quote before writing it back into source.
Keep this encoding policy at the Angular raw-source boundary.

## Scope-and-Constraints

- Preserve string content, quote boundaries, necessary escapes, and class sorting.
- Do not replace the JavaScript raw-string policy or change unrelated JSX entity handling.
- Do not treat successful permissive directive parsing as proof of unchanged binding semantics.

## Verification

- Format the escaped binding and parse it again with a suitable Angular binding parser.
- Require the original string value and expression structure to survive formatting.

## Publication-Blockers

Publication requires approval of the exact Publication-Draft and recorded target.
The real `__ng_binding` round trip passed; no Angular compiler run is claimed.
The user authorized this finding's PR implementation and a new GitHub PR on 2026-10-21.

## Next-Action

Summary: Approve exact pull request draft
Action: Review and approve the exact Publication-Draft and its upstream/base/head target.
Done-When: The user explicitly approves the unchanged full draft and exact target.

## Bug-Reproduction

Environment: Ubuntu 24.04.5, Node.js 26.12.0, Prettier 3.9.8, and Tailwind CSS 4.3.3.
Procedure: Format the input with the original source plugin and parser `angular`.

```html
<div [ngClass]="'flex before:content-[\'x\']'"></div>
```

Observed output:

```html
<div [ngClass]="'flex before:content-['x']'"></div>
```

Expected behavior: Embedded quotes remain escaped and represent the original class string.
The disposable source-plugin harness has been removed.
No Angular compiler failure is claimed as an observed result.

## Pull-Request-Implementation

Branch: fix/issue-003-preserve-angular-escapes
Base: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`
Scope: Preserve Angular string-literal escape semantics when class bindings are rewritten.
Commit: d8f9a93f6e1df182343f5d9781f5884f3d496d15
Push: MikeeI:fix/issue-003-preserve-angular-escapes
Checks:

- `pnpm run format --check` → passed.
- `pnpm run lint` → passed with existing Knip configuration hints.
- `pnpm run build` → passed.
- `env npm_config_userconfig=/dev/null pnpm run test --run` → 112 passed, four existing skips.
- Real `__ng_binding` round trip → preserved `+`, embedded quotes, `U+0008`, and literal backslashes.

## Publication-Draft

Target: https://github.com/tailwindlabs/prettier-plugin-tailwindcss
Base: `main`
Head: `MikeeI:fix/issue-003-preserve-angular-escapes`
Title: Preserve string escapes in Angular class bindings
Approval: Pending.

```markdown
## Summary

Angular binding literals provide cooked values, but their replacement spans address raw source.
Escaped apostrophes in `before:content-[\'x\']` currently lose their backslashes during formatting.
Encode sorted cooked values for the original quote before writing them back.
Use Unicode control escapes so Angular does not reinterpret JavaScript-only escape forms.
Cooked utilities such as `\u0070-0` still participate in class sorting.

Related JavaScript/Vue work: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/461
This correction stays at Angular's separate cooked-to-raw boundary.

## Verification

- Added a built-plugin regression covering escaped quotes, cooked utility sorting, and idempotency.
- A real `__ng_binding` round trip preserved expression structure, quotes, control values, and backslashes.
- `pnpm run format --check`, `pnpm run lint`, and `pnpm run build` passed.
- `pnpm run test --run`: 112 passed, four existing skips.

Tested with Node.js 26.12.0, pnpm 10.28.2, and lockfile-pinned Prettier 3.9.6.
No Angular compiler run is claimed.
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
