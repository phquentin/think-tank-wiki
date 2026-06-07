# think-tank-wiki

A reusable **LLM Wiki template** for working out anything that benefits from
a persistent, citation-tracked knowledge base: business plans, research
projects, technical investigations, life decisions, market analyses,
literature reviews, etc.

You clone this template, sit down with the agent for one short session
to fill in [`wiki/charter.md`](wiki/charter.md) (the agent asks, you
answer, the agent drafts) saying what *this* instance of the wiki is
for, and from then on the agent maintains the knowledge base for you —
disciplined about citations, contradictions, and accumulated state —
instead of re-deriving everything from raw documents on each query.

Throughout, **you drive changes via chat, not by hand-editing wiki
pages.** The agent's catalogs (`index.md`, `log.md`, `open-questions`,
`todos`) only stay coherent if writes flow through the workflows below.

This follows [Andrej Karpathy's LLM Wiki pattern][gist].

The authoritative configuration is [`AGENTS.md`](AGENTS.md) — read that
before doing anything substantive with the wiki. The agent reads it at
the start of every session.

[gist]: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

## Three-layer architecture

```
<your-repo>/
├── AGENTS.md     ← the schema the agent follows (start here)
├── CLAUDE.md     ← thin pointer that imports AGENTS.md (for Claude Code)
├── raw/          ← immutable source documents
│   └── notes/    ← user-asserted facts via /note
└── wiki/         ← agent-maintained, user-driven (edits via workflows, not by hand)
    ├── charter.md ← purpose of this wiki (drafted with the agent)
    ├── index.md  ← content catalog
    ├── log.md    ← chronological history
    └── decisions/ ← user-set targets/decisions via /decide
```

- `raw/` is the **source of truth**. Immutable. The agent reads from it,
  never modifies it. Files arrive there in three ways: you drop them in,
  the agent fetches them via `/research`, or the agent writes user
  assertions there via `/note`.
- `wiki/` is **agent-maintained, user-driven**. The agent does the
  writing — compiling, cross-referencing, and maintaining the knowledge
  derived from `raw/` — but every change is initiated by you through a
  workflow (or its natural-language equivalent). You read it; you don't
  hand-edit it.
- `AGENTS.md` is the **schema** — what makes the agent a disciplined wiki
  maintainer instead of a generic chatbot. It is the single source of truth.
  Cursor reads it directly; Claude Code reads it via `CLAUDE.md`, a thin
  pointer that does nothing but `@`-import `AGENTS.md`. Don't put schema
  content in `CLAUDE.md` — keep it in `AGENTS.md` so the two never drift.

## Workflows

Invoke from your LLM coding environment (Cursor, Claude Code, etc.) by
name, or describe the intent in natural language and the agent will
confirm before acting.

| Command | What it does |
| --- | --- |
| `/ingest <path-in-raw/>` | Integrate one raw source into the wiki, updating relevant entity/concept pages, `index.md`, and `log.md`. Idempotent. |
| `/query <question>` | Answer from the wiki, with citations. If coverage is thin, the agent will refuse to improvise and propose `/research`, `/note`, or `/decide`. Good answers can be filed back as `wiki/synthesis/<slug>.md`. |
| `/research <topic>` | Cold-start / gap-filling. The agent searches the web, presents ranked candidate sources, you approve, it saves to `raw/<YYYY-MM-DD>-<slug>.md` with provenance, then `/ingest`s each. Ends by answering the original question if there was one. |
| `/note <fact>` | Capture an empirical fact you already know. Saved to `raw/notes/`, then ingested. Future citations read `[fact, per user]`. |
| `/decide <decision-or-target>` | Capture a decision/target/constraint (e.g. *"scope: EU only in year 1"*). Saved as a living page in `wiki/decisions/`. Used as a constraint in subsequent `/query` answers. |
| `/braindump <free-text>` | Capture a mixed dump of facts + targets + URLs + open questions in one pass. The agent decomposes it into `/note`, `/decide`, and `/research` candidates and waits for your confirmation before filing. |
| `/lint` | Periodic health check: contradictions, stale claims, orphan pages, unsupported facts, gaps. Produces a report and regenerates the `[[open-questions]]` and `[[todos]]` catalogs. Never auto-fixes content pages. |

If you just state a fact or set a target in plain chat, the agent will
ask whether to file it as `/note` or `/decide` — so nothing valuable
leaks into chat history.

## First session

The template ships **empty by design** — placeholder charter, empty
index, empty log, empty content directories. To start using it:

1. **Skim [`AGENTS.md`](AGENTS.md)** so you know how the agent is
   supposed to behave. Push back if anything feels wrong; the schema
   is meant to co-evolve with you.
2. **Fill in [`wiki/charter.md`](wiki/charter.md) with the agent.**
   This is the *one* piece of specialization that turns the template
   into your wiki. The agent will ask a few targeted questions, draft
   the page, and you confirm — same collaborative shape as `/decide`.
   It should answer: what is this wiki for? What's in scope, what's
   not, and what would cause you to revisit the charter? See
   `AGENTS.md` §7 for the exact contract.
3. **Optionally fill in `wiki/decisions/mission.md` with the agent.**
   Same collaborative flow as the charter. **Mission is the exception,
   not the rule** — most endeavors are charter-only. Earn its keep
   with the *pivot test*: could you change *what* you're working on
   without changing *why* this wiki exists? Ventures (a business, a
   product, a hypothesis-under-test) usually pass; investigative
   endeavors (research, literature review) don't. See `AGENTS.md` §7
   for details. When in doubt, skip it — you can always add it later.
4. **Open this repo in Obsidian as a vault.** Open the *whole repo*
   (not just `wiki/`) so links from wiki pages into `raw/` resolve.
   See "Browsing in Obsidian" below.
5. **Start a session in your LLM coding tool.** A natural first move
   is `/decide` (capturing your initial assumptions and constraints)
   followed by `/research` (mapping the landscape).

## Browsing the wiki in Obsidian

Open the **whole repo as your Obsidian vault** (not just `wiki/`).
That way:

- `[[wiki-link]]` cross-links between wiki pages resolve.
- Citations like `[label](raw/notes/foo.md)` from wiki pages into raw
  sources also resolve — one click takes you from a synthesis page to
  the underlying note or article.
- The graph view, backlinks panel, and search work out of the box.
- The [Dataview](https://github.com/blacksmithgu/obsidian-dataview)
  plugin can run queries over wiki frontmatter (e.g. *"all decisions
  with status: committed"*) — optional but useful.

Personal Obsidian workspace state is gitignored via `.gitignore`.

## Customizing the schema

`AGENTS.md` is meant to **co-evolve** with how you actually use the
wiki. If you and the agent repeatedly hit the same workflow problem,
edit the schema. Specifically, after ~5–10 ingests you'll have a feel
for the recurring page themes in your endeavor — add a short list of
them under §7 of `AGENTS.md` so future sessions have a domain hint.

If you maintain multiple wiki instances from this template, periodically
diff their `AGENTS.md` files to backport improvements. Schema fixes
made in one instance are usually relevant to the others.
