---
title: Log
type: meta
tags: [log]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Log

Append-only history of wiki operations. Entries start with
`## [YYYY-MM-DD] <op> | <title>` so this works:

```bash
grep "^## \[" wiki/log.md | tail -n 20
```

Ops: `init` | `ingest` | `research` | `note` | `decide` | `query` | `lint` |
`braindump` | `meta`. See [`AGENTS.md`](../AGENTS.md) §6.

---

## [YYYY-MM-DD] init | template instantiated

Scaffold created from the `think-tank-wiki` template: `AGENTS.md`, empty
`raw/` (with `notes/` and gitignored `private/`), empty `wiki/` content
directories, placeholder `charter.md`, this log, an empty index, and empty
marker catalogs. No subject yet; `wiki/charter.md` must be written before
substantive workflows begin.
