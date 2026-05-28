---
title: Log
type: meta
tags: [log]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Log

Append-only chronological history of wiki operations. Entries are
prefixed `## [YYYY-MM-DD] <op> | <title>` so they're greppable:

```bash
grep "^## \[" wiki/log.md | tail -n 20
```

Op types: `init` | `ingest` | `research` | `note` | `decide` | `query` |
`lint` | `braindump`. See [`AGENTS.md`](../AGENTS.md) §6 for the contract.

---

## [YYYY-MM-DD] init | template instantiated

Three-layer scaffold created from the `think-tank-wiki` template:
`AGENTS.md` schema, empty `raw/` (with `notes/` subdir), empty `wiki/`
(with `entities/`, `concepts/`, `sources/`, `synthesis/`, `decisions/`,
`charter.md` placeholder, `fundamentals.md` stub, `index.md`, this `log.md`,
empty `open-questions.md` and `todos.md`). No subject yet — `wiki/charter.md`
needs to be written before substantive workflows begin.
