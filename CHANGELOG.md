# Changelog

All notable changes to `loom` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
