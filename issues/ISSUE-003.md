# ISSUE-003 — Angular class rewriting drops required string escapes

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

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until prior art and a minimal compiler-valid reproduction are recorded.

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

Prior art and a minimal public reproduction repository remain missing.
A real Angular compiler check has not been performed.
The user authorized this finding's PR implementation and a new GitHub PR on 2026-10-21.
The exact external draft and target still require review before publication.

## Next-Action

Summary: Search Angular escape loss prior art
Action: Search upstream work for cooked Angular class strings losing quote escapes.
Done-When: Relevant reports are classified and the remaining compiler reproduction gap is recorded.

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
Commit: Pending.
Push: Pending.
Checks:

- Implementation and focused regression verification are pending.
