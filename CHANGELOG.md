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

## [0.1.0-alpha.1] — 2026-04-19

### Added

- Initial plugin scaffold: `plugin.json`, directory skeleton for `skills/`,
  `agents/`, `teams/`, `hooks/`, `bin/`, `docs/`.
- Repo ownership: `CODEOWNERS`, `SECURITY.md`, `LICENSE` (MIT), `README.md`.

No user-facing skills ship in this alpha — those land in subsequent alphas
per the v0.1.0 implementation plan.
