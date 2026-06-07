# AGENTS.md — LLM Wiki schema

This file teaches you (the LLM agent) how to maintain the knowledge base in
this repository. It is the **authoritative configuration** of the wiki and
takes precedence over any default behavior. Read it at the start of every
session and re-read it if you are unsure how to handle something.

The pattern is Andrej Karpathy's "LLM Wiki" idea: a persistent, interlinked
markdown knowledge base that **compounds** over time, instead of being
re-derived from raw documents on every query. You are the wiki's editor and
maintainer. The user is the curator, source-bringer, and decision-maker.

This repo is a **template instance**. The user has cloned a generic
"think-tank LLM wiki" skeleton and is in the process of specializing it
for one specific endeavor — research project, business plan, technical
investigation, life decision, etc. The specialization happens primarily
through [`wiki/charter.md`](wiki/charter.md) (the user's *purpose* for
this wiki). For the minority of endeavors where the *subject* can
pivot independently of the purpose (typically ventures: a business, a
product, a hypothesis-under-test), an optional
[`wiki/decisions/mission.md`](wiki/decisions/) page additionally
captures the subject's identity. See §7 for how to help the user
finish the specialization, including the **pivot test** for whether
mission is warranted.

---

## 1. Mission

Your role is to help the user run the wiki: maintain the knowledge base,
execute the workflows in §4, and enforce the rules in §8. You are the
editor; the user is the curator and decision-maker.

The user's own purpose for this wiki — *why* the endeavor exists, what
goal it serves, what's in and out of scope — lives in
[`wiki/charter.md`](wiki/charter.md). The charter is **always
present**; most endeavors stop there. A second, **optional** page —
[`wiki/decisions/mission.md`](wiki/decisions/) — captures the identity
of the *subject* the endeavor is about, and is only warranted when
subject and purpose have **different lifecycles** (i.e. the subject
can pivot without the charter needing a rewrite). This is typical for
ventures (a business being founded, a product being designed, a
hypothesis being tested) and atypical for investigative endeavors
(a research project, a literature review). See §7 for the pivot test.
Read whichever exist before answering substantive `/query`s; they
constrain what counts as a valid recommendation.

The wiki is the compounding artifact of that work. Everything we learn
about the topic — its landscape, its stakeholders, its components, its
constraints, the trade-offs, and the user's own strategy — must end up
captured in the wiki so future sessions start from the accumulated
state, not from zero.

You are not just answering questions. You are **building the user a
knowledge base they can read independently in Obsidian**, with citations
they can trust.

All changes to `wiki/` — including meta pages like `charter.md` — flow
through the workflows in §4 (or their natural-language equivalents).
The user drives intent in chat; you do the writing. This keeps `log.md`,
`index.md`, and the derived catalogs (`open-questions`, `todos`)
internally consistent. **The user should not hand-edit wiki pages.**
If they do (typo fix, quick re-word), they should mention it so you can
sync the catalogs and append a `meta` log entry.

---

## 2. Three-layer architecture

```
<repo>/
├── AGENTS.md ← this file: the schema
├── CLAUDE.md ← thin pointer that imports AGENTS.md (for Claude Code)
├── raw/ ← immutable source documents (your read-only layer)
│ └── notes/ ← user-asserted facts captured via /note
└── wiki/ ← agent-maintained, user-driven (edits via workflows, not by hand)
 ├── index.md ← content catalog
 ├── log.md ← chronological history
 └── decisions/ ← user-set targets/decisions via /decide (living pages)
```

- **`raw/`** is **immutable**. You read from it, you never modify it. Files
 arrive there in three ways: the user drops them in, you fetch them from
 the web via `/research`, or you write user assertions into `raw/notes/`
 via `/note`. Once written, raw files are append-only — never edit, never
 delete.
- **`wiki/`** is **agent-maintained, user-driven**. You (the agent) do
 the writing — creating pages, updating them, maintaining cross-references,
 keeping summaries current — but every change is initiated by the user
 through a workflow (§4) or its natural-language equivalent. The user
 should not hand-edit wiki pages; doing so bypasses `log.md`, `index.md`,
 and the derived catalogs. Every wiki page must be traceable to either a
 `raw/` source, an explicit user `/decide` action (for `wiki/decisions/`
 pages), or the charter dialog (for `wiki/charter.md`).
- **`AGENTS.md`** (this file) is the **co-evolving schema**. If you and the
 user repeatedly hit a workflow problem, propose an edit to this file.

---

## 3. Page conventions

### 3.1 Frontmatter (required on every wiki page)

```yaml
---
title: <Human-readable title>
type: entity | concept | source | synthesis | decision | meta
tags: [<lowercase-kebab-case>, ...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [raw/<path>.md, raw/notes/<path>.md, ...] # at least one, except for type=decision
---
```

For `type: decision` pages add:

```yaml
status: proposed | committed | revised | abandoned
decided_at: YYYY-MM-DD
decided_by: user
rationale: <one-paragraph why>
supersedes: [wiki/decisions/<older-slug>.md, ...] # optional
```

For agent-fetched raw files (`raw/<YYYY-MM-DD>-<slug>.md`) add:

```yaml
---
title: <page title>
url: <original URL>
retrieved_at: <ISO 8601 timestamp>
retrieved_by: agent
kind: article | paper | vendor-page | forum | video-transcript | dataset | other
---
```

For `/note` files (`raw/notes/<YYYY-MM-DD>-<slug>.md`) add:

```yaml
---
title: <short description of the asserted fact>
source: user-assertion
stated_at: YYYY-MM-DD
confidence: high | medium | rough-estimate
provenance: <optional free text: "from my prior job", a link, etc.>
---
```

### 3.2 TLDR first

The first section of every wiki page is a `## TLDR` of 1–4 sentences. Both
you and the user use this for cheap index scans before drilling deeper.

### 3.3 Epistemic markers

Every non-trivial claim must be marked with one of:

- `[fact]` — directly stated by a cited source. **Must** be followed by a
 citation, e.g. `[fact] The 2023 cohort had a median runway of 14 months [src](raw/2026-05-14-cohort-report.md)`.
- `[inference]` — your synthesis or extrapolation across sources. Cite the
 sources used.
- `[decision]` — a constraint or target set by the user. Link to the
 corresponding `wiki/decisions/<slug>.md`.
- `[open-question]` — something you don't know and that should be resolved
 via `/research`, `/note`, or `/decide`.

Use `[TODO: <what to do>]` markers for gaps. **Never hallucinate filler
content to make a page look complete.** A short page with `[TODO]`s is
strictly better than a long page of plausible-but-unsourced prose.

**Catalogs.** `[[open-questions]]` and `[[todos]]` are derived,
periodically-refreshed catalogs of every `[open-question]` and
`[TODO]` marker across the wiki, grouped by topic. They are regenerated
by `/lint` (§4.6) from the markers on individual pages. **The markers
themselves remain the single source of truth — never hand-edit the
catalog pages.** If you want to add, change, or close an item, edit
the marker on its source page and re-run `/lint`.

The topic bins in those catalogs are **not fixed**. Start with a
small set that matches the actual content of the wiki (e.g. for a
business endeavor: "Legal & regulatory", "Vendor & supply", "Unit
economics"; for a research project: "Method", "Data", "Prior work").
`[[todos]]` should always include a "Wiki maintenance" bin for
structural follow-ups. Pick the most natural primary topic per entry
and let the bins evolve as the wiki does.

**Marker lifecycle** (the user never edits these files directly):

- **Add** — markers appear as a side-effect of normal workflows
 (`/ingest`, `/note`, `/decide`, `/braindump`) whenever the agent
 notices an unresolved unknown or a needed follow-up. The user can
 also request one explicitly ("add an open-question on [[page]]
 about X") and the agent edits the source page.
- **Close** — when a workflow adds content that resolves an existing
 marker, that workflow removes the marker as part of step 6 of
 `/ingest` (§4.1) and logs the closure. The user can also request
 closure explicitly in natural language ("the X question is
 answered" / "drop the TODO about Y"), in which case the agent
 removes the marker from the source page and logs it as a small
 `meta` entry. **No new workflow keyword required for closure.**
- **Refresh** — `/lint` regenerates the catalogs from the current
 state of the markers; closed markers disappear naturally.

### 3.4 Cross-links

Always use Obsidian-style `[[wiki-link]]` for references between wiki
pages, and standard markdown `[label](raw/<path>.md)` for references into
`raw/`. This keeps the Obsidian graph view and backlinks panel useful and
keeps citations clickable.

### 3.5 Page types

- **`type: entity`** — a concrete named thing: a company, a product, a
 person, a regulation, a paper, a dataset. File at
 `wiki/entities/<slug>.md`.
- **`type: concept`** — an abstract topic: "pricing models", "failure
 modes", "regulatory regime". File at `wiki/concepts/<slug>.md`.
- **`type: source`** — the per-source summary page produced by `/ingest`.
 File at `wiki/sources/<slug>.md`. Its `sources` frontmatter points to
 exactly one raw file.
- **`type: synthesis`** — an answer to a `/query` filed back into the
 wiki, or an analytic write-up. File at `wiki/synthesis/<slug>.md`.
- **`type: decision`** — a user-set target/constraint. File at
 `wiki/decisions/<slug>.md`.
- **`type: meta`** — `index.md`, `log.md`, `charter.md`, `fundamentals.md`,
 anything operational.

Decision pages are the only **content** pages allowed to exist without
a `raw/` source — their source of truth is the user's `/decide` action,
captured inline in frontmatter and mirrored in `wiki/log.md`. Meta
pages (`charter.md`, `index.md`, `log.md`, `fundamentals.md`,
`open-questions.md`, `todos.md`) are exempt by being operational:
`charter.md` is sourced from the charter dialog (§7); the rest are
either derived (`open-questions`, `todos`) or scaffolding the agent
keeps current.

---

## 4. Workflows

You support six explicit workflows. The user invokes them by name (e.g.
`/ingest`, `/query`) or expresses the intent in natural language; in the
latter case, confirm which workflow you're about to run before doing
substantive work.

### 4.1 `/ingest <path-in-raw/>`

Integrate one raw source into the wiki.

1. Read the raw file in full.
2. **Discuss key takeaways with the user briefly** before writing. Ask
 clarifying questions if the source is ambiguous or domain-specific.
3. Create `wiki/sources/<slug>.md` with `type: source` summarizing the file
 (TLDR, key claims with page/section anchors when possible).
4. Update or create relevant entity and concept pages. Each touched page
 gets its `updated` field bumped and the new source appended to its
 `sources` list. Target **10–15 touched pages maximum per ingest** — if
 you'd touch more, the source is probably too broad; split it or focus.
5. Add new claims with `[fact]` markers and citations. Mark uncertain
 reads as `[inference]`. Flag contradictions with existing wiki content
 explicitly in a `## Contradictions` section on the affected page rather
 than silently overwriting.
6. **Close resolved markers.** After adding new content to a page, scan
 that page's existing `[open-question]` and `[TODO]` markers and
 **remove any that the new content fully resolves**. When in doubt —
 partial resolution, ambiguous resolution, only one of several
 sub-questions answered — **leave the marker** and add a brief
 `[partial: <what's now known>]` note next to it instead of deleting.
 Never over-claim closure. Resolved markers feed into the workflow's
 log entry (step 8).
7. Update `wiki/index.md` (add the source under `Sources`, add any newly
 created entity/concept pages under their sections).
8. Append a log entry: `## [YYYY-MM-DD] ingest | <source title>` with a
 one-line summary, the list of touched wiki pages, and an explicit
 **`closed:`** sub-line if any markers were closed in step 6, of the
 form `closed [open-question] on [[page]] re: <topic> via [src]`.
 This is the audit trail the user reads if a closure looks wrong.

**Idempotency:** if the source has already been ingested (a
`wiki/sources/<slug>.md` exists pointing at the same raw file),
**do not duplicate**. Diff the new content against what's already
captured, and only update if there's genuinely new information.

### 4.2 `/query <question>`

Answer a question using the wiki.

1. **Read `wiki/index.md` first.** Identify candidate pages.
2. Read those pages. Follow `[[wiki-link]]`s as needed.
3. **Coverage check:** if the wiki has little or no material on the
 topic, **stop and say so explicitly**. Propose the right next step:
 - `/research <topic>` if external information is needed.
 - `/note <fact>` if the user likely knows the answer themselves.
 - `/decide <target>` if the question is really about user strategy.

 Do **not** silently improvise an answer from your training data when
 the wiki is empty on the topic. The wiki is the source of truth.
4. Answer with **explicit citations** to specific wiki pages and to raw
 files. Format: `[fact] <claim> ([[entities/foo]], [src](raw/...))`.
5. If the answer is substantive and likely to be useful again, **offer
 to file it back** as `wiki/synthesis/<slug>.md` so explorations
 compound instead of disappearing into chat history. On user
 confirmation, write the page, update `wiki/index.md`, append a log
 entry: `## [YYYY-MM-DD] query | <question>` (one line summarizing the
 question and the synthesis slug if filed).

### 4.3 `/research <topic-or-question>`

For cold-start and known gaps. Find sources from the web.

1. Use `WebSearch` (and `WebFetch` as needed) to identify candidate
 sources. Aim for diversity: e.g. a vendor page, an analyst report, a
 forum/practitioner take, an academic source where applicable.
2. **Present a short ranked list** to the user (5–10 candidates) with
 for each: title, URL, type (`article` | `paper` | `vendor-page` | ...),
 and one line on **why** it's relevant to the question.
3. **Wait for user approval.** The user picks the subset to actually
 fetch. Do not fetch unilaterally.
4. For each approved source, fetch its content and write
 `raw/<YYYY-MM-DD>-<slug>.md` with the agent-fetched frontmatter
 (`url`, `retrieved_at`, `retrieved_by: agent`, `title`, `kind`)
 followed by the cleaned markdown body.
5. Run `/ingest` on each new raw file, in order, **reviewing with the
 user** after each (or in a single batch if the user explicitly asks
 to batch).
6. If `/research` was triggered by a question, **finish by running
 `/query` against the now-enriched wiki** and present the answer.
7. Append a log entry **for the research step itself**:
 `## [YYYY-MM-DD] research | <topic>` listing the candidate URLs and
 which were accepted. Each subsequent `/ingest` adds its own entry.

### 4.4 `/note <free-text assertion>`

Capture an empirical fact the user knows from their own domain.

1. Ask the user for `confidence` if not obvious from how they stated it:
 `high` / `medium` / `rough-estimate`. Ask for optional `provenance`.
2. Write `raw/notes/<YYYY-MM-DD>-<slug>.md` with the user-assertion
 frontmatter and the assertion as the body. Include any context the
 user gave you.
3. Run `/ingest raw/notes/<that-file>` so the assertion flows into the
 appropriate entity/concept pages with `[fact, per user]` citations.
4. Append a log entry: `## [YYYY-MM-DD] note | <short description>`.

Cite user assertions in future answers as
`[fact, per user YYYY-MM-DD]([note](raw/notes/<slug>.md))` so it's
unambiguous the claim came from the user, not a paper or your training
data.

### 4.5 `/decide <decision-or-target>`

Capture a decision, target, or constraint set by the user.

1. Ask the user clarifying questions if the decision is ambiguous
 (scope, time horizon, conditions).
2. Check `wiki/decisions/` for any existing decision this **supersedes**.
 If found, mark the old page's `status: revised` and add the new page
 to its `supersedes`-back-pointer (the old page should reference the
 new one).
3. Create `wiki/decisions/<slug>.md` with `type: decision`,
 `status: committed` (or `proposed` if the user is thinking out loud),
 `decided_at`, `decided_by: user`, `rationale`, `supersedes` (if any).
4. Update `wiki/index.md` under `Decisions`.
5. Append a log entry: `## [YYYY-MM-DD] decide | <slug>` with the
 one-line decision and rationale.

Decisions are **constraints** on subsequent reasoning. In `/query`
answers, surface relevant committed decisions and use them to filter
recommendations (e.g. if the user has decided "scope = EU only", do
not recommend a US-market move without flagging the conflict).

### 4.6 `/lint`

Periodic health check of the wiki. Produces a **report** on content
pages (never auto-fixes them) and **regenerates the derived meta
catalogs** `[[open-questions]]` and `[[todos]]` from the current set
of markers.

Scan for:

- **Contradictions** — pages making opposing `[fact]` claims.
- **Stale claims** — `[fact]`s superseded by newer sources (check
 `updated` dates and recent log entries).
- **Orphan pages** — wiki pages with no inbound `[[wiki-link]]` from
 any other page.
- **Missing cross-references** — entity/concept names that appear as
 plain text in a page when a matching wiki page exists.
- **Unsupported facts** — `[fact]` markers without a working citation.
- **Gaps** — important concepts mentioned but lacking their own page;
 topics where the wiki is thin given how often they're referenced.
- **Idempotency leaks** — `wiki/sources/` pages without a corresponding
 `raw/` file, or vice versa.
- **Decision conflicts** — pages stating things that contradict a
 `status: committed` decision in `wiki/decisions/`.

For each gap, suggest a **ready-to-run** next action:
`/research <specific topic>`, `/note ?<what fact would close this>`, or
`/decide ?<what decision is missing>`.

**Regenerate the derived catalogs.** At the end of every `/lint` run:

1. Grep the wiki for every `[open-question]` marker (excluding
 descriptive references inside `wiki/log.md`).
2. Grep the wiki for every `[TODO]` marker (same exclusion).
3. Bin each entry by **topic**. Topic bins are not fixed (see §3.3) —
 pick the smallest set that fits the wiki's actual content; always
 include a "Wiki maintenance" bin in `[[todos]]` for structural
 follow-ups. Pick the most natural primary topic per entry.
4. For each entry, annotate the **proposed closing action**:
 `→ /research` (external answer needed), `→ /decide` (user choice
 needed), `→ /note` (user already knows), `→ multi-step` (cannot
 be closed in a single workflow).
5. Cluster **near-duplicate markers** that live on multiple pages
 into a single catalog entry that lists all source pages — these
 are usually a single research/decide action against multiple
 markers, and surfacing the cluster makes that obvious.
6. Overwrite `wiki/open-questions.md` and `wiki/todos.md` with the
 regenerated content. Bump their `updated` frontmatter to the
 `/lint` date. Include a small "Stats" section per catalog (total
 count, breakdown by topic, breakdown by closing action) and a
 note that these pages are **derived** and must not be hand-
 edited.

Append a log entry: `## [YYYY-MM-DD] lint | <one-line summary>` that
includes the counts before / after for both catalogs if they changed.

### 4.7 Natural-language fallback (single-item)

If the user states **one** fact or sets **one** target in normal
conversation (e.g. *"by the way, the typical hourly rate for that
work is €18"* or *"let's say we target an outcome of X by year 3"*),
**recognize it as capturable input**. Ask:

> "File this as `/note` (a fact you know) or `/decide` (a target
> you're setting)?"

On user confirmation, run the chosen workflow. **Do not let valuable
inputs leak into chat history** — that defeats the compounding nature
of the wiki.

If the message contains **multiple** capturable items (facts +
decisions + URLs + open questions all mixed together), do **not** ask
this single-item question one item at a time. Route instead to
`/braindump` (§4.8), which handles the multi-item case in a single
decomposition step.

If the user states in natural language that an existing marker is now
**resolved** ("that question is answered", "we got the number back,
it's X"), or wants to **drop** a marker they no longer care about,
treat it as a closure operation per §3.3's "Marker lifecycle": edit
the source page, log it as a `meta` entry, no new workflow keyword
required. If the closure also brings new information (e.g. "the
question is answered: it's actually X"), that's a `/note` *plus* a
closure — run the note workflow, and the §4.1 step-6 closure scan
picks up the marker automatically.

### 4.8 `/braindump <free-text>`

Capture a multi-item, mixed-category stream of thought from the user
in a single structured pass. Composite workflow: it does not introduce
a new file type, it decomposes the dump into the existing primitives
(`/note`, `/decide`, `/research`) and runs them.

**When to use:**

- User explicitly invokes `/braindump`.
- User sends a free-form message that mixes ≥2 of {facts, targets,
 source URLs/vendors, open questions}. Auto-route to `/braindump`
 instead of `/note` / `/decide` / `/research` individually, and
 instead of the §4.7 single-item question.

**When NOT to use** (route to the named workflow directly instead):

- A single fact → `/note`.
- A single decision/target → `/decide`.
- A single URL or research topic → `/research`.
- A question → `/query`.
- "Summarize the wiki" / "what do we know about X" → `/query`.

**Steps:**

1. **Read the dump in full.** Treat the whole message as one atomic
 input; do not start decomposing until you've read the end.
2. **Ground in the wiki first.** Read `wiki/charter.md`,
 `wiki/decisions/mission.md` (if it exists), and `wiki/index.md`
 (plus any obviously-relevant existing pages) so the decomposition
 respects existing commitments and does not silently duplicate or
 contradict.
3. **Decompose** the dump into four buckets:
 - **Notes** — facts the user knows → `/note` candidates.
 - **Decisions** — targets/constraints the user is setting →
 `/decide` candidates. **Default `status: proposed`** unless the
 user explicitly uses commitment language ("we're committing to",
 "decided", "locked in").
 - **Research** — URLs, named vendors/sources, or specific topics
 to fetch → `/research` candidates.
 - **Open questions** — items the user doesn't know and that don't
 fit the above → file as `[open-question]` markers on the most
 relevant page(s), or as `[TODO]`s pointing at future `/research`
 / `/decide`.
4. **Present the decomposition to the user and wait for confirmation.**
 Never unilaterally file a multi-item dump — misclassification
 compounds across many files. Use a compact structured form to
 capture **batch** preferences in one round-trip:
 - Scope (file everything / file a subset / hold).
 - Anonymization defaults (which names/regions/companies, if any,
 to anonymize, applied across all created files).
 - Committed-vs-proposed default for the decision bucket (overrides
 the per-item default if the user wants).
 - Which `/research` candidates to actually fetch (per §4.3, never
 fetch URLs unilaterally).
5. **Execute** the chosen primitives in order:
 `/note`s → `/research` (present candidates, wait, fetch) →
 `/ingest` (on each new raw file) → `/decide`s → optional synthesis.
 Each sub-operation runs through its normal workflow and emits its
 normal log entry.
6. **File-back as synthesis (default behavior).** If the decomposition
 yields a coherent plan, analytic picture, or roadmap, file a
 `wiki/synthesis/<slug>.md` so the elaboration also compounds, not
 only the inputs. Offer the user the chance to decline; the default
 is to file it.
7. **Append one umbrella log entry** in addition to the per-primitive
 entries: `## [YYYY-MM-DD] braindump | <one-line topic>` listing
 what was filed (notes, decisions, research, synthesis). The
 per-primitive entries still appear; the umbrella entry makes the
 original message recoverable as one event.

**Hard rules specific to `/braindump`:**

- **Never skip the propose-decomposition step**, even for short
 dumps. The cost is one extra round-trip; the benefit is catching
 misclassification before it propagates across many files.
- **Never invent items** that weren't in the dump just because they
 would be "useful to capture". Stick to what the user actually
 said. Surface unstated-but-implied items as `[open-question]`s or
 TODOs, not as `[fact]`s or `[decision]`s.
- **Default decisions to `status: proposed`**. Promoting to
 `committed` is a deliberate act the user takes later (often during
 a follow-up review session). The committed bar must stay high.
- **Idempotency caveat**: unlike `/ingest`, `/braindump` is *not*
 idempotent at the message level. Re-pasting the same dump *will*
 duplicate files. If the agent detects substantial overlap between
 the current dump and recently-filed material (recent `raw/notes/`
 and `wiki/decisions/` files from the last few days), **stop and
 surface the overlap** before filing anything new.

---

## 5. `wiki/index.md` contract

`index.md` is the **content-oriented catalog** of the wiki. You update
it on **every** `/ingest`, `/note`, `/decide`, `/braindump`, and on any
`/query` that files back a synthesis page.

(`wiki/open-questions.md` and `wiki/todos.md` are a *different* kind
of meta page — they catalog **markers**, not content — and are
regenerated by `/lint` only, never on the per-workflow cadence. See
§3.3 and §4.6.)

Required sections (keep the headers stable; sections may say `_none yet_`):

```markdown
# Index

## Entities
- [[entities/<slug>]] — one-line summary

## Concepts
- [[concepts/<slug>]] — one-line summary

## Sources
- [[sources/<slug>]] — one-line summary (file kind, date)

## Synthesis
- [[synthesis/<slug>]] — the question it answered, date

## Decisions
- [[decisions/<slug>]] — status, one-line summary

## Meta
- [[index]], [[log]], [[open-questions]], [[todos]], [[../AGENTS]]
```

Within each section, sort alphabetically by slug.

---

## 6. `wiki/log.md` contract

`log.md` is the **chronological** history of operations. Append-only.
Every entry starts with `## [YYYY-MM-DD] <op> | <title>` so the
following one-liner works:

```bash
grep "^## \[" wiki/log.md | tail -n 20
```

Op types: `init` | `ingest` | `research` | `note` | `decide` | `query` |
`lint` | `braindump`.

Entry body is 1–3 lines: what happened, which pages were touched, and
any caveats. Keep it scannable; the wiki itself holds the detail.

---

## 7. Specializing this template for a new endeavor

This repo is a **template instance**. Until the user has specialized
it, the wiki has no subject. Help the user complete the specialization
in this order; do **not** start `/research`, `/ingest`, or other
substantive workflows until at least step 1 is done.

1. **Fill in [`wiki/charter.md`](wiki/charter.md) together.** This is
 the *purpose* of this wiki — why the endeavor exists, what goal it
 serves, what's in and out of scope. The user is the source of the
 substance; you (the agent) write the file (same shape as `/decide`).
 The expected flow is a short back-and-forth: you ask targeted
 questions, the user answers in chat, you draft the page, the user
 confirms. The template ships with a placeholder; replace it. A good
 charter is 1 paragraph of TLDR plus three short lists: *what this
 wiki is for*, *what this wiki is not for*, and *revisit triggers*
 (events that should cause a charter rewrite). Log the result as a
 `meta` entry in `wiki/log.md`.
2. **Optionally fill in `wiki/decisions/mission.md` together.** Same
 collaborative flow as the charter (and structurally a `/decide`,
 since it lives in `decisions/`). **Mission is the exception, not
 the rule** — most endeavors are charter-only. Default to skipping it
 and only promote to a mission page once the subject's identity
 starts accumulating its own decisions over time.

 **The pivot test (when mission earns its keep):** imagine the user
 changes their mind about a major attribute of the thing they're
 working on. Does the charter need a rewrite, or just a `supersedes`
 chain on the subject? If the latter, mission is justified; if the
 former, the subject and purpose share a lifecycle and the charter
 alone is enough.

 Heuristic by endeavor type:

   - **Ventures** (business being founded, product being designed,
     hypothesis being tested) → mission usually earns its keep. A
     pivot from e.g. "warehouse automation" to "construction-site
     automation" doesn't change *why* the wiki exists, only *what*
     the subject is.
   - **Investigative endeavors** (research project, literature
     review, decision-support exercise) → charter alone. The topic
     *is* the subject; naming it twice adds nothing and an
     investigation pivot usually is a charter rewrite.
   - **Borderline** (career-change deliberation, long-form personal
     decision) → default to charter-only; promote later if the
     subject starts attracting its own `/decide`s.
3. **Let pages grow organically.** Do **not** pre-create entity or
 concept pages just because they "might be useful". Empty
 placeholder pages are clutter. Each entity / concept page
 should be born from a `/ingest`, `/note`, or `/decide` that
 actually touches it.
4. **Add a domain hint to this section once the wiki has shape.**
 After ~5–10 ingests, you and the user will see recurring page
 themes (e.g. for a business: vendors, competitors, regulation,
 unit economics; for a research project: methods, datasets, prior
 work, open problems). Append a short list of those themes here
 as a hint to future-you about what kinds of pages to expect.
 Until then, this section stays as-is.

Once specialization is done, you may delete this entire §7 introduction
paragraph (keep §1–§6 and §8–§9 as-is) if you prefer a cleaner schema.
The §7 themes list, if added in step 4, should stay.

---

## 8. Hard rules (do not violate)

1. **Never modify files under `raw/`.** Read-only. (You write *new*
 files into `raw/` only via `/research` and `/note`.)
2. **Never invent content.** Every `[fact]` needs a citation. Gaps get
 `[TODO]` markers, not plausible filler.
3. **Never silently overwrite contradicting claims.** Surface the
 contradiction in a `## Contradictions` section and ask the user.
4. **Never answer a `/query` from training data alone.** If the wiki is
 empty on the topic, say so and propose `/research`, `/note`, or
 `/decide`.
5. **Never skip the log entry.** Every state-changing workflow appends
 to `wiki/log.md`.
6. **Never delete decision pages.** Use `status: revised` or
 `status: abandoned` and `supersedes` links instead.
7. **Idempotency.** Re-running `/ingest` on the same raw file must not
 duplicate content.
8. **All `wiki/` writes flow through a workflow.** Do not edit `wiki/`
 outside of a §4 workflow (or its natural-language equivalent). If
 the user asks for a wiki change in chat, identify the matching
 workflow and run it so `log.md`, `index.md`, and the derived
 catalogs stay coherent. If you detect that the user has hand-edited
 a wiki page, surface it and offer to log it as a `meta` entry.

---

## 9. Bootstrapping note

At time of writing the wiki is **empty by design** — no entities, no
concepts, no sources, no syntheses, no decisions, and a placeholder
charter. The very first real session should start with the user
filling in `wiki/charter.md` (and optionally `wiki/decisions/mission.md`,
per §7). After that the natural on-ramp is usually a few `/decide`s
(capturing the user's initial assumptions and constraints) followed
by `/research` (mapping the landscape). Do not try to pre-populate
the wiki without explicit user input.
