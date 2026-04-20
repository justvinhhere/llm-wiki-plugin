---
name: wiki-history
description: Use when the user runs /history <page> [--diffs], says "how did this page evolve", "show me the history of [[X]]", or wants a narrated timeline of a single page's commits. Classifies each commit (creation / expansion / maintenance / rename) and narrates. Read-only.
---

# wiki-history

Narrates the commit history of a single wiki page.

## Refuse conditions

1. No `.llm-wiki.yaml` in CWD or ancestors.
2. `git log --follow -- <page>` returns empty and `<page>` is not in the working tree — the page never existed.

## Workflow

### 1. Pull the log

```
git log --follow --format="%H %ai %s" --reverse -- <page>
git log --follow --name-status --diff-filter=R -- <page>
```

First query yields SHA + date + subject per commit. Second detects rename boundaries (entries marked `R<score>`).

### 2. Classify each commit

| Signal | Class |
| --- | --- |
| `git show --stat <sha>` lists the page as `create mode` | Creation |
| Scope `ingest(<slug>)`, not creation | Expansion via ingest |
| Scope `revise(<slug>)`, not creation | Human revision |
| Scope `lint(<date>)` | Maintenance |
| Rename flag from second query | Rename (record old and new path) |
| Scope `bootstrap:` | Scaffolding |

### 3. Narrate

Single paragraph, forward-chronological. Include short SHAs and dates. Quote revise commit bodies when non-trivial.

Example shape:

```
wiki/concepts/memex.md — <N> commits, <first-date> to <last-date>.

Created during ingest(karpathy-llm-wiki) on 2026-04-12 (a1b2c3d). Expanded during ingest(bush-as-we-may-think) on 2026-04-15 (e4f5g6h) with the Bush-1945 lineage. Contradiction with [[llm-wiki]] flagged and resolved during lint(2026-04-20) (i7j8k9l). Revised on 2026-04-20 (m0n1o2p, revise(memex): "clarified selection-by-association").
```

### 4. Current-state footer

Always append:

```
Status: <frontmatter.status>
Updated: <frontmatter.updated>
Sources: [<frontmatter.sources>]
Inbound links: <count from `git grep -l "\[\[<slug>\]\]" -- 'wiki/**/*.md' | wc -l`>
Outbound links: <count of [[X]] in page body>
```

### 5. Optional `--diffs`

If `--diffs` passed, after each narrated commit, inline its diff:

```
git show <sha> -- <page>
```

## Notes

- Short SHAs (7 chars) in narration.
- Dates ISO 8601.
- Rename cases use the new path for subsequent commits but name the old path at the rename boundary.
