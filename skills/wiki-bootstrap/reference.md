# wiki-bootstrap — verbatim stubs

The agent copies these files as-is during `/wiki-init`. Dated fields (`created:`, today's date in `ingested:`, etc.) are filled at write-time.

---

## `.gitignore`

```gitignore
# Plugin caches
.llm-wiki-cache/

# Large or non-public raw sources
raw/assets/*.pdf
raw/**/*.mp4
raw/**/*.mov

# Editor noise
.obsidian/workspace*
.obsidian/cache
.DS_Store
Thumbs.db

# Scratch
__pycache__/
node_modules/
*.pyc
```

---

## `.llm-wiki.yaml`

```yaml
schema_version: 1
wiki_root: ./wiki
raw_root: ./raw

commit:
  style: conventional
  scopes: [ingest, query, lint, revise, revert, bootstrap]

conventions:
  filename_case: kebab-case
  require_frontmatter: true

ingest:
  tldr_bullets: 5
  raw_layout:
    article: raw/articles
    paper: raw/papers
    video: raw/transcripts
    note: raw/notes

query:
  top_k_candidates: 8
  file_synthesis: true
  dedupe_against_queries: true

lint:
  default_scope: full
  stale_days: 180
  orphan_allowlist:
    - index.md
    - wiki/topics/**
  suggest_after_n_ingests: 10

revert:
  recent_commits_window: 20
```

---

## `CLAUDE.md`

```markdown
# Wiki conventions

This wiki is maintained by the `llm-wiki` Claude Code plugin. Tunables live in `.llm-wiki.yaml`.

## Slash commands

- `/ingest <path-or-url>` — read a source and file it into the wiki; commits automatically.
- `/query <question>` — search + synthesize from wiki content. Synthesis answers are auto-filed as query pages.
- `/review` — audit + commit uncommitted wiki edits with a conventional subject.
- `/blame <page> [line-or-phrase]` — trace a claim back to its commit, ingest scope, and raw source passage.
- `/history <page> [--diffs]` — narrate a page's commit history.
- `/lint [--since <ref>]` — structural audit (orphans, dead links, stale, contradictions, index drift). Read-only.
- `/revert [<sha>]` — back out a wiki-mutation commit via `git revert`. Defaults to the most recent ingest.

## House rules

- Every wiki mutation is committed. No silent writes.
- `raw/` is immutable after ingest. Re-processing requires a new slug; the old source page becomes `status: deprecated`.
- Cross-references use `[[page-slug]]`. Case-insensitive. Ambiguous basenames use `[[category/slug]]`.
- Hand-edits to `wiki/**/*.md` are fine; running `/review` commits them with a `revise(...)` scope.
```

---

## `README.md` (wiki-level)

```markdown
# <wiki name>

A knowledge base maintained together with Claude Code via the `llm-wiki` plugin.

## Layout

- `wiki/` — LLM-maintained markdown, grouped by `entities/`, `concepts/`, `sources/`, `topics/`, `queries/`.
- `raw/` — immutable source material.
- `index.md` — catalog, kept in sync during ingest and query.
- `.llm-wiki.yaml` — tunable conventions.
- `CLAUDE.md` — authoritative rules for the plugin and hand-edits.
```

---

## `index.md`

```markdown
# Index

_Maintained by `/ingest` and `/query`. Grouped by category, alphabetical within each group._

## Entities

_(empty)_

## Concepts

_(empty)_

## Sources

_(empty)_

## Topics

_(empty)_

## Queries

_(empty)_
```
