# wiki-bootstrap — verbatim stubs

The agent copies these files as-is during `/llm-wiki:wiki-init`. Dated fields (`created:`, today's date in `ingested:`, etc.) are filled at write-time.

---

## `.gitignore`

```gitignore
# Plugin caches
.llm-wiki-cache/

# Large or non-public raw sources
raw/assets/*.pdf
raw/**/*.mp4
raw/**/*.mov

# Obsidian workspace noise (keep plugin/theme choices if you want them versioned)
.obsidian/workspace*
.obsidian/cache

# OS noise
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

All commands are invocable as `/llm-wiki:<name>` (fully qualified) or as `/<name>` (short form, when there's no collision with other plugins).

- `/llm-wiki:ingest <path-or-url>` — read a source and file it into the wiki. Commits as `ingest(<slug>)`.
- `/llm-wiki:query <question>` — search + synthesize from wiki content. Synthesis answers auto-file as query pages and commit as `query(<slug>)`.
- `/llm-wiki:review` — audit + commit any uncommitted wiki edits with a conventional subject.
- `/llm-wiki:blame <page> [line-or-phrase]` — trace a claim back to its commit, ingest scope, and raw source passage.
- `/llm-wiki:history <page> [--diffs]` — narrate a page's commit history.
- `/llm-wiki:lint [--since <ref>]` — structural audit (orphans, dead links, stale pages, contradictions, index drift). Read-only.
- `/llm-wiki:revert [<sha>]` — back out a wiki-mutation commit via `git revert`. Defaults to the most recent ingest.

## House rules

- Every wiki mutation is committed. No silent writes.
- `raw/` is immutable after ingest. Re-processing requires a new slug; the old source page becomes `status: deprecated`.
- Cross-references use `[[page-slug]]`. Case-insensitive. Ambiguous basenames use `[[category/slug]]`.
- Hand-edits to `wiki/**/*.md` are fine; running `/llm-wiki:review` commits them with a `revise(...)` scope.
- `index.md` is plugin-maintained. Do not hand-edit it.
- Frontmatter `sources:` is append-only during ingest. Do not replace.

## Viewer

See [OBSIDIAN.md](./OBSIDIAN.md) for Obsidian setup, recommended plugins, and the Obsidian-plus-Claude-Code workflow.
```

---

## `OBSIDIAN.md` (wiki-level)

```markdown
# Using Obsidian with this wiki

This wiki is optimized for viewing in Obsidian. Obsidian is not required — every `/llm-wiki:` command runs without it — but the format maps cleanly onto Obsidian's wikilinks, graph view, and Dataview.

## Setup

1. Install [Obsidian](https://obsidian.md/).
2. **Open folder as vault** → pick this repository's root.
3. Settings → **Files & Links**:
   - **Use [[Wikilinks]]** = ON.
   - **New link format** = Shortest path when possible.
   - **Automatically update internal links** = OFF.

## Plugins

Core plugins (ship with Obsidian; enable under Settings → Core plugins):

- **Graph view** — visualize the `[[links]]` network.
- **Backlinks** — inbound references panel.
- **Outline** — in-page navigation.
- **Tag pane** — browse by `tags:` frontmatter.

Community plugins (Settings → Community plugins → Browse):

- **Dataview** — essential. Exposes frontmatter as queryable tables. Example:

  ````
  ```dataview
  TABLE status, updated, sources
  FROM "wiki/entities"
  WHERE status != "deprecated"
  SORT updated DESC
  ```
  ````

- **Obsidian Git** — optional; commit from inside Obsidian. For plugin-driven edits, Claude Code commits on its own.

## Workflow

| Task | Tool |
| --- | --- |
| Read / navigate | Obsidian |
| Visualize the graph | Obsidian (Graph view) |
| Browse by tag / type / status | Obsidian (Dataview) |
| Small hand-edits | Obsidian + `/llm-wiki:review` to commit |
| Ingest a source | `/llm-wiki:ingest` |
| Ask a question | `/llm-wiki:query` |
| Trace provenance | `/llm-wiki:blame` |
| Structural audit | `/llm-wiki:lint` |
| Undo an ingest | `/llm-wiki:revert` |

**Obsidian = reader + micro-editor. Claude Code = agent.** Both work on the same git-tracked filesystem.

## Do not

- Edit files under `raw/` — immutable after ingest.
- Hand-edit `index.md` — plugin-maintained.
- Rename or move wiki pages without running `/llm-wiki:review` after.
- Use Obsidian Sync — sync via git instead.
```

---

## `README.md` (wiki-level)

```markdown
# <wiki name>

A knowledge base maintained with Claude Code via the [llm-wiki](https://github.com/justvinhhere/llm-wiki-plugin) plugin.

## Layout

- `wiki/` — LLM-maintained markdown, grouped by `entities/`, `concepts/`, `sources/`, `topics/`, `queries/`.
- `raw/` — immutable source material.
- `index.md` — catalog, plugin-maintained.
- `.llm-wiki.yaml` — tunable conventions.
- `CLAUDE.md` — authoritative rules for the plugin and hand-edits.
- `OBSIDIAN.md` — Obsidian setup and workflow.
```

---

## `index.md`

```markdown
# Index

_Maintained by `/llm-wiki:ingest` and `/llm-wiki:query`. Grouped by category, alphabetical within each group._

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
