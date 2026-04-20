# Using Obsidian with llm-wiki

Obsidian is not required, but the llm-wiki format maps cleanly onto it: `[[wikilinks]]` are native, the graph view visualizes the page network, and Dataview exposes frontmatter as queryable data. This guide covers setup, recommended plugins, required settings, and the division of labor between Obsidian and Claude Code.

---

## Setup

1. Install [Obsidian](https://obsidian.md/).
2. In Obsidian, choose **Open folder as vault** and pick your **wiki repository** (the one you created with `/llm-wiki:wiki-init`) — not the plugin repo.
3. Obsidian creates `.obsidian/` inside the vault. The plugin's seed `.gitignore` already excludes `.obsidian/workspace*` and `.obsidian/cache`; your theme and plugin choices live in `.obsidian/` and can be committed if you want them versioned.

## Required settings

Open **Settings → Files & Links**:

- **Use [[Wikilinks]]** → **ON**. The plugin writes `[[page-slug]]` everywhere; Obsidian must read and generate the same syntax.
- **New link format** → **Shortest path when possible**. Matches the plugin's resolution algorithm (basename match, category disambiguation only when ambiguous).
- **Default location for new notes** → any reasonable choice. The plugin does not use this; hand-created notes land wherever you pick.
- **Automatically update internal links** → **OFF**. The plugin tracks links via `[[slug]]`, not file paths; Obsidian's automatic rename-on-move rewrites files silently and collides with the plugin's commit invariant.

## Recommended core plugins

All ship with Obsidian; enable under **Settings → Core plugins**.

- **Graph view** — `[[links]]` rendered as a graph. Invaluable for spotting orphan clusters and dense concept neighborhoods.
- **Backlinks** — side panel listing every inbound reference to the current page. Works with or against `/llm-wiki:blame`; use backlinks for "who cites this concept", `/llm-wiki:blame` for "which source introduced this line".
- **Outline** — in-page navigation.
- **Templates** — optional; the plugin ships its own templates under `templates/` and applies them during `/llm-wiki:ingest`, so this is only useful if you want Obsidian's in-app template insertion for rare manual cases.
- **Tag pane** — useful with the plugin's frontmatter `tags:`.

## Recommended community plugins

Install via **Settings → Community plugins → Browse**.

- **Dataview** — essential. Exposes frontmatter as queryable tables.

  Example queries to keep in dedicated pages:

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

- **Obsidian Git** — commit and pull from inside Obsidian. Handy when you've made a small hand-edit and don't want to switch to a terminal. For plugin-driven edits, Claude Code commits on its own, so this is purely for manual work.

- **Dataview** queries make the dynamic "what's stale", "what's tagged X", "what's unlinked" views trivial. The plugin's `/llm-wiki:lint` gives you the audit; Dataview gives you the browseable view.

Optional extras:

- **Templater** — power-user templating; overlaps somewhat with the plugin's `templates/`, but handy if you want keyboard shortcuts to insert custom blocks.
- **Tag Wrangler** — rename tags across the vault safely.
- **Excalidraw** — sketch diagrams inline; useful for topic pages that benefit from a visual.

## Workflow: who does what

| Task | Tool |
| --- | --- |
| Read / navigate the wiki | Obsidian |
| Visualize the page graph | Obsidian (Graph view) |
| Browse by tag / status / type | Obsidian (Dataview) |
| Small hand-edits (typos, clarifications) | Obsidian |
| Commit hand-edits | Claude Code `/llm-wiki:review` |
| Ingest a new source | Claude Code `/llm-wiki:ingest` |
| Ask a question against wiki content | Claude Code `/llm-wiki:query` |
| Trace provenance of a claim | Claude Code `/llm-wiki:blame` |
| Narrate a page's history | Claude Code `/llm-wiki:history` |
| Structural audit | Claude Code `/llm-wiki:lint` |
| Undo an ingest | Claude Code `/llm-wiki:revert` |

The division is: **Obsidian is a reader and micro-editor; Claude Code is the agent that ingests, synthesizes, audits, and maintains.** Both operate on the same git-tracked filesystem and don't step on each other.

## Anti-patterns

- **Don't edit files under `raw/` in Obsidian.** `raw/` is immutable by convention — re-processing a source means ingesting it again under a new slug, not editing the original.
- **Don't hand-edit `index.md` in Obsidian.** The plugin's `/llm-wiki:ingest` and `/llm-wiki:query` keep it in sync; manual edits create drift that `/llm-wiki:lint` will then flag.
- **Don't use Obsidian Sync for the wiki.** The wiki is git-versioned; use git for multi-device sync. Obsidian Sync creates a parallel sync channel that won't replay through `/llm-wiki:` commands and loses provenance.
- **Don't rename or move wiki pages in Obsidian** without running `/llm-wiki:review` immediately after to catch the resulting `[[dead-link]]` findings. Safer: rename via a normal hand-edit + `/llm-wiki:review`, or just leave the old filename (the wiki is append-only in practice — superseded pages become `status: deprecated`, not deleted).
- **Don't hand-edit frontmatter `sources:` or `contradictions:`** unless you know what you're doing. These fields have schema invariants enforced by `/llm-wiki:lint`.

## Tips

- The plugin's `OBSIDIAN.md` stub installed by `/llm-wiki:wiki-init` lives inside your wiki repo, alongside `CLAUDE.md`, and serves as a quick-reference once you're in the vault.
- When browsing in Obsidian, use the Graph view filters to show only `wiki/topics/` nodes to get a high-level subject map of what the wiki covers.
- `contradictions:` frontmatter entries are the right place to flag "I don't believe this claim" during a read — Obsidian lets you edit that field inline; then `/llm-wiki:lint` surfaces them for resolution.
