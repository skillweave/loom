---
title: Sample smoke-test spec
date: 2026-04-19
status: proposed
---

# Sample smoke-test spec

## Summary

A minimal spec used to exercise `loom:spec-review` end-to-end. It is
intentionally small but contains a handful of claims reviewers can
either verify or challenge so the dispatcher has something to produce
findings about.

## Goals

- Demonstrate that `loom:spec-review` can read this file, inject the
  surrounding project's `.loom/project.md` context, dispatch four
  reviewer agents, and merge their findings.
- Give each of the four reviewer roles something to react to.

## Design sketch

Introduce a new helper function `countOpenLocks()` that returns the
number of active advisory locks under `.loom/teams/state/`. It reads
the directory tree, checks each `.lock` file's mtime against
`StaleLockTTL`, and returns the count of locks that are live.

It should robustly handle symlinks and gracefully recover from I/O
errors. The implementation is straightforward — walk the tree, stat
each `.lock`, compare timestamps.

## Non-goals

- Expose the count via a CLI subcommand. Callers use the Go API
  directly.

## Security

The helper reads `.lock` file mtimes only. It does not parse their
contents. Path traversal is not a concern since the walk starts at a
fixed root. Secrets are not touched.

## Open questions

- Should "stale" locks be reported separately from live ones? Deferred.

## Known risks

None — this is a read-only helper.
