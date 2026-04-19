# smoke-dispatch fixture

Minimal project tree used to smoke-test `loom:dispatch-team` and
`loom:spec-review` end-to-end. Contents:

- `.loom/project.md` — the five required sections at `schema_version: 1`.
- `docs/specs/001-sample-spec.md` — a short spec with enough concrete
  claims for the four-reviewer team to produce findings about.

## Running the smoke test

From a Claude Code session with `loom@skillweave` installed and
`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` set:

```
cd tests/fixtures/smoke-dispatch
/loom:spec-review docs/specs/001-sample-spec.md
```

Expected behavior:

1. Skill finds `.loom/project.md` in the current directory.
2. Parses it without errors (no warnings expected).
3. Computes `state_key = sha256("docs/specs/001-sample-spec.md")[:16]`.
4. Invokes `loom:dispatch-team` with the four-round-reviewers team.
5. Dispatch runs round 1 (four reviewers); produces merged findings.
6. Between-round prompt appears; user can `c` to continue to round 2 or
   `x` to exit early.
7. Triage UI walks each finding; user picks accept / reject / defer.
8. Exits cleanly; `.loom/teams/state/four-round-reviewers/<state_key>/`
   contains `meta.json`, `findings.log.md`, and
   `trace/<session_id>.jsonl`.

## Re-running

The fixture's `.loom/teams/state/` directory is gitignored
(`/loom/.gitignore` already excludes `tests/fixtures/**/.loom/tmp/`
and `tests/fixtures/**/.loom/teams/state/`). Rerun the smoke test and
a second session's findings get appended to `findings.log.md`; the
state grows across invocations for the same spec path.

To start fresh, `rm -rf .loom/teams/state/`.
