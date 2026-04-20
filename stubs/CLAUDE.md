# Wiki conventions

This wiki is maintained by the `llm-wiki` Claude Code plugin. Tunables live in `.llm-wiki.yaml`.

## Slash commands

All commands are invocable as `/llm-wiki:<name>` (fully qualified) or as `/<name>` (short form, when there's no collision with other plugins).

- `/llm-wiki:ingest <path-or-url>` — read a source and file it into the wiki. Commits as `ingest(<slug>)`.
- `/llm-wiki:query <question>` — search + synthesize from wiki content. Synthesis answers auto-file as query pages and commit as `query(<slug>)`.
- `/llm-wiki:review` — audit + commit any uncommitted wiki edits with a conventional subject.
- `/llm-wiki:history <page> [--diffs]` — narrate a page's commit history.
- `/llm-wiki:lint [--since <ref>]` — structural audit (orphans, dead links, stale pages, contradictions, index drift). Read-only.

## House rules

- Every wiki mutation is committed. No silent writes.
- `raw/` is immutable after ingest. Re-processing requires a new slug; the old source page becomes `status: deprecated`.
- Cross-references use `[[page-slug]]`. Case-insensitive. Ambiguous basenames use `[[category/slug]]`.
- Hand-edits to `wiki/**/*.md` are fine; running `/llm-wiki:review` commits them with a `revise(...)` scope.
- `index.md` is plugin-maintained. Do not hand-edit it.
- Frontmatter `sources:` is append-only during ingest. Do not replace.

## Viewer

See [OBSIDIAN.md](./OBSIDIAN.md) for Obsidian setup, recommended plugins, and the Obsidian-plus-Claude-Code workflow.
