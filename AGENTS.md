# LLM Wiki — Schema

This repo is an **LLM Wiki**: a persistent, LLM-maintained knowledge base,
built on the pattern described in [Karpathy's LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

This file is the schema for this repo: it tells you (the agent) how the wiki
is structured, what conventions to follow, and what to do for the three
operations — **ingest**, **query**, **lint**.

This is a template — you and the user should adapt the specifics (directory
layout, page types, frontmatter fields) as the domain becomes clear,
and update this file when conventions change.

This wiki is **incrementally built and kept current**. When a new source comes in,
the agent reads it, extracts what matters, and integrates it into existing pages -
updating concept pages, revising summaries, flagging contradictions with prior claims,
and strengthening cross-references. The wiki compounds over time instead of resetting
on every question.

You (the agent) own the wiki layer entirely: you create pages, update them,
cross-link them, and keep them consistent. The user is editor-in-chief, he
curates sources, directs the work, and asks the questions.

## The three layers

1. **`raw/`** - The source of truth. Source documents: articles, papers,
   clipped web pages, PDFs, screenshots, transcripts, journal entries, images,
  data files. Immutable — read from it, never edit it.
2. **`wiki/`** — everything you (the LLM) write: summaries, entity pages,
   concept pages, source pages, comparisons, an overview, an evolving synthesis -
   whatever page types fit the domain. The user reads this layer; you own and
   maintain it. Freely create, edit, and restructure this layer as the wiki grows.
3. **This file (`AGENTS.md`)** — the schema. Update it as conventions evolve; it
   should always reflect current practice, not aspirational structure.

If `raw/` or `wiki/` don't exist yet, create them on first use rather than
assuming a structure. The directory layout under `wiki/` should emerge from
what gets ingested and from the domain itself — e.g. `wiki/people/`,
`wiki/concepts/`, `wiki/sources/`, `wiki/timeline/`, or something else entirely.
Once a layout is established, record it in this file, 'Directory Layout' section
below and keep this information accurate when the layout changes.

## Directory layout

```markdown
raw/                  curated source documents (immutable)
  assets/             downloaded images referenced by sources
wiki/
  overview.md          top-level synthesis / current thesis (create on first ingest)
  entities/           one page per person, org, product, place, etc.
  concepts/           one page per idea, theme, or recurring topic
  sources/            one summary page per raw source, same stem as the raw file
templates/            starter frontmatter/section shapes for new pages
index.md            catalog of every page in the wiki
log.md              append-only chronological record of activity
```

Add new subdirectories under `wiki/` when a new page type earns one (e.g.
`wiki/comparisons/`, `wiki/characters/`, `wiki/companies/`) — don't force
everything into entities/concepts if the domain wants something else. If you
add a directory, note it here.

## Page conventions

- **Filenames**: kebab-case, matching the page title (`grover-cleveland.md`,
  not `Grover Cleveland.md`). Knowledge notes should be concept-based,
  meeting notes should be date-based.
- **Links**: use relative markdown links (`[label](relative/path.md)`), not
  Obsidian-style `[[wiki-links]]`. Link liberally — every entity or concept
  mentioned on a page should link to its own page if one exists, or be a
  candidate for a new page if it's mentioned more than once.
- **Frontmatter**: every wiki page (not `index.md` or `log.md`) starts with
  YAML frontmatter:

  ```yaml
  ---
  title: Page Title
  type: entity | concept | source | synthesis
  tags: [tag-one, tag-two]
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  sources: [source-file-stem-one, source-file-stem-two]
  ---
  ```

  `sources` lists the source pages (by stem, matching `wiki/sources/*.md`)
  that back the claims on the page — this is what makes citations checkable.
- **Body**: short lead paragraph, then sections as needed. Prefer prose with
  inline citations like `(see [[source-name]])` over bare claims. Flag
  uncertainty explicitly rather than smoothing it over — "X claims Y, but
  [[other-source]] contradicts this" is more useful than picking one.
- Starter shapes for some page types live in `templates/` — read the
  relevant one before creating a new page of that type, but treat it as a
  starting point, not a rigid format.
- Write wiki pages in a pragmatic, to-the-point style. Make use of 'stop slop'
  skill if available. State claims directly instead of announcing them. Cut
  filler openers ("here's the thing," "it's worth noting"), adverbs, hedging,
  em dashes, and formulaic contrasts ("not X, it's Y"). Use active voice with
  a real subject doing the action, not passive constructions or inanimate
  things performing human verbs. Say the specific thing instead of a
  vague, self-important declarative ("the implications are significant").
  When editing or reviewing a page, remove AI slop on sight rather than
  leaving it for a later pass.

## Operations

### Ingest

Triggered when the user adds a file to `raw/` (or pastes/describes a new
source) and asks you to process it. Prefer ingesting one source at a time
with the user involved, unless told to batch.

1. Read the new source in full.
2. Discuss key takeaways with the user before writing anything, unless
   they've asked for unsupervised batch ingestion.
3. Write a source summary page in `wiki/sources/` (same stem as the raw
   file), with frontmatter `type: source`.
4. Update every entity/concept page the source touches — add new facts, flag
   contradictions with existing claims instead of silently overwriting
   them, extend cross-references. A single source can reasonably touch
   many pages; don't stop at the first page you'd naturally think of.
5. Create new entity/concept pages for anything mentioned that doesn't have
   one yet and is likely to recur.
6. Update `index.md` with any new or changed pages.
7. Update `wiki/overview.md` if the source shifts the overall synthesis.
8. Append an entry to `log.md` (format below).

### Query

Triggered when the user asks a question against the wiki.

1. Read `index.md` first to find candidate pages — don't re-derive
   from `raw/` unless the wiki doesn't yet cover the question.
2. Read the relevant pages (and their linked pages, one hop, if needed).
3. Synthesize an answer with citations back to wiki pages (and through them
   to sources). Answers can take different forms — a markdown page, a
   comparison table, a slide deck, a diagram — depending on what's asked.
4. If the answer is substantial — a comparison, an analysis, a non-obvious
   connection — offer to file it back into the wiki as a new page (in
   `wiki/concepts/` or a new type-specific directory) rather than letting it
   live only in chat history. Update the index and log if you do.

### Lint

Triggered periodically or on request. Check for:

- Contradictions between pages that weren't flagged when introduced.
- Stale claims a newer source has superseded but an older page still states
  as current.
- Orphan pages with no inbound `[[links]]`.
- Concepts mentioned repeatedly across pages but lacking their own page.
- Missing cross-references (A mentions B by name, B has a page, but no
  link).
- Gaps a targeted web search or a request for a specific source could fill.

Report findings to the user before making sweeping changes; fix
uncontroversial ones (missing links, obvious staleness) directly and log
them.

## index.md format

`index.md` is a catalog, organized by category, one line per page:

```markdown
## Entities
- [[page-name]] — one-line summary (updated YYYY-MM-DD)

## Concepts
- [[page-name]] — one-line summary (updated YYYY-MM-DD)

## Sources
- [[page-name]] — one-line summary (ingested YYYY-MM-DD)
```

## log.md format

Append-only. Every entry starts with a consistent, greppable prefix:

```markdown
## [YYYY-MM-DD] ingest | Source Title
Touched: [[page-one]], [[page-two]], [[page-three]]
Notes: one or two lines on what changed and why it mattered.

## [YYYY-MM-DD] query | Short question paraphrase
Filed: [[new-page-if-any]]

## [YYYY-MM-DD] lint
Found: N contradictions, N orphans, N gaps. Fixed: ...
```

This keeps `grep "^## \[" wiki/log.md | tail -5` useful for "what happened
recently."

## Notes for the agent

- Never edit files under `raw/` — if a source needs correcting, note the
  correction on its `wiki/sources/` summary page instead.
- Prefer many small, well-linked pages over few large ones.
- When in doubt about whether something deserves its own page: if it's
  likely to be referenced from more than one other page, give it one.
- This is a template repo — if `wiki/`, `raw/`, and `templates/` are still
  empty (or don't exist), ask the user what domain this wiki is for before inventing page
  types. Update the "Directory layout" and page-type list above once the
  domain is clear.
- This file should evolve. As real conventions get established through
  use - directory layout, page templates, tagging scheme — update this
  document to match. It documents actual practice, not aspiration.
- Treat this file itself as a wiki page: keep it current.
- Also refine these notes as the wiki grows.
