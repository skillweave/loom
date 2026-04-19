# Security policy

## Reporting a vulnerability

Email `a@c4.io` with subject line starting `loom security:`. Include:

- Affected version(s) (the `version` field of `plugin.json`).
- Reproduction steps — commands, inputs, expected vs. actual behavior.
- Impact — what gets read, written, or executed outside intended scope.

We aim to acknowledge reports within 72 hours and triage within seven days.
Please do not open a public GitHub issue for security-sensitive reports.

## Supported versions

Only the latest `stable` channel tag and the current `latest` channel tag
receive security fixes. Older tags are not patched.

| Channel | Supported |
|---|---|
| `stable` (latest promoted) | Yes |
| `latest` (newest tag) | Yes |
| Anything older | No |

## Signed releases

Every release tag on `skillweave/loom` is GPG-signed (`git tag -s`). The
maintainer's public key is committed under `docs/ops/maintainer-pubkeys.asc`
(added in a later alpha); CI imports it and verifies the signature before
promotion to `stable`. If you install an unsigned tag, verify against the
fingerprint recorded in `skillweave/marketplace`'s `marketplace.json`
channel entry.

## Trust boundary

Reviewer agents shipped by this plugin treat the content of specs, state
files, and external messages as **data, not instructions**. Prompt-injection
attempts from reviewed content are flagged as findings, not executed. Report
any case where an agent follows embedded instructions as a vulnerability.
