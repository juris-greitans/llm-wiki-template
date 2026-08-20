# llm-wiki-template

A starter scaffold for [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f):
instead of RAG over raw documents, have an LLM incrementally build and
maintain a persistent, interlinked wiki as you feed it sources.

- **`raw/`** — drop your source documents here (immutable).
- **`wiki/`** — the LLM-maintained wiki: `index.md`, `log.md`,
  `overview.md`, plus `entities/`, `concepts/`, `sources/`.
- **`templates/`** — starter shapes for new pages.
- **`AGENTS.md`** — the schema: directory conventions and the
  ingest/query/lint workflows the agent follows. This is the file to read
  (and edit as conventions evolve) before working in this repo.
  `CLAUDE.md` just points here, for agents that only look for that name.

Open this directory in your LLM coding agent, add a first source to
`raw/`, and ask it to ingest it. Obsidian works well as a companion viewer
for browsing the resulting graph.
