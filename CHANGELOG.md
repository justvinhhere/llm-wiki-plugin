# Changelog

## 0.1.3 — 2026-04-20

Scope reduction: drop the git-operation skill wrappers.

- Removed `/llm-wiki:blame` and the `wiki-blame` skill. Use `git blame`, `git log -S`, or `git log --follow` directly — commit scopes (`ingest(<slug>)`, `query(<slug>)`) already carry the provenance pointer.
- Removed `/llm-wiki:revert` and the `wiki-revert` skill. Use `git revert <sha>` directly.
- Removed `revert` from `.llm-wiki.yaml:commit.scopes` and deleted the `revert:` config block. Existing wikis with `revert(…)` commits in history are unaffected; new reverts authored by hand can use any scope the user prefers.
- README "How git is used" section removed. Git is still the substrate; it no longer needs its own marketing section now that the only skills that surfaced it are gone.

## 0.1.2 — 2026-04-20

Bootstrap stubs extracted to standalone files — single source of truth.

- New `stubs/` directory at plugin root with `gitignore`, `llm-wiki.yaml`, `CLAUDE.md`, `README.md`, `index.md`. `wiki-bootstrap` now copies these files verbatim at `/wiki-init` time instead of reading them from inline blocks inside `skills/wiki-bootstrap/reference.md`.
- `OBSIDIAN.md` at the plugin root is now the canonical copy used for both GitHub browsing and for the wiki-level `OBSIDIAN.md` that `/wiki-init` installs. Drift eliminated.
- `OBSIDIAN.md` rewritten to be audience-neutral so the same file reads correctly whether you find it on GitHub or inside a wiki.
- `skills/wiki-bootstrap/reference.md` shrunk to a short source → destination mapping table.

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
