# ISSUE-008 — Whitespace domains disagree about valid class-token boundaries

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`

## Root-Cause

Audit reference: `E8`, retained by the self-review.
Root-Cause [S]: Token splitting uses HTML ASCII whitespace, while trimming and interpolation checks use JavaScript `\s`.
These sets disagree on characters such as U+00A0, which can occur inside an HTML class token.
The owning boundaries include `src/sorting.ts:48,83-89` and `src/index.ts:478-499`.

## Reach-and-Impact

Reach [O]: HTML, JSX, tagged templates, and the public sorter API reproduce the mismatch.
Impact [O]: Formatting removes a character from a class name and produces non-idempotent results.
Impact [O]: Template formatting also attaches an interpolated suffix to a different token.
Severity: Medium.

## Evidence

- [S] `src/sorting.ts:48` splits using `[\t\r\f\n ]`, not the broader JavaScript whitespace set.
- [O] A class string beginning with U+00A0 becomes `"p-4 m-2"` on the first pass.
- [O] A second pass produces `"m-2 p-4"`; HTML, JSX, and `sortClassAttributes` agree.
- [O] U+00A0 before `${suffix}` changes the token binding even with `tailwindPreserveWhitespace: true`.
- [S] Removing U+00A0 changes an unknown token into the valid utility `p-4`.

## Prior-Art

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until prior art distinguishes HTML class-token rules from JavaScript whitespace rules.

## Proposed-Change

Use the same ASCII whitespace set for splitting, trimming, and interpolation-boundary protection.
Do not broaden token splitting to `\s`, which would also alter valid HTML class names.

## Scope-and-Constraints

- Preserve non-ASCII characters inside class tokens.
- Preserve ordinary whitespace normalization and the whitespace-preservation option.
- Keep incorrect adjacent-quasi selection in `ISSUE-007` separate.

## Verification

- Repeatedly sort a class string containing U+00A0 and require unchanged token identity and stable output.
- Check a U+00A0 interpolation boundary for unchanged suffix attachment, including whitespace preservation.

## Publication-Blockers

Prior art and a minimal public reproduction repository remain missing.
The user has not selected implementation work or approved publication.

## Next-Action

Summary: Search Unicode class boundary prior art
Action: Search upstream work for NBSP removal or inconsistent class-token whitespace definitions.
Done-When: Relevant matches are classified and the governing token-boundary contract is recorded.

## Bug-Reproduction

Environment: Ubuntu 24.04.5, Node.js 26.12.0, Prettier 3.9.8, and Tailwind CSS 4.3.3.
Input notation: `"\u00a0p-4 m-2"` denotes an actual leading U+00A0 character, not six literal escape characters.
Observed public sorter results are `"p-4 m-2"` and then `"m-2 p-4"` on successive passes.
Expected behavior: The original U+00A0-bearing token remains intact.
The tagged-template case uses an actual U+00A0 at the indicated position:

```text
tw`p-4 m-2<U+00A0>${suffix}`
```

Observed first output:

```text
tw`m-2<U+00A0> p-4${suffix}`
```

The disposable full-pipeline and public-API harnesses have been removed.
