# Changelog

All notable changes to `loom` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0-alpha.6] -- 2026-04-19

### Changed

- **`dispatch-team` SKILL.md rewritten against coarse-grained treadle
  lifecycle subcommands** (`dispatch-init`, `round-init`,
  `round-finalize`, `dispatch-end` -- new in treadle v0.1.2). Alpha.5's
  smoke #4 Part A surfaced ~180k tokens / ~25 pre-first-review Bash
  approvals per dispatch: "massive bloat", "terrible UX". Root cause
  was fine-grained orchestration -- the SKILL.md chained 60 to 80
  atomic helper calls per dispatch for operations that never needed
  LLM judgment (args parse, plugin-root resolve, fs-locality, lock,
  parse-team, policy resolve, state-dir compute, per-round dispatch
  and kickoff traces, append-findings-log, save-meta, TeamDelete, lock
  release). Those now collapse into four JSON-in/JSON-out subcommands
  that each bundle 5 to 15 atomic ops plus the relevant trace events.
  The SKILL.md retains prompt composition + the teams-primitive tool
  call sequence + the round-drive watch loop + LLM-judgment work
  (structural sentinel deviation detection, semantic finding dedup
  across members). Target metrics for a clean 2-round 4-reviewer
  dispatch: **6 to 10 Bash calls per dispatch, under 5 before the
  first reviewer fires, under 200-ish lines in SKILL.md, under 40k
  tokens per spec review**.
- **`bin/VERSION` pinned to `0.1.2`.** Bumped from `0.1.1` to pull the
  new lifecycle subcommands. Shim cache path encodes the version, so
  the first call from each user re-fetches + re-verifies SHA256 on
  v0.1.2.
- **`bin/treadle` shim exports `LOOM_PLUGIN_ROOT`** before exec so
  subcommands that need the plugin root (e.g., `dispatch-init`
  resolving `teams/<name>.md`) can read it from env without the
  SKILL.md having to walk up the shim's path in a separate Bash call.

## [0.1.0-alpha.5] — 2026-04-19

### Fixed

- **Sentinel-format drift in reviewer agents.** Smoke test #3 (alpha.4)
  surfaced reviewers improvising round-complete sentinels — `---END
  tech-reviewer---`, `<<<END:security-reviewer>>>`, `<!--
  skeptic-reviewer:end -->` — instead of the prescribed
  `===<ROLE>-REVIEWER-FINAL===`. The dispatcher (LLM-driven, not
  regex-driven) accepted them on judgment, but the variants undermine
  determinism and would defeat any future automated parser. Two-part
  fix:
  - Each reviewer agent (`tech-reviewer`, `coverage-reviewer`,
    `skeptic-reviewer`, `security-reviewer`) now has a dedicated
    `## Sentinel — output verbatim` section that quotes the literal
    string in a fenced code block, names the observed anti-patterns,
    and explicitly forbids markdown emphasis / extra spaces / alternate
    framing.
  - `loom:dispatch-team` Step 8.1's per-member prompt reinforces the
    byte-equal requirement, and Step 8.4 now traces deviations as
    `agent-sentinel-deviation` events while still accepting them as
    completion signals (don't block round completion on cosmetic
    drift).

### Changed

- **Per-round kickoff message + bounded silence clock.** Step 8.3
  now sends each dispatched member an explicit `SendMessage` kickoff
  after dispatch. Two reasons: (1) it gives the dispatcher a single
  audited "go" signal correlatable with later peer chatter, and (2)
  it defines T0 for the per-member silence countdown — without it,
  "silent for 60s" is ambiguous because dispatch latency varies. Each
  kickoff is traced as `agent-kickoff`. Smoke #3 worked without it,
  but the ambiguity around T0 made the silence rule fragile.
- **One re-nudge before degrading silent reviewers.** Step 8.4's
  silence rule was "no sentinel after 120s -> degraded" with no
  recovery path. Now: at 60s of silence, dispatcher sends one
  `SendMessage` reminder (traced as `agent-renudge`); at ~120s
  total (60s post-renudge), the member is marked degraded with
  `reason: no sentinel after 60s post-renudge`. Cheap insurance
  against a single missed wake-up.

### Documentation

- **`team-forced-cleanup` trace event is now formalized.** Step 8.7
  previously said "trace `team-forced-cleanup`" inline without a
  command block. Replaced with the full `treadle trace ...
  --json-fields=...` invocation including `team_name` + `reason`
  fields, matching the style of the other trace events in this
  skill. Operators can now grep the trace for forced cleanups
  consistently.

## [0.1.0-alpha.4] — 2026-04-19

### Fixed

- **`SendMessage` added to reviewer agents' tool allow-lists.** Alpha.3
  instructed reviewers to SendMessage their findings to team-lead but
  omitted the tool from the `tools:` frontmatter. Per Phase 0 Prereq 3,
  the harness refuses calls to tools not in the allow-list — so the
  agents literally could not originate a SendMessage despite being
  told to. Smoke test #2 caught this: reviewers idled silently. All
  four agents now list `[Read, Grep, Glob, SendMessage]`; peer
  cross-collab (reviewer-to-reviewer) also now works for the same
  reason.

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
