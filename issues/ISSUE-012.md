# ISSUE-012 — Release channel extraction misclassifies named prereleases

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: Build
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/main@5b27f947f5d3e6de8702dc40d4054e02fbf4cdd8`

## Root-Cause

Audit reference: `E12`, retained by the self-review as a direct CLI-output defect.
Root-Cause [S]: The release-channel regex requires a dot after the prerelease identifier and can capture build metadata.
The owning code is `scripts/release-channel.js:19-23`.

## Reach-and-Impact

Reach [O]: Direct invocation with valid named prereleases can produce the wrong npm channel.
Impact [O]: Unnumbered `beta` maps to `latest`, and a build-metadata form produces `beta+build`.
Impact [A]: A release workflow using this output without another guard could choose an unintended publication tag.
No npm publication or changed release state was performed or observed.
Severity: Low.

## Evidence

- [S] `scripts/release-channel.js:1-8` describes deriving the npm release channel from a version.
- [S] `package.json:40` exposes the script as `release-channel`.
- [O] `node scripts/release-channel.js 1.2.3-beta` prints `latest`.
- [O] `node scripts/release-channel.js 1.2.3-beta.1` prints `beta`.
- [O] `node scripts/release-channel.js 1.2.3-beta+build.42` prints `beta+build`.

## Prior-Art

Coverage: The local index and active records were empty before allocation.
Gaps: Upstream issues, PRs, discussions, releases, and linked forums have not been searched.
Contribution fit: Unresolved until maintainer value and supported release-version forms are established.

## Proposed-Change

Extract a named prerelease channel before the first dot or `+`, without requiring a numeric suffix.
Retain `latest` for stable versions.

## Scope-and-Constraints

- Preserve the documented numbered `alpha` and hashed `insiders` examples.
- Do not invent a publication policy for every possible numeric SemVer prerelease.
- Keep the finding scoped to observed CLI output rather than claiming an actual bad publication.
- Do not execute release or publishing commands during verification.

## Verification

- Run the direct read-only CLI cases recorded in Evidence after correction.
- Require both demonstrated `beta` forms and the build-metadata case to print `beta`.
- Confirm the documented stable, `alpha`, and `insiders` channel examples remain valid.

## Publication-Blockers

Prior art and supported release-version policy remain unresolved.
No evidence establishes that an active publication workflow consumes an affected version form.
The user has not selected implementation work or approved an external target and draft.

## Next-Action

Summary: Search release channel policy prior art
Action: Search upstream work for supported prerelease forms and existing channel-extraction fixes.
Done-When: The supported input contract and contribution value are recorded without claiming an observed publication.

## Bug-Reproduction

Environment: Ubuntu 24.04.5 and Node.js 26.12.0.
The three commands in Evidence ran directly against the repository script without harness substitutions.
Expected behavior for the demonstrated named prereleases: The channel is `beta`, without build metadata.
Only local CLI output was produced.
