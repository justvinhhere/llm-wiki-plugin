---
name: wiki-lint
description: Use when the user runs /lint [--since <ref>], says "health-check the wiki", "find orphans", "check for dead links", or wants a structural audit. Scans the filesystem for six categories of issue — orphans, dead/ambiguous links, stale pages, missing cross-references, unresolved contradictions, index drift. Read-only; reports only; does not auto-fix.
---

# wiki-lint

Full-wiki structural audit.

## Refuse conditions

1. No `.llm-wiki.yaml` in CWD or ancestors.
2. `.llm-wiki.yaml:schema_version != 1`.

## Workflow

### 1. Build the page index

Walk `wiki/**/*.md`. For each page, capture: path, slug, frontmatter (`type`, `tags`, `status`, `updated`, `sources`, `contradictions`, `allow_orphan`), outbound `[[X]]` list.

Load `.llm-wiki.yaml:lint` — `stale_days`, `orphan_allowlist`, `default_scope`, `suggest_after_n_ingests`.

### 2. Scope

| `--since` / config | Pages scanned |
| --- | --- |
| `full` (default) | All pages. |
| `since-last-commit` | `git diff --name-only HEAD~1 HEAD -- 'wiki/**/*.md'`. |
| `<git-ref>` | `git diff --name-only <ref> HEAD -- 'wiki/**/*.md'`. |

Orphan and dead-link checks resolve against the full page index regardless of scope; scope limits which pages get flagged.

### 3. Checks

See `reference.md` for detection algorithms and false-positive guards.

| Check | Output item |
| --- | --- |
| Orphans | Page path, type, updated. |
| Dead links | Page:line, target, `dead-link` or `dead-link-ambiguous`. |
| Stale | Page, `updated`, days past `stale_days`. |
| Missing cross-refs | Page:line, unwrapped mention, suggested wrap. |
| Contradictions | Page, each `{source, note}` entry. |
| Index drift | Missing entries (page not indexed) and stale entries (index points to missing page). |

### 4. Output

Structured report grouped by check, with counts per check and a total. If zero findings across all checks, output `No findings.`.

Append a nudge when `git log --grep "^ingest" --oneline` since the last `lint(...)` commit exceeds `lint.suggest_after_n_ingests`:

```
Note: <N> ingests since last lint pass.
```

### 5. No auto-fix

Lint writes nothing. Users fix findings via `/revise` or `/ingest`.
