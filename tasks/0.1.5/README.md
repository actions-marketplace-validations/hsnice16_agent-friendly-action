# 0.1.5 — match repo file lookups case-insensitively

**Status**: shipped (2026-08-23)

Mirrors upstream [`9371d6c`](https://github.com/hsnice16/agent-friendly-code/commit/9371d6c) into the vendored scorer.

Candidate paths were looked up with exact-match `existsSync`, but README / LICENSE / CONTRIBUTING casing varies genuinely in the wild (`readme.md`, `Readme.md`, `README.MD`). On a case-sensitive filesystem those files read as missing — and this action runs on Linux runners, so the wrong values were the ones consumers saw in their PR comments.

## What shipped

- **Case-insensitive path resolution** — `firstExisting` now resolves each path segment against a case-folded directory index, joined by `resolveRelative` (single lookup) and `resolveAllRelative` (deduped by resolved path). Exact spelling wins, so an exact match is never shadowed by a differently-cased sibling; entries are sorted so a genuine `README.md` + `readme.md` collision resolves identically on every run.
- **Nine signals** moved off raw `existsSync(join(repo, …))`; `gemini-md`'s hand-rolled case-insensitive scan folded into the shared resolver; glob regexes for `.csproj` / `.cabal` / `.nimble` / `.mdc` / `.ya?ml` made case-insensitive.
- **`dev_env` no longer double-counts one file.** Its candidate list carried both `Makefile` and `makefile`; with case-insensitive resolution both would have matched the same file and pushed the signal from 0.7 to a full 1.0. `resolveAllRelative` dedupes by resolved path and the redundant spelling is gone.

## Impact on PR comments

A repo whose README is spelled `readme.md` previously showed `README — No README found` in the score breakdown. It now scores normally.

The first run after upgrading will report a **positive delta** on affected repos — up to 18 points where README, LICENSE, and CONTRIBUTING were all mis-detected. That delta reflects the correction, not a change in the repo. Widening a matcher can only turn a miss into a hit, so no repo loses a signal it previously passed.
