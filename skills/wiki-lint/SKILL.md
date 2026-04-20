---
name: wiki-lint
description: Use when the user runs /lint [--since <ref>], says "fix wiki issues", "resolve contradictions", "heal the wiki", "clean up dead links", "health-check the wiki", or wants a full structural pass. Scans the wiki for six categories (orphans, dead/ambiguous links, stale pages, missing cross-references, unresolved contradictions, index drift) and fixes each in place — the agent picks the tactic per finding — committing the pass as lint(<yyyy-mm-dd>). Fully automatic. Detection algorithms and healing principles live in reference.md.
---

# wiki-lint

Full-wiki structural audit-and-repair. One invocation produces exactly one `lint(<date>)` commit (or none, if the wiki is already healthy).

## Refuse conditions

1. No `.llm-wiki.yaml` in CWD or ancestors.
2. `.llm-wiki.yaml:schema_version != 1`.
3. `git status --porcelain -- wiki/ index.md` non-empty — working tree dirty on wiki paths. The caller must commit or stash before linting (use `/review` for hand-edits).

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

Orphan, dead-link, and cross-ref checks resolve against the full page index regardless of scope; the scope filter limits which pages can produce findings (and therefore which pages get mutated).

### 3. Detect

Run all six checks from `reference.md`. Build an internal list of findings grouped by check:

| Check | Finding shape |
| --- | --- |
| Orphans | Page path, type. |
| Dead links | Page:line, target, `dead-link` or `dead-link-ambiguous`. |
| Stale | Page, `updated`, days past `stale_days`. |
| Missing cross-refs | Page:line, bare mention, candidate slug. |
| Contradictions | Page, each `{source, note}` entry. |
| Index drift | Missing entries and stale entries. |

If zero findings across all checks, skip to step 6 with the healthy-wiki output; do not create a commit.

### 4. Heal

For each finding, pick a tactic guided by the **healing principles** in `reference.md`. Mutate in place.

The agent is explicitly authorized to use its judgment — read source pages, consult `raw/` passages for contradictions, pick the better-supported side, rewrite prose, add or remove `[[…]]`, set frontmatter `status:`, regenerate `index.md` entries. No rule table prescribes the tactic; the principles describe the target state, the agent picks the mechanism.

Keep a running log, internal to this run: `{page, finding-category, one-sentence-call}`. The log feeds the commit body in step 5.

### 5. Commit

Single batch.

```
git add <explicit-paths-touched>
git commit -F <message-file>
```

Subject: `lint(<yyyy-mm-dd>): fix N orphan(s), M dead link(s), K contradiction(s), …` — list only categories with non-zero fixes. Singular / plural per count. Subject ≤72 chars; drop least-important categories first if it overflows.

Body: one short line per resolved contradiction naming the page and the call the agent made, plus any non-obvious judgment from other categories (e.g. "merged [[foo]] into [[bar]] — same subject"). Straightforward fixes (a typo'd link, an index entry) need no body line. No `Co-Authored-By` trailer.

### 6. Output

Single line.

- With fixes: `Linted <yyyy-mm-dd>: N fix(es) across M page(s). Commit <short-sha>.`
- No findings: `Wiki is healthy — no findings.`
