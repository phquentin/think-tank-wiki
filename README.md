# think-tank-wiki

A reusable **LLM Wiki template** for anything that benefits from a
persistent, citation-tracked knowledge base: a research project, a business
plan, a technical investigation, a personal case you are working out, a
literature review.

You clone it, sit down with the agent for one short session to write
[`wiki/charter.md`](wiki/charter.md) (the agent asks, you answer, the agent
drafts), and from then on the agent maintains the knowledge base for you:
disciplined about citations, contradictions, and accumulated state, instead
of re-deriving everything from raw documents on each question.

**You drive changes via chat, not by hand-editing wiki pages.** The
catalogs (`index.md`, `log.md`, `open-questions.md`, `todos.md`) only stay
coherent if writes flow through the workflows below.

This follows [Andrej Karpathy's LLM Wiki pattern][gist]. The authoritative
configuration is [`AGENTS.md`](AGENTS.md); the agent reads it at the start
of every session.

[gist]: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

## Layout

```
<your-repo>/
├── AGENTS.md             ← the schema the agent follows (start here)
├── CLAUDE.md             ← thin pointer that imports AGENTS.md (for Claude Code)
├── .claude/settings.json ← permission allow-list so routine wiki edits don't prompt
├── raw/                  ← immutable source documents
│   ├── notes/            ← facts you asserted via /note
│   └── private/          ← gitignored originals (personal records, paywalled PDFs)
├── wiki/                 ← agent-maintained, user-driven
│   ├── charter.md        ← purpose and scope of this wiki
│   ├── index.md          ← content catalog
│   ├── log.md            ← chronological history
│   ├── open-questions.md ← derived catalog of [open-question] markers (/lint)
│   ├── todos.md          ← derived catalog of [TODO] markers (/lint)
│   ├── entities/         ← concrete named things
│   ├── concepts/         ← abstract topics
│   ├── sources/          ← one digest per raw source
│   ├── synthesis/        ← filed-back answers and living hub pages
│   └── decisions/        ← your targets and constraints, via /decide
└── working/              ← optional: drafts and output that are neither source nor knowledge
```

- **`raw/`** is the source of truth and is never modified by the agent.
  Prefer cleaned markdown. Anything that must not reach the remote goes in
  `raw/private/`; the agent's digest in `wiki/sources/` is written to stand
  on its own if that original is missing on another machine.
- **`wiki/`** is written by the agent, every page traceable to a raw file, a
  decision you made, or the charter dialog.
- **`working/`** is for the things a wiki is not: an email draft, a rendered
  document, a build script. The reasoning behind them lives on wiki pages.
- **`AGENTS.md`** is what makes the agent a disciplined maintainer instead
  of a chatbot. Cursor reads it directly; Claude Code reads it through
  `CLAUDE.md`, which does nothing but import it.

## Workflows

| Command | What it does |
| --- | --- |
| `/ingest <path-in-raw/>` | Integrate one raw source: a source digest plus updates to the entity, concept, and hub pages it bears on. Idempotent. |
| `/query <question>` | Answer from the wiki with citations. If coverage is thin, the agent says so and proposes `/research`, `/note`, or `/decide` instead of improvising. Reusable answers are filed back as synthesis pages. |
| `/research <topic>` | Web search, a ranked candidate list with access level flagged, you pick, the agent saves each to `raw/` with provenance and ingests it. Ends by answering the question if there was one. |
| `/note <fact>` | Capture something you know. Saved to `raw/notes/`, then ingested; cited as `[fact, per user <date>]`. |
| `/decide <target>` | Capture a decision or constraint as a living page in `wiki/decisions/`. Later answers are filtered through it. |
| `/braindump <text>` | A mixed dump of facts, targets, URLs, and questions. The agent proposes a decomposition into the primitives above and waits for your confirmation before filing. |
| `/lint` | Health check: contradictions, stale claims, orphans, unsourced claims, access gaps. Regenerates the two marker catalogs. |

State a fact or a target in plain chat and the agent asks whether to file it
as `/note` or `/decide`. Tell it a question is answered or a claim is wrong
and it closes the marker or corrects the page in place, logged as `meta`.

## First session

The template ships empty: placeholder charter, empty index and log, empty
content directories.

1. **Skim [`AGENTS.md`](AGENTS.md)** so you know how the agent should
   behave, and push back on anything that feels wrong.
2. **Write [`wiki/charter.md`](wiki/charter.md) with the agent.** What is
   this wiki for, what is out of scope, what would make you revisit it.
   This is the one piece of specialization; `AGENTS.md` §7 has the details,
   including when the optional mission page is worth adding (rarely).
3. **Open the whole repo in Obsidian as a vault**, not just `wiki/`, so
   links into `raw/` resolve.
4. **Start working.** A few `/note`s or a `/braindump` of what you already
   know, then `/research` to map the landscape, is the usual on-ramp.

## Browsing in Obsidian

With the repo root as the vault, `[[wiki-links]]` resolve, citations into
`raw/` are one click away, and the graph view and backlinks panel work out
of the box. The [Dataview](https://github.com/blacksmithgu/obsidian-dataview)
plugin can query frontmatter (all decisions with `status: committed`, all
sources with `access: abstract`). Personal Obsidian state is gitignored.

## Customizing the schema

`AGENTS.md` is meant to co-evolve with how you actually use the wiki. When
you and the agent hit the same problem twice, edit the schema and log it.
After ten or so ingests, add the recurring page themes of your endeavor
under §7 as a domain hint.

If you run several instances from this template, diff their `AGENTS.md`
files now and then and backport what one instance learned to the others
and to this template.
