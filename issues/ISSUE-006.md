# ISSUE-006 — Svelte sorts token fragments across concatenation boundaries

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

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until prior art identifies the owning Svelte expression-boundary work.

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

Prior art and a minimal public reproduction repository remain missing.
The user authorized this finding's PR implementation and a new GitHub PR on 2026-10-21.
The exact external draft and target still require review before publication.

## Next-Action

Summary: Search Svelte token boundary prior art
Action: Search upstream work for sorting of partial string operands in Svelte class concatenations.
Done-When: Relevant matches are classified and an owning thread or evidence gap is recorded.

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
Commit: Pending.
Push: Pending.
Checks:

- Implementation and focused regression verification are pending.
