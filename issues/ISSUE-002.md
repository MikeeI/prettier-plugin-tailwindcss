# ISSUE-002 — Normalized object keys overwrite independent class conditions

State: Submitted
Authorized-Work: Pull-Request-Implementation
Publication-Target: New-pull-request
External-Reference: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/479
Contribution-Priority: High
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`

## Root-Cause

Audit reference: `E2`, expanded by the self-review to cover unknown object composition.
Root-Cause [S]: Independent class-key normalization can merge or split JavaScript property-identity groups.
The visitor has no sibling-key collision check at `src/index.ts:608-613`.

## Reach-and-Impact

Reach [O]: Configured `clsx` calls containing distinct keys with the same sorted classes are affected.
Impact [O]: A later false property overwrites an earlier true property and removes the activated classes.
Severity: High.

## Evidence

- [S] `src/index.ts:658` visits configured call arguments, including object keys.
- [O] Full formatting makes both keys in `{"flex block": true, "block flex": false}` equal to `"block flex"`.
- [O] Actual `clsx` evaluation changes from `"flex block"` to `""` after this transformation.
- [O] Additional real `clsx` evaluations confirm the same collision with spread and computed properties.
- [S] A check limited to explicitly written static sibling keys cannot establish the complete runtime key set.
- [O] A built-plugin regression reproduced Unicode-escaped duplicates splitting into distinct emitted keys.

## Prior-Art

Coverage: The same six combined issue/PR searches recorded in `ISSUE-001` and all open PRs were checked.
Relevant class-object, raw-escaping, and cross-string sorting threads were read on 2026-10-21.
Gaps: Global forum, release, and discussion coverage is not exhaustive.

- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/155 establishes class-key sorting support.
- Related: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/461 preserves raw string escapes.
- Distinct: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/issues/422 requests sorting across separate strings.

Contribution fit: A new collision-preservation PR is distinct from the checked work.
No direct active key-collision implementation was identified.

## Proposed-Change

Plan actual serialized identities before mutation and preserve groups that would merge or split.
Allow existing duplicate keys to move together only when their identity cohort stays intact.
Treat spread/computed composition conservatively without evaluating expressions.
Preserve sorting for collision-free supported static keys and keep reentry whitespace policy stable.

## Scope-and-Constraints

- Preserve property order, property values, evaluation order, and independent activation conditions.
- Do not evaluate dynamic keys or spread expressions during formatting.
- Keep traversal of comparison expressions in `ISSUE-001` separate.

## Verification

- Format a static collision and evaluate the result with actual `clsx`.
- Check spread and computed collisions for unchanged activated classes, alongside a sortable collision-free object.

## Publication-Blockers

None; the exact approved draft and target were published on 2026-10-21.
The user authorized this finding's PR implementation and a new GitHub PR on 2026-10-21.

## Next-Action

Summary: Await upstream review
Action: Review maintainer feedback when upstream responds to the submitted pull request.
Done-When: Upstream merges, closes, or requests a concrete follow-up change.

## Bug-Reproduction

Environment: Ubuntu 24.04.5, Node.js 26.12.0, Prettier 3.9.8, and Tailwind CSS 4.3.3.
Procedure: Format with parser `babel` and `tailwindFunctions: ['clsx']`.

```js
clsx({"flex block": true, "block flex": false})
```

Observed output:

```js
clsx({ "block flex": true, "block flex": false });
```

Expected behavior: Formatting preserves the active class-token set.
The self-review additionally evaluated these object forms with real `clsx`:

```js
const flags = { "block flex": false }
clsx({ "flex block": true, ...flags })
const key = "block flex"
clsx({ "flex block": true, [key]: false })
```

Replacing only the first key with `"block flex"` changes both results to `""`.
Those additional checks evaluate the collision semantics, not a second full formatting pipeline.
The original disposable formatting harness has been removed.

## Pull-Request-Implementation

Branch: fix/issue-002-preserve-object-key-identity
Base: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`
Scope: Preserve independent class-object property identities, including unknown spread and computed keys.
Commit: 60b1ad8ab15a1af489d87e197571427aeafdd94b
Push: MikeeI:fix/issue-002-preserve-object-key-identity
Checks:

- `pnpm run format --check` → passed.
- `pnpm run lint` → passed with existing Knip configuration hints.
- `pnpm run build` → passed.
- `env npm_config_userconfig=/dev/null pnpm run test --run` → 120 passed, four existing skips.
- Actual `clsx` round trips → unchanged static-collision, nested-spread, and escaped-duplicate results.

## Publication-Draft

Target: https://github.com/tailwindlabs/prettier-plugin-tailwindcss
Base: `main`
Head: `MikeeI:fix/issue-002-preserve-object-key-identity`
Title: Preserve class-object property identities during sorting
Approval: Approved on 2026-10-21; user selected `Alle sechs unverändert veröffentlichen`.

```markdown
## Summary

Sorting can merge distinct conditional keys or split duplicate identities, changing which property wins.
For example, `clsx({ 'underline flex': true, 'flex underline': false })` loses its active classes.
Plan actual serialized key identities before mutation and preserve unsafe identity groups.
Keep collision-free supported static keys sortable, including duplicate cohorts that move together safely.
Conservatively preserve keys under spread/computed composition without evaluating expressions.
Carry full key context through global sorter reentry so identity plans and writes agree on whitespace.

Previous class-key support: https://github.com/tailwindlabs/prettier-plugin-tailwindcss/pull/155

## Verification

- Added parser-family regressions for collisions, raw escapes, unknown composition, and global reentry.
- Real `clsx` evaluation preserved static-collision, nested-spread, and escaped-duplicate results.
- `pnpm run format --check`, `pnpm run lint`, and `pnpm run build` passed.
- `pnpm run test --run`: 120 passed, four existing skips.

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
