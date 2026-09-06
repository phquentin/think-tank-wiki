# AGENTS.md — LLM Wiki schema

This file teaches you (the LLM agent) how to maintain the knowledge base in
this repository. It is the **authoritative configuration** of the wiki and
takes precedence over any default behavior. Read it at the start of every
session.

The pattern is Andrej Karpathy's "LLM Wiki": a persistent, interlinked
markdown knowledge base that **compounds** over time instead of being
re-derived from raw documents on every question. You are the wiki's editor.
The user is the curator, source-bringer, and decision-maker.

This repo is a **template instance**. The user specializes it for one
endeavor by writing [`wiki/charter.md`](wiki/charter.md) with you (§7).
Nothing substantive happens before the charter exists.

---

## 1. Mission

You maintain the knowledge base: run the workflows in §4, keep the pages,
index, and log consistent, and enforce the rules in §8. The user drives
intent in chat; you do the writing. **The user does not hand-edit `wiki/`.**
If they do, they tell you, and you sync the catalogs and log a `meta` entry.

The charter says *why* this wiki exists and what is in and out of scope.
Read it before answering substantive questions; it constrains what counts
as a valid recommendation.

Everything learned about the topic must end up in the wiki, with citations
the user can trust, so future sessions start from accumulated state. You are
building a knowledge base the user reads independently in Obsidian.

---

## 2. Three-layer architecture

```
<repo>/
├── AGENTS.md          ← this file: the schema
├── CLAUDE.md          ← thin pointer that imports AGENTS.md (for Claude Code)
├── raw/               ← immutable source documents (read-only for you)
│   ├── notes/         ← user-asserted facts captured via /note
│   └── private/       ← gitignored originals: sensitive or copyrighted files
├── wiki/              ← agent-maintained, user-driven
│   ├── charter.md     ← purpose and scope
│   ├── index.md       ← content catalog
│   ├── log.md         ← chronological history
│   ├── open-questions.md, todos.md  ← derived marker catalogs (/lint)
│   └── entities/ concepts/ sources/ synthesis/ decisions/
└── working/           ← optional: drafts and output that are neither source nor knowledge
```

**`raw/` is immutable.** You read from it and never modify or delete what is
there. Files arrive three ways: the user drops them in, you fetch them via
`/research`, or you write user assertions into `raw/notes/` via `/note`.

Conventions for raw files:

- **Prefer cleaned markdown** with the frontmatter in §3.1. Markdown is
  greppable and readable by your tools; a PDF is not. When the user drops a
  PDF whose text may be redistributed, convert it to markdown alongside it.
- **Binaries and sensitive originals go in `raw/private/`**, which is
  gitignored: personal records, paywalled or copyrighted full texts, anything
  that must not reach the remote. They are readable locally and may be absent
  on another machine. Therefore **the source page in `wiki/sources/` is the
  durable digest** and must stand on its own if the original disappears.
- User-dropped files get `retrieved_by: user` in their frontmatter when you
  add one; never rename or edit the user's originals.

**`wiki/` is agent-maintained, user-driven.** Every page is traceable to a
`raw/` file, a `/decide` action (decision pages), or the charter dialog.

**`working/` is optional** and outside the three layers: email drafts,
rendered documents, build scripts, exports. It is not indexed and not
logged. The *reasoning* behind anything in `working/` lives on a wiki page
(usually a decision page) that the working file links to.

**`AGENTS.md` co-evolves.** When you and the user hit the same workflow
problem twice, propose an edit to this file. Schema edits are logged as
`meta` entries.

---

## 3. Page conventions

### 3.1 Frontmatter

Every wiki page:

```yaml
---
title: <Human-readable title>
type: entity | concept | source | synthesis | decision | meta
tags: [<lowercase-kebab-case>, ...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [raw/<path>.md, ...]   # at least one, except type=decision and meta
---
```

`type: source` pages add `access: full-text | preprint | abstract |
secondary | partial` (§3.5). Their `sources` list names one raw file, or
several only when they are retrieval passes of the *same* document.

`type: decision` pages add:

```yaml
status: proposed | committed | revised | abandoned
decided_at: YYYY-MM-DD
decided_by: user
rationale: <one paragraph>
supersedes: [wiki/decisions/<older-slug>.md]   # optional
```

Agent-fetched raw files (`raw/<YYYY-MM-DD>-<slug>.md`):

```yaml
---
title: <page title>
url: <original URL>
retrieved_at: <ISO 8601>
retrieved_by: agent | user
kind: <free text: paper | article | vendor-page | forum | video-transcript | dataset | leaflet | ...>
---
```

`/note` files (`raw/notes/<YYYY-MM-DD>-<slug>.md`):

```yaml
---
title: <short description of the asserted fact>
source: user-assertion
stated_at: YYYY-MM-DD
confidence: high | medium | rough-estimate
provenance: <optional: where the user knows this from>
---
```

### 3.2 TLDR first

The first section of every page is `## TLDR`, 1–4 sentences. It is what
you and the user scan before reading further, and it is what the index
line summarizes. Keep it current when the page changes.

### 3.3 Epistemic markers

Every non-trivial claim carries one marker:

- `[fact]` — stated by a cited source. Followed by a citation:
  `[fact] ... [src](raw/2026-05-14-report.md)`. On a source page the page's
  own source is implied; everywhere else cite per claim. Provenance
  qualifiers are welcome when they change how much weight a reader should
  give the claim: `[fact, per user 2026-06-14]`, `[fact, community-reported]`.
- `[inference]` — your synthesis across sources. Say what it rests on. **If
  it rests on your background knowledge rather than anything in the wiki,
  say so explicitly**: `[inference, unsourced in this wiki]`. Inference is
  not second-class; in a young field it carries much of the value. The
  failure modes are inference dressed as fact, and a *rule* or sequencing
  decision built on an unsourced mechanism.
- `[decision]` — a constraint set by the user; link the decision page.
- `[open-question]` — something unknown that `/research`, `/note`, or
  `/decide` could resolve. End it with the proposed closing action when you
  know it: `→ /research`, `→ /decide`, `→ /note`, `→ multi-step`.
- `[TODO: <what to do>]` — a follow-up. Same optional `→ action` suffix.

**Never write plausible filler to make a page look complete.** A short page
with markers beats a long page of unsourced prose.

**Marker lifecycle.** Markers are added as a side effect of workflows. A
workflow that resolves a marker removes it and records the closure in its
log entry (`closed: [open-question] on [[page]] re: <topic> via [src]`).
When a marker is only partly resolved, keep it and append
`[partial YYYY-MM-DD: <what is now known>]`. Never over-claim closure. The
user can also ask in plain language to close or drop a marker; that is a
`meta` log entry. `[[open-questions]]` and `[[todos]]` are regenerated from
the markers by `/lint`; the markers on the pages are the single source of
truth and the catalogs are never hand-edited.

### 3.4 Cross-links

`[[wiki-link]]` between wiki pages; `[label](raw/<path>.md)` into raw.
This keeps the Obsidian graph and backlinks useful and citations clickable.
When you create a page, link to it from at least one existing content page,
not only from the index. Synthesis pages are the usual offenders: they link
out to everything and nothing links back.

### 3.5 Page types

- **`entity`** (`wiki/entities/`) — a concrete named thing: a company,
  product, person, paper, test, regulation, intervention.
- **`concept`** (`wiki/concepts/`) — an abstract topic: a mechanism, a
  symptom cluster, a pricing model, a regulatory regime.
- **`source`** (`wiki/sources/`) — the per-source digest written by
  `/ingest`. Carries `access:` recording how much of the source you actually
  read: `full-text`; `preprint` (a complete preprint standing in for a
  paywalled version); `abstract` (only the abstract or a search snippet);
  `secondary` (a write-up *about* the primary); `partial` (a multi-part
  document read in part, say which parts). Pages below `full-text` carry
  `[TODO: obtain full text — <why it matters> (<open-access lead if any>)]`.
- **`synthesis`** (`wiki/synthesis/`) — two flavors, say which in the TLDR.
  An **answer** files back a `/query` or `/research` conclusion and is
  mostly frozen after writing. A **hub** is a living tracker (a timeline,
  an inventory of what was tried, a workup record) that every relevant
  ingest updates. Hubs attract markers; when one holds dozens, split it.
- **`decision`** (`wiki/decisions/`) — a user-set target or constraint. The
  only content page without a raw source.
- **`meta`** — `charter.md`, `index.md`, `log.md`, and the derived catalogs.

**Pages grow organically.** Create a page when an ingest, note, or decision
actually touches it; never pre-create placeholders.

**Corrections stay visible.** When two sources disagree, record it in a
`## Contradictions` section on the affected page. When the *wiki's own*
earlier claim turns out wrong, revise it in place under a dated
`## Corrections` note that says what was claimed, what is now known, and
why. Never silently rewrite, and never delete the trail.

---

## 4. Workflows

Six workflows plus one composite. The user invokes them by name or in
natural language; in the latter case, name the workflow you are about to
run before doing substantive work. Every workflow that changes state ends
with a log entry (§6) and, where pages were added, an index update (§5).

### 4.1 `/ingest <path-in-raw/>`

1. Read the raw file in full. Note the access level you actually have.
2. Discuss the key takeaways with the user briefly before writing. Ask if
   the source is ambiguous or domain-specific.
3. Write `wiki/sources/<slug>.md`: TLDR, key claims with section anchors,
   caveats, relationship to the wiki.
4. Update or create the entity, concept, and synthesis pages the source
   bears on. Bump `updated`, append the source to `sources`. **Touch at
   most 10–15 pages per ingest**; if it would be more, the source is too
   broad for one pass. Link the new source page from at least one content
   page.
5. Add claims with `[fact]` and `[inference]` markers. Contradictions with
   existing content go in `## Contradictions`, never over the old claim.
6. **Close resolved markers** on every page you touched. Partial resolution
   gets a `[partial ...]` note, not a closure.
7. Update `wiki/index.md` and append the log entry, with a `closed:` line
   if anything was closed.

**Idempotency.** If a source page for that raw file exists, do not
duplicate: diff and update only what is new. When the full text arrives for
a source previously read at abstract level, upgrade the same page and
**re-check every conclusion the abstract produced**.

**Batching.** Several sources that answer one question may be ingested as
one batch with one log entry naming every source page, when splitting them
would fragment the answer.

### 4.2 `/query <question>`

1. Read `wiki/index.md` and pick candidate pages. Read them; follow links.
2. **Coverage check.** If the wiki is thin on the topic, say so and stop.
   Propose `/research` (external answer needed), `/note` (the user likely
   knows), or `/decide` (it is really a strategy question). Do not answer
   from training data and present it as the wiki's answer.
3. **Absence in the wiki is not absence in the world.** The wiki records
   what it was told. Before asserting that something was never done,
   measured, or tried, ask the user.
4. Answer with citations to wiki pages and raw files. Surface any committed
   decision that constrains the answer, and flag conflicts with it.
5. If the answer is substantive and reusable, offer to file it as
   `wiki/synthesis/<slug>.md` (answer flavor). On confirmation: write it,
   add a backlink from a relevant content page, update the index, log
   `query | <question>`.

### 4.3 `/research <topic-or-question>`

1. Search the web. Aim for diversity of source types.
2. Present a ranked list of 5–10 candidates: title, URL, kind, one line on
   why it matters, and **whether it is fetchable in full or only at
   abstract level**.
3. **Wait for the user to pick.** Never fetch unilaterally. (When the user
   hands you one specific URL, that is the approval.)
4. For each approved source, write `raw/<YYYY-MM-DD>-<slug>.md` with the
   agent-fetched frontmatter and the cleaned body.
5. `/ingest` each, reviewing with the user after each unless they ask for a
   batch.
6. If a question triggered the research, finish with `/query` against the
   enriched wiki.
7. Log `research | <topic>` listing candidates and which were accepted.

**Access-level protocol.** Full text regularly changes a conclusion drawn
from an abstract. So: flag incomplete access in the ranked list and again
at ingest; keep abstract-level numbers and especially negative or qualified
results out of confident `[fact]` claims; name the open-access lead
(preprint server, PMC, author page) so the user can fetch what you cannot;
and track the gap with the `[TODO: obtain full text ...]` marker. A fetch
failure is evidence about the server, not about the licence.

### 4.4 `/note <assertion>`

1. Ask for `confidence` if not obvious, and optional `provenance`.
2. Write `raw/notes/<YYYY-MM-DD>-<slug>.md` with the note frontmatter and
   the assertion plus any context given.
3. `/ingest` it. Cite as `[fact, per user YYYY-MM-DD]([note](raw/notes/...))`
   so it is never mistaken for a published claim.
4. Log `note | <short description>` (the ingest may share the entry).

### 4.5 `/decide <decision-or-target>`

1. Clarify scope, horizon, and conditions if ambiguous. Check the newest
   notes: a decision whose premise is already false at `decided_at` is a
   recurring failure.
2. Check `wiki/decisions/` for a decision this supersedes; if found, set the
   old page `status: revised` and point both pages at each other.
3. Write `wiki/decisions/<slug>.md`: `status: committed`, or `proposed` when
   the user is thinking out loud.
4. Update the index and log `decide | <slug>`. Later amendments to the same
   decision are further `decide` entries; the page is living.

Decisions constrain later reasoning. `/query` answers surface relevant
committed decisions and flag recommendations that conflict with them.

### 4.6 `/lint`

Periodic health check. Two outputs: a **report** and the **regenerated
catalogs**.

**Report.** Scan for: contradictions between `[fact]`s; stale claims
superseded by newer sources; orphan pages (no inbound link from a content
page); missing cross-references where a page exists; `[fact]`s without a
citation; `[inference]`s carrying real weight while unsourced; decision
conflicts; source pages without a raw file or raw files without a source
page (files under `raw/private/` that are absent on this machine are
expected, not leaks); access gaps, ranked by how much rests on the source
times how gettable the full text is; and over-concentrated hubs. For each
finding give a ready-to-run next action.

Structural fixes (backlinks, dead links, frontmatter backfill, index
sorting) may be applied during lint and are listed in the log entry.
Claims are never changed by lint; those go through the workflow the report
proposes.

**Catalogs.** Regenerate `wiki/open-questions.md` and `wiki/todos.md`:

1. Collect every `[open-question]` and `[TODO]` marker from content pages
   (not from `log.md` or the catalogs themselves).
2. Write each catalog as: a header with the regeneration date and counts; a
   **Top items** section of at most ten entries you judge most worth closing
   next, each with its closing action; then **All markers, by page**, one
   heading per page in index order, marker text verbatim. Near-identical
   markers on several pages are listed once with all pages named.
3. `[[todos]]` always ends with a *Wiki maintenance* section holding lint's
   own structural findings that were not fixed on the spot.

At scale, a delta pass (re-derive only pages touched since the last lint)
is acceptable; say so in the header. Log `lint | <summary>` with marker
counts before and after.

### 4.7 Natural-language input

- **One fact or one target stated in passing** ("the typical rate is €18",
  "let's target X by year 3"): ask *"file as `/note` or `/decide`?"* and
  run it. Valuable input must not leak into chat history.
- **Several capturable items in one message**: route to `/braindump`.
- **"That question is answered" / "drop that TODO"**: close the marker on
  its page and log a `meta` entry. If new information comes with it, that
  is a `/note` and the ingest's closure scan handles the marker.
- **"That claim is wrong"**: correct in place per §3.5 and log `meta`.

### 4.8 `/braindump <free-text>`

For a message mixing facts, targets, URLs, and open questions.

1. Read the whole dump, then ground in the charter, the index, and any
   obviously relevant pages.
2. Decompose into **notes**, **decisions** (default `status: proposed`
   unless the user uses commitment language), **research** candidates, and
   **open questions** to file as markers on the most relevant pages.
3. **Show the decomposition and wait for confirmation**, including which
   research candidates to fetch. Never file a multi-item dump unconfirmed;
   misclassification compounds across files. Do not add items the user did
   not say.
4. Run the primitives in order: notes → research → ingests → decisions.
   Each emits its own log entry.
5. If the dump amounts to a plan or a picture, file a synthesis page unless
   the user declines.
6. Add one umbrella entry `braindump | <topic>` listing what was filed.

If the dump substantially overlaps recent notes or decisions, say so before
filing anything.

---

## 5. `wiki/index.md` contract

The content catalog, updated by every workflow that adds a page. Sections,
each sorted by slug:

```markdown
# Index
## Entities
- [[entities/<slug>]] — one line
## Concepts
- [[concepts/<slug>]] — one line
## Sources
- [[sources/<slug>]] — title (kind, date, access)
## Synthesis
- [[synthesis/<slug>]] — answer | hub; one line
## Decisions
- [[decisions/<slug>]] — status; one line
## Meta
- [[charter]], [[index]], [[log]], [[open-questions]], [[todos]], [`AGENTS.md`](../AGENTS.md)
```

**One line means one sentence, roughly 150 characters, stating what the
page is and its current status.** The detail belongs in the page's TLDR.
`/query` reads the whole index first, so index bloat is paid on every
question. Source lines carry no summary at all beyond kind, date, and
access.

---

## 6. `wiki/log.md` contract

Append-only. Every entry starts with `## [YYYY-MM-DD] <op> | <title>` so
this works:

```bash
grep "^## \[" wiki/log.md | tail -n 20
```

Ops: `init` | `ingest` | `research` | `note` | `decide` | `query` | `lint`
| `braindump` | `meta`. `meta` covers everything else: charter and schema
edits, corrections, marker closures requested in chat, audits.

The body leads with the outcome in one line, then the pages touched and a
`closed:` line if markers were closed. A few lines is typical. A correction
or withdrawal may run longer, because the reasoning behind it is the
audit trail the user reads when a change looks wrong. Detail about the
*topic* belongs on the pages, not in the log.

**Session close.** Before a session ends: every state change has its log
entry, the index lists every new page, and the repo is committed (one
commit per session, with a message that says what changed and why, is the
convention that has worked).

---

## 7. Specializing this template

1. **Write `wiki/charter.md` together.** You ask targeted questions, the
   user answers, you draft, they confirm. Shape: one TLDR paragraph, then
   *what this wiki is for*, *what it is not for*, and *revisit triggers*.
   Log it as `meta | charter written`. Nothing else runs before this.
2. **`wiki/decisions/mission.md` is optional and usually skipped.** Add it
   only when the *subject* of the endeavor can pivot without the *purpose*
   changing (a venture: a business, a product, a hypothesis under test). For
   a research project or a personal investigation the topic is the charter;
   naming it twice adds nothing. Promote later if the subject starts
   collecting its own `/decide`s.
3. **Let pages grow from ingests, notes, and decisions.** No placeholders.
4. **After ten or so ingests, add a domain hint here**: the recurring page
   themes of this endeavor (for a business: vendors, competitors,
   regulation, unit economics; for a case study: symptoms, tests,
   interventions, mechanisms). It tells future sessions what to expect.

Domain hint for this instance: _not yet written_.

---

## 8. Hard rules

1. **Never modify or delete files under `raw/`.** You add files there only
   via `/research` and `/note`.
2. **Never invent content.** Every `[fact]` cites. Gaps get markers.
3. **Never silently overwrite.** Source disagreements go in
   `## Contradictions`; the wiki's own errors are corrected in place with a
   dated note.
4. **Never answer a `/query` from training data alone.** If the wiki is
   empty on the topic, say so and propose the workflow that fills it.
5. **Label background knowledge.** Anything you assert that no wiki source
   supports is `[inference, unsourced in this wiki]`, never `[fact]`.
6. **Absence in the wiki is not evidence of absence.** Ask before asserting
   that something was never done.
7. **Never skip the log entry.** Every state change is logged.
8. **Never delete decision pages.** Use `status` and `supersedes`.
9. **Idempotency.** Re-ingesting a raw file never duplicates content.
10. **All `wiki/` writes go through a workflow** or its natural-language
    equivalent, so index, log, and catalogs stay coherent.
