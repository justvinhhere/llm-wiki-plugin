# Changelog

## 0.1.1 — 2026-04-20

Docs + bootstrap refinements.

- New `OBSIDIAN.md` in the plugin repo: setup, required settings, recommended plugins (Dataview, Obsidian Git), division of labor between Obsidian and Claude Code, anti-patterns.
- `wiki-bootstrap` now writes an `OBSIDIAN.md` into the user's wiki on `/wiki-init` alongside `CLAUDE.md`.
- Fixed command references in the bootstrapped `CLAUDE.md`: invocations now use the `/llm-wiki:<name>` fully-qualified form, with the short form noted for unambiguous cases.
- `README.md` rewrite: "How git is used" replaces the earlier comparison framing; added Obsidian section; commands table updated to the qualified form.

## 0.1.0 — 2026-04-20

Initial release.

- 8 skills: `wiki-bootstrap`, `wiki-ingest`, `wiki-query`, `wiki-review`, `wiki-blame`, `wiki-history`, `wiki-lint`, `wiki-revert`.
- 8 slash commands: `/wiki-init`, `/ingest`, `/query`, `/review`, `/blame`, `/history`, `/lint`, `/revert`.
- 5 page templates: `entity`, `concept`, `source`, `topic`, `query`.
- Schema version: 1.
- No hooks, no MCP, no shipped scripts (guidance-over-scripts design).
