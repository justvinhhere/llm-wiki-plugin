---
name: wiki-ingest
description: Use when the user runs /ingest <path-or-url>, says "add this source", "ingest this article", "read and file this", or drops a URL/PDF/markdown file for incorporation into the wiki. Reads a source in full, executes all page edits, and commits with an ingest(<source-slug>) subject — fully automatic. Conventions, frontmatter schema, slug rule, and link resolution live in reference.md.
---

# wiki-ingest

Writes a new source into the wiki and every page it substantiates. One invocation produces exactly one commit.

## Refuse conditions

1. No `.llm-wiki.yaml` in CWD or ancestors.
2. `.llm-wiki.yaml:schema_version != 1`.
3. `git status --porcelain -- wiki/ index.md` non-empty — working tree dirty on wiki paths. The caller must commit or stash before ingesting.
4. Source unreachable (URL returns non-2xx; path does not exist).
5. Source slug collides with an existing `wiki/sources/<slug>.md` whose `url:` or `raw_path:` matches — the source is already ingested. To re-ingest a revised version, the caller picks a versioned slug explicitly.

## Workflow

### 1. Park the source in `raw/` (immutable)

Derive `<slug>` per the rule in `reference.md`. Route by kind using `.llm-wiki.yaml:ingest.raw_layout`:

| Kind | Writes |
| --- | --- |
| `.md` / `.txt` local | `<raw_layout.article>/<slug>.<ext>` |
| URL (HTML) | `<raw_layout.article>/<slug>.html` **and** `<raw_layout.article>/<slug>.md` (markdown extraction) |
| `.pdf` | `<raw_layout.paper>/<slug>.pdf` **and** `<raw_layout.paper>/<slug>.md` (best-effort transcript) |
| transcript (`.vtt`, `.srt`, video URL) | `<raw_layout.video>/<slug>.md` |
| other | `<raw_layout.note>/<slug>.<ext>` + markdown alongside |

The chosen path is `raw_path` in `sources/<slug>.md` frontmatter. `raw/` is immutable after this step.

### 2. Read the source in full

Complete read. Partial reads are forbidden — they produce partial claims that bake into blame history.

### 3. Compute the edit set

Build, internally, the list of file mutations:

- **New** `wiki/sources/<slug>.md` from `templates/source.md`, frontmatter filled.
- **New** entity / concept / topic pages from the matching template, where the source introduces a subject the wiki does not yet cover. Cap `ingest.tldr_bullets` does not apply to this list — ingest creates as many pages as the source substantively introduces.
- **Updates** to existing entity / concept / topic pages the source substantiates. Each update:
  - Bumps `updated:` to today.
  - Appends `<slug>` to `sources:` (append-only; never replace).
  - Adds prose under the appropriate section with `[[…]]` cross-references resolved per `reference.md`.
  - If the source contradicts an existing claim, appends `{source: <slug>, note: "<one-sentence tension>"}` to `contradictions:` instead of modifying the claim body. Resolution is left to `/lint`.
- **Update** `index.md`: new entries under their category, alphabetical within category.

### 4. Execute the edit set

Single batch. Any write failure (permission, disk, invalid frontmatter) aborts the entire ingest and rolls back partially-written `raw/` artifacts.

### 5. Commit

```
git add <explicit-paths-from-edit-set>
git commit -m "ingest(<slug>): add source, update N page(s)"
```

N from `git diff --cached --numstat | wc -l` minus the source page itself. Singular `page` when N=1.

Commit body (optional): up to `ingest.tldr_bullets` TL;DR bullets from the source. Useful for `git log --grep`; not required.

No `Co-Authored-By` trailer.

### 6. Output

Single line: `Ingested <slug>: N page(s) updated. Commit <short-sha>.`
