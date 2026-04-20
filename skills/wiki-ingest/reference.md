# wiki-ingest — reference

Schemas and algorithms consumed by every skill. Canonical source; other skills link here.

---

## Frontmatter schema

### Common to all page types

```yaml
---
title: Andrej Karpathy
type: entity                     # entity | concept | source | topic | query
tags: [ml, researcher]
status: active                   # stub | active | stale | deprecated
created: 2026-04-20
updated: 2026-04-20
sources: [karpathy-llm-wiki]
contradictions: []               # list of {source: <slug>, note: "..."}
---
```

Invariants:

- `updated:` equals the commit date of any commit that touches the page body or frontmatter.
- `sources:` is append-only during ingest.
- `contradictions:` entries are objects `{source: <slug>, note: "<free text>"}`. Lint rejects bare strings.
- `status:` transitions:
  - `stub` → `active` when body exceeds a single definition sentence.
  - `active` → `stale` when `today - updated > lint.stale_days`.
  - `active` → `deprecated` when a successor page supersedes. Deprecated pages remain in the wiki as redirects (body: `Superseded by [[new-page]].`); they are never deleted.

### Type-specific fields

| Type | Additional keys |
| --- | --- |
| `entity` | `kind: person \| org \| place \| tool`, `aliases: []` |
| `concept` | `parent_concepts: []`, `related: []` |
| `source` | `author:`, `published: <YYYY-MM-DD>`, `url:`, `raw_path:`, `ingested: <YYYY-MM-DD>` |
| `topic` | `synthesizes: []`, `allow_orphan: true` |
| `query` | `question:`, `asked: <YYYY-MM-DD>`, `answer_summary:`, `cites_pages: []` |

---

## Cross-reference resolution

Given `[[X]]` or `[[X|display]]`:

1. If target contains `/` (e.g., `[[concepts/memex]]`), glob `wiki/<target>.md` exactly. One hit → resolved. Zero hits → `dead-link`.
2. Otherwise: lowercase target, replace spaces/underscores with `-` → `slug`.
3. Glob `wiki/**/<slug>.md`.
   - One hit → resolved.
   - Multiple hits → `dead-link-ambiguous` (authors use explicit path form).
   - Zero hits → `dead-link`.

Relative markdown links `[text](path)` are never used for wiki-internal references.

---

## Slug derivation

Used for filenames, commit scopes, `sources:` entries, `raw_path:` suffixes.

1. Lowercase.
2. Replace every non-alphanumeric (except `-`) with `-`.
3. Collapse runs of `-` to single `-`.
4. Trim leading / trailing `-`.
5. Truncate to 60 chars at the last `-` before the cutoff.
6. Collision: if `<slug>.md` exists in the target directory with different content, append `-2`, `-3`, … Stay within the directory.

**Query-specific pre-step:** before step 1, strip stopwords from the question — `what, why, how, when, where, who, is, are, does, the, a, an, of, to, for, in, on, at, by`.

---

## Raw-layout routing

From `.llm-wiki.yaml:ingest.raw_layout`. Defaults:

| Kind | Destination |
| --- | --- |
| Article (HTML, markdown) | `raw/articles/` |
| Paper (PDF, LaTeX) | `raw/papers/` |
| Video / audio transcript | `raw/transcripts/` |
| Other | `raw/notes/` |

Kind inference:
- `.pdf` → paper.
- `.html`, URL → article.
- `.vtt`, `.srt`, YouTube URL → transcript.
- `.md`, `.txt` → article unless filename suggests otherwise.
- Anything else → note.

---

## Commit message format

Conventional Commits. Scope vocabulary from `.llm-wiki.yaml:commit.scopes`.

```
ingest(<source-slug>):  add source, update N page(s)
query(<query-slug>):    file synthesized answer
lint(<yyyy-mm-dd>):     fix N orphan(s), M dead link(s)
revise(<page-slug>):    <short why>
bootstrap:              initialize llm-wiki skeleton
```

- Singular for N=1.
- Subject ≤72 chars.
- No `Co-Authored-By` trailer.
- Authored with the user's git config.

---

## Skill-invariant contract

Post-preflight, every skill may assume:

- Wiki root reachable by walking up from CWD for `.llm-wiki.yaml`.
- `.llm-wiki.yaml:schema_version == 1`.
- Frontmatter schema above applies to every `wiki/**/*.md`.
- Every wiki-path commit subject matches the commit-message format above.

A violation of any invariant is reported as a lint finding (`unstructured-commit`, `invalid-frontmatter`, `schema-mismatch`) rather than silently worked around.
