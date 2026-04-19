# loom

Universal Claude Code plugin: skills, agents, first-class team templates,
and hooks that adapt to every project you work in — without per-project
plugin copies.

Part of the [SkillWeave](https://github.com/skillweave) ecosystem. Shipped
through the [`skillweave` marketplace](https://github.com/skillweave/marketplace).

## Status

**Pre-alpha (`0.1.0-alpha.1`).** This repository holds the plugin scaffold
only. User-facing skills — `loom:init`, `loom:update-conventions`,
`loom:spec-review`, and the internal `loom:dispatch-team` — land in
subsequent alphas per the v0.1.0 implementation plan.

## Install (once skills ship)

```
/plugin marketplace add skillweave/marketplace
/plugin install loom@skillweave
```

## Runtime requirements

- Claude Code `2.1.114` or newer.
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in the environment — the
  team-dispatch primitive depends on the experimental teams feature.
- No language runtimes on your machine. The Go helper binary `treadle` is
  fetched on first use from
  [`skillweave/treadle`](https://github.com/skillweave/treadle) releases
  and cached under `~/.cache/loom/treadle-v<ver>/<os>-<arch>/`.

## License

MIT — see `LICENSE`.
