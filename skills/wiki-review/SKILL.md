---
name: wiki-review
description: Use when the user runs /review, says "show pending changes", "what's uncommitted", or wants an audit of uncommitted wiki edits before they land in history. Narrates uncommitted changes, flags structural issues scoped to those changes, and commits them with a conventional subject. Fully automatic.
---

# wiki-review

Audits and commits the uncommitted diff on wiki paths. Intended for hand-edits or mid-ingest interruptions; a normal `/ingest` already commits on its own.

## Refuse conditions

1. No `.llm-wiki.yaml` in CWD or ancestors.
2. `.llm-wiki.yaml:schema_version != 1`.
3. Working tree clean on `wiki/` and `index.md` — nothing to review.

## Workflow

### 1. Narrate the diff

- `git status -- wiki/ index.md raw/`
- `git diff --stat -- wiki/ index.md`
- For each changed file, `git diff <file>` and report:
  - Changed sections (heading / frontmatter / list / prose).
  - Frontmatter invariants: `updated:` bumped to today; `sources:` appended not replaced; `contradictions:` entries well-shaped.
  - Literal change narration for hand-edits; no inferred motivation.

### 2. Structural flags scoped to changed files

Using the resolution rules in `skills/wiki-ingest/reference.md`:

- New dead `[[link]]` (zero hits).
- New ambiguous `[[link]]` (multiple hits).
- New orphan page (created without any inbound link and not covered by `lint.orphan_allowlist`).
- New `contradictions:` entries.
- Index drift: new page missing from `index.md`, or an index entry pointing to a now-missing page.

If any flag fires, list them. They do not block the commit.

### 3. Pick commit scope

- Single file under `wiki/` → `revise(<page-slug>): <first-heading-of-diff-or-filename>`.
- Multi-file under `wiki/` → `revise(multi): <count> page(s)`.
- Includes `.llm-wiki.yaml` or other config changes → split: commit wiki changes first with a wiki scope, then commit config with `chore: <summary>`.
- Includes `raw/` additions without a corresponding `wiki/sources/<slug>.md` → refuse: raw without a source page means an ingest was interrupted; run `/ingest` to complete it.

### 4. Commit

```
git add <explicit-paths>
git commit -m "<scope>: <subject>"
```

Never `git add .`.

## Output

Single block:

```
Reviewed <N> file(s). Flags: <count or "none">. Committed as <short-sha> <scope>: <subject>.
```
