# Changelog

All notable changes to `loom` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0-alpha.3] — 2026-04-19

### Fixed

- **Reviewer agents now deliver findings via `SendMessage`.** The smoke
  test surfaced that plain-text output from a teammate stays local —
  `team-lead` never sees it. Updated all four reviewer agent bodies
  (`tech-reviewer`, `coverage-reviewer`, `skeptic-reviewer`,
  `security-reviewer`) with an explicit "SendMessage your findings
  block to team-lead" instruction, and added the same reminder to
  `dispatch-team`'s per-member prompt (Step 8.1). Belt + suspenders.
- **`loom:spec-review` now walks up from cwd** to find `.loom/project.md`
  (with fallbacks to git toplevel and pwd). Previously `REPO_ROOT` was
  pinned to `git rev-parse --show-toplevel`, which meant running from
  inside a subdir of a repo where `.loom/` isn't at the git root would
  fail. `loom:dispatch-team` Step 4 got the same walk-up so `STATE_DIR`
  lands next to `.loom/project.md` regardless of caller cwd.
- **`coverage-reviewer`'s 10-finding cap is now explicit.** Smoke test
  saw 14 findings emitted; hard-cap language added to the agent body.

### Changed

- `bin/VERSION` pinned to `0.1.1` to pull the treadle flag-parsing fix
  (see [`skillweave/treadle` v0.1.1](https://github.com/skillweave/treadle/releases/tag/v0.1.1)).
  Shim cache path encodes version, so the first call from each user
  re-fetches + re-verifies SHA256 on v0.1.1.

## [0.1.0-alpha.2] — 2026-04-19

### Added

- `agents/tech-reviewer.md`, `agents/coverage-reviewer.md`,
  `agents/skeptic-reviewer.md`, `agents/security-reviewer.md` — stack-neutral
  reviewer roles with `tools: [Read, Grep, Glob]`. Each emits a named
  sentinel line when its per-round findings are final.
- `teams/four-round-reviewers.md` — the first team template. Declares
  members + policy (with `max_rounds_ceiling`), persistence (mode
  `persistent`, appendable findings log), and `sharing.scope: local`.
  Orchestration body documents per-round protocol, fresh-dispatch between
  rounds, dedup rule, sentinel-based round completion, between-round user
  prompt, degraded-round handling, and final synthesis.
- `bin/treadle` + `bin/VERSION` — POSIX sh shim that resolves OS/arch,
  fetches the pinned treadle binary from
  [`skillweave/treadle`](https://github.com/skillweave/treadle) releases
  on first use, verifies SHA256 against the release's
  `checksums.sha256`, caches under
  `~/.cache/loom/treadle-v<ver>/<os>-<arch>/treadle`, and execs.
  Plugin `bin/` is auto-prepended to `PATH`, so skills invoke bare
  `treadle <cmd>`. VERSION pins to `0.1.0`.
- `skills/dispatch-team/SKILL.md` — internal helper skill. Parses a
  team template, resolves policy (with `max_rounds_ceiling` enforcement),
  manages persistent state (atomic writes, advisory lock with 1h
  stale-TTL reclaim, template-evolution quarantine), dispatches members
  per round via `loom:<member>` namespace, drives sentinel-based round
  completion, persists JSONL trace + append-only findings log. Returns
  a structured result to the calling skill.
- `skills/spec-review/SKILL.md` — reference skill. Canonicalizes the
  spec path, parses `.loom/project.md`, computes a `state_key`,
  invokes `loom:dispatch-team`, and runs a per-finding triage loop
  (accept / reject / defer) before applying user-supplied edits only
  to the spec file under review.

## [0.1.0-alpha.1] — 2026-04-19

### Added

- Initial plugin scaffold: `plugin.json`, directory skeleton for `skills/`,
  `agents/`, `teams/`, `hooks/`, `bin/`, `docs/`.
- Repo ownership: `CODEOWNERS`, `SECURITY.md`, `LICENSE` (MIT), `README.md`.

No user-facing skills ship in this alpha — those land in subsequent alphas
per the v0.1.0 implementation plan.
