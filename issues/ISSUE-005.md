# ISSUE-005 — Glimmer class composition loses separators and token bindings

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

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until prior art and Glimmer composition compatibility are established.

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

Prior art and a minimal public reproduction repository remain missing.
The user authorized this finding's PR implementation and a new GitHub PR on 2026-10-21.
The exact external draft and target still require review before publication.

## Next-Action

Summary: Search Glimmer composition prior art
Action: Search upstream work for separators or prefix bindings lost in Glimmer class composition.
Done-When: Relevant matches are classified and the owning thread or evidence gap is recorded.

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
Commit: Pending.
Push: Pending.
Checks:

- Implementation and focused regression verification are pending.
