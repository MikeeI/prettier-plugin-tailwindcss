# Repository Guidelines

## Project Overview

`prettier-plugin-tailwindcss` is a Prettier plugin that sorts Tailwind CSS classes in the recommended class order.
It supports Tailwind CSS v3 and v4 and wraps Prettier parsers for HTML, CSS, JavaScript, TypeScript, and template languages.
The implementation is TypeScript, built with `tsdown` and tested with Vitest.
The package declares Node.js `>=20.19` and pins `pnpm@10.28.2`; CI uses Node.js 22.

## Fork & Upstream Contribution Intent

- Official upstream: [tailwindlabs/prettier-plugin-tailwindcss](https://github.com/tailwindlabs/prettier-plugin-tailwindcss).
- This checkout is the [MikeeI/prettier-plugin-tailwindcss](https://github.com/MikeeI/prettier-plugin-tailwindcss) fork, not an independently owned product.
- Related local checkout: `$HOME/projects/project-prettier-fork` tracks contributions to Prettier core (`prettier/prettier`).
- The goal is to support upstream with evidence-backed issues, comments, and pull requests.
- `ISSUES.md` provides the compact finding overview and global ID allocator.
- `issues/ISSUE-NNN.md` owns the complete durable record for one root cause.
- `FORMAT.md` owns research, drafting, implementation authorization, approval, and publication rules.
- Prefer small, well-scoped corrections with outsized maintainer or user value.
- Recommend a pull request when a bounded verified fix is ready and no active implementation owns it.
- Otherwise recommend a comment, a new issue, or continued investigation according to the evidence.
- Apply `skill-fork-contribution-tracking` for ledger, lifecycle, personal-branch, and upstream handoff work.
- Apply `skill-maintainer-communication` before external issues, pull requests, reviews, comments, or discussions.
- Search existing work first and follow current upstream templates and disclosure rules.
- Upstream accepts bug reports only with a minimal public reproduction repository per `.github/ISSUE_TEMPLATE/bug-report.md`.
- Upstream routes help requests and feature ideas to `tailwindlabs/tailwindcss` Discussions per `.github/ISSUE_TEMPLATE/config.yml`.
- Apply `skill-semantic-compression-3` when authoring or restructuring tracking content.
- Apply `skill-git-commit-format` while respecting explicit upstream contribution conventions.
- Never choose `Authorized-Work` or `Publication-Target` on the user's behalf.
- `Research-and-Reporting` permits research, issues, and comments but no source implementation.
- `Pull-Request-Implementation` authorizes only the scoped implementation recorded for that finding.
- Base upstream contribution branches on current `upstream/main`.
- Keep fork-only context, ledgers, configuration, and personal commits out of upstream contribution diffs.
- Reproduce claimed bugs against current upstream and run the narrowest conclusive verification.
- Publish one coherent root cause per issue, comment, or pull request.

## Finding and Contribution Ledger

- At the start of every agent session, agents MUST read root `ISSUES.md` before repository work.
- `ISSUES.md` owns the global `Next finding ID` allocator and compact cross-finding overview.
- Each `issues/ISSUE-NNN.md` owns one finding's state, evidence, drafts, and next action.
- `FORMAT.md` is authoritative for research, drafting, implementation boundaries, and publication format.
- Before adding a finding, search the index and every relevant issue record for the same symptom or root cause.
- New findings MUST use `Next finding ID`.
- Create the issue file, add its index row, and increment the allocator together.
- Finding IDs use `ISSUE-NNN`, start at `ISSUE-001`, and remain permanent.
- Never reuse, renumber, or scope IDs by subsystem, status, session, or contribution type.
- Update the issue file and `ISSUES.md` together after state, authorization, target, priority, next action, or reference changes.
- Every issue record MUST use the field and section contract in `FORMAT.md`.
- New findings start with `State: Investigating`, `Authorized-Work: Not-Selected`, and `Publication-Target: Not-Selected`.
- New findings use `External-Reference: Not published.` until an external reference exists.
- Keep findings Investigating until currentness, prior art, impact, and correction value are evidence-backed.
- Clone detectors, AST matches, text similarity, shared names, and TODOs produce candidates only.
- A duplication finding requires shared change pressure, realistic drift, and simpler consolidation.
- The user selects `Authorized-Work` for each finding.
- `Research-and-Reporting` MUST NOT implement the finding.
- `Pull-Request-Implementation` MAY implement only the recorded scope after research resolves callers and failure modes.
- Pull-request work MUST verify behavior, commit, push, and reach `PR-Ready` before publication.
- Show the exact draft and target before publishing an issue, comment, or pull request to official upstream.
- Publish to official upstream only after the user approves the exact current draft and target.
- Any draft or target change requires showing the complete current draft and target again before publication.
- Run the read-only validator bundled with `skill-fork-contribution-tracking` after every ledger mutation.
- Record the final external URL in `External-Reference` immediately after publication.
- Keep `FORMAT.md`, `ISSUES.md`, `issues/`, and fork-only `AGENTS.md` changes out of upstream contribution diffs.

### External publication approval

Only an external issue, comment, review, discussion, or pull request write is approval-gated.
Before publication, read current contribution guidance and explain applicable project policy.
The human must be able to review and own every submission statement.
Fork commits, pushes, tracking updates, and source implementation follow the active repository contract.

## Architecture and Data Flow

- `src/index.ts` defines per-language transforms that walk parser ASTs and rewrite class attributes, strings, and template literals.
- `src/create-plugin.ts` wraps the base Prettier parsers and printers, including compatible third-party plugins, with those transforms.
- `src/options.ts` declares plugin options such as `tailwindStylesheet`, `tailwindConfig`, `tailwindAttributes`, and `tailwindFunctions`.
- `src/config.ts` and `src/resolve.ts` locate the project's Tailwind installation, config, or stylesheet relative to the formatted file.
- `src/versions/v3.ts` and `src/versions/v4.ts` adapt Tailwind v3 and v4 into the shared `UnifiedApi` from `src/types.ts`.
- `src/sorting.ts` orders class lists; `src/sorter.ts` backs the public `prettier-plugin-tailwindcss/sorter` export.
- `tests/` holds Vitest suites for formatting, third-party plugin compatibility, the sorter API, and fixture projects under `tests/fixtures/`.

## Development Commands

- Install dependencies with `pnpm install`.
- Build with `pnpm run build`.
- Run tests with `pnpm run test`; `pretest` runs `npm install` in every fixture directory first.
- Run a focused suite with `pnpm exec vitest run tests/<file>.test.ts`.
- Lint with `pnpm run lint`, which runs `knip` and type-aware `oxlint`.
- Format with `pnpm run format`; CI checks it with `pnpm run format --check`.
- Do not replace the repository's pnpm workflow with Bun, npm, or Yarn.

## Code Conventions

- Follow the existing TypeScript style: 100-column print width, single quotes, and no semicolons.
- Add language support as a transform in `src/index.ts` and reuse the existing AST visitors and class-sorting helpers.
- Keep Tailwind v3 and v4 behavior consistent through `UnifiedApi` instead of branching in transforms.
- Add an entry under `## [Unreleased]` in `CHANGELOG.md` for user-visible changes, following Keep a Changelog.

## Important Files

- `package.json` defines scripts, peer-dependency plugin compatibility, the Node.js engine, and the pinned pnpm version.
- `.github/workflows/ci.yml` defines the required format, lint, build, and test gates.
- `.github/ISSUE_TEMPLATE/` defines upstream bug-report requirements and discussion routing.
- `tsdown.config.ts` patches `jiti` and inlines CSS assets during the build.
- `ISSUES.md`, `FORMAT.md`, and `issues/` are personal-fork tracking files, not upstream contribution files.

## Runtime and Tooling

- Node.js `>=20.19` is required by `package.json`; CI runs Node.js 22.
- Use the repository-pinned `pnpm@10.28.2`.
- Fixture projects install their own dependencies with npm through `scripts/install-fixture-deps.js`.

## Testing and Quality Assurance

- Reproduce sorting or plugin-compatibility defects with a focused test case or fixture before changing behavior.
- Use the narrowest relevant Vitest suite during iteration, then run the CI gates the change affects.
- Cover both Tailwind v3 and v4 when a change touches shared sorting or config loading.
- Do not describe a source-level invariant as observed user impact without reproduction.
