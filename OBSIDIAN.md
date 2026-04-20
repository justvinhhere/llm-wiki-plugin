# Using Obsidian with llm-wiki

Obsidian is not required — every `/llm-wiki:` command runs without it — but the llm-wiki format maps cleanly onto Obsidian's wikilinks, graph view, and Dataview.

## Setup

1. Install [Obsidian](https://obsidian.md/).
2. **Open folder as vault** → pick the wiki repository root (the directory containing `.llm-wiki.yaml`).
3. Obsidian creates `.obsidian/` inside the vault. The seed `.gitignore` excludes the entire `.obsidian/**` directory — theme, plugin choices, and workspace state stay local and out of git.

## Required settings

Open **Settings → Files & Links**:

- **Use [[Wikilinks]]** = ON. The plugin writes `[[page-slug]]` everywhere; Obsidian must read and generate the same syntax.
- **New link format** = Shortest path when possible. Matches the plugin's resolution algorithm (basename match, category disambiguation only when ambiguous).
- **Automatically update internal links** = OFF. The plugin tracks links by `[[slug]]`, not file paths; Obsidian's automatic rename-on-move rewrites files silently and bypasses the plugin's commit invariant.

## Recommended core plugins

Ship with Obsidian; enable under **Settings → Core plugins**.

- **Graph view** — `[[links]]` rendered as a graph. Spot orphan clusters and dense concept neighborhoods.
- **Backlinks** — side panel listing every inbound reference to the current page. Complements `/llm-wiki:blame` (which answers "which source introduced this line").
- **Outline** — in-page navigation.
- **Tag pane** — browse by frontmatter `tags:`.

## Recommended community plugins

Install via **Settings → Community plugins → Browse**.

- **Dataview** — essential. Exposes frontmatter as queryable tables. Examples:

  ````
  ```dataview
  TABLE status, updated, sources
  FROM "wiki/entities"
  WHERE status != "deprecated"
  SORT updated DESC
  ```
  ````

  ````
  ```dataview
  LIST
  FROM "wiki"
  WHERE contains(tags, "ml") AND status = "stale"
  ```
  ````

  The plugin's `/llm-wiki:lint` gives you the audit; Dataview gives you the browseable view.

- **Obsidian Git** — optional. Commit and pull from inside Obsidian for quick hand-edits. For plugin-driven edits, Claude Code commits automatically.

Optional extras: **Templater** (keyboard-shortcut template insertion for custom workflows), **Tag Wrangler** (safe tag renames), **Excalidraw** (inline diagrams for topic pages).

## Workflow

| Task | Tool |
| --- | --- |
| Read / navigate the wiki | Obsidian |
| Visualize the page graph | Obsidian (Graph view) |
| Browse by tag / status / type | Obsidian (Dataview) |
| Small hand-edits (typos, clarifications) | Obsidian + `/llm-wiki:review` to commit |
| Ingest a new source | `/llm-wiki:ingest` |
| Ask a question against wiki content | `/llm-wiki:query` |
| Trace provenance of a claim | `/llm-wiki:blame` |
| Narrate a page's history | `/llm-wiki:history` |
| Structural audit | `/llm-wiki:lint` |
| Undo an ingest | `/llm-wiki:revert` |

**Obsidian = reader + micro-editor. Claude Code = agent.** Both work on the same git-tracked filesystem and don't step on each other.

## Anti-patterns

- **Don't edit files under `raw/` in Obsidian.** `raw/` is immutable by convention — re-processing a source means ingesting it again under a new slug.
- **Don't hand-edit `index.md`.** The plugin's `/llm-wiki:ingest` and `/llm-wiki:query` keep it in sync; manual edits produce drift that `/llm-wiki:lint` will then flag.
- **Don't use Obsidian Sync for the wiki.** The wiki is git-versioned; sync via git. Obsidian Sync creates a parallel channel that won't replay through `/llm-wiki:` commands and loses provenance.
- **Don't rename or move wiki pages in Obsidian** without running `/llm-wiki:review` immediately after to catch the resulting `[[dead-link]]` findings. Safer: leave superseded pages in place with `status: deprecated` — the wiki is append-only in practice.
- **Don't hand-edit frontmatter `sources:` or `contradictions:`.** These have schema invariants enforced by `/llm-wiki:lint`.

## Tips

- When browsing in Obsidian, filter the Graph view to show only `wiki/topics/` nodes for a high-level subject map of what the wiki covers.
- `contradictions:` frontmatter entries are the right place to flag "I don't believe this claim" during a read — Obsidian lets you edit that field inline; then `/llm-wiki:lint` surfaces them for resolution.
- Dataview queries live nicely inside `wiki/topics/` pages as section bodies — a topic page can carry both synthesis prose and a live Dataview of related entities.
