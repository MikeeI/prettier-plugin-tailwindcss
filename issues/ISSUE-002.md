# ISSUE-002 — Normalized object keys overwrite independent class conditions

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: High
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`

## Root-Cause

Audit reference: `E2`, expanded by the self-review to cover unknown object composition.
Root-Cause [S]: Independently normalizing class-object keys can merge distinct JavaScript property identities.
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

## Prior-Art

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until prior art and compatibility requirements are established.

## Proposed-Change

For objects with fully known keys, determine normalized keys before mutation and preserve colliding groups.
Leave the object's keys unchanged when spread or computed keys prevent proving the final key set.
Preserve documented sorting for collision-free objects with fully known keys.

## Scope-and-Constraints

- Preserve property order, property values, evaluation order, and independent activation conditions.
- Do not evaluate dynamic keys or spread expressions during formatting.
- Keep traversal of comparison expressions in `ISSUE-001` separate.

## Verification

- Format a static collision and evaluate the result with actual `clsx`.
- Check spread and computed collisions for unchanged activated classes, alongside a sortable collision-free object.

## Publication-Blockers

Prior art, a minimal public reproduction repository, and compatibility review remain incomplete.
The user has not selected implementation work or approved an external target and draft.

## Next-Action

Summary: Search object key collision prior art
Action: Search upstream work for normalized class-object keys that overwrite independent conditions.
Done-When: Static and dynamic collision reports are classified and the owning thread or gap is recorded.

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
