---
name: wiki-blame
description: Use when the user runs /blame <page> [line-or-phrase], says "why does the wiki say X", "which source introduced this claim", or asks for the provenance of a specific wiki assertion. Traces a line back to its commit SHA, parses the commit scope, and surfaces the source page's frontmatter + the corresponding raw passage. Read-only.
---

# wiki-blame

Traces `git blame` → commit scope → source page → raw passage, in one step.

## Refuse conditions

1. No `.llm-wiki.yaml` in CWD or ancestors.
2. Target page does not exist under `wiki/`.
3. Target page is uncommitted (no git history).

## Workflow

### 1. Resolve the blame target

| Invocation | Command |
| --- | --- |
| `/blame <page>` | `git blame <page>` (full file; group contiguous same-SHA runs). |
| `/blame <page> <N>` | `git blame -L <N>,<N> <page>`. |
| `/blame <page> <N>-<M>` | `git blame -L <N>,<M> <page>`. |
| `/blame <page> "<phrase>"` | `git grep -n "<phrase>" <page>` → take line → `git blame -L <line>,<line> <page>`. Blame each matching line. |

### 2. Per distinct SHA

`git show --stat <sha>`. Parse scope prefix and act:

| Scope | Action |
| --- | --- |
| `ingest(<slug>)` | Read `wiki/sources/<slug>.md` frontmatter → `raw_path`, `url`, `author`, `published`. Locate the corresponding passage in `raw_path` by grepping 3–5 distinctive phrases from the wiki line. If `raw_path` is gitignored or binary, report source-page metadata only. If multiple raw passages match, include the top 2. |
| `query(<slug>)` | Read `wiki/queries/<slug>.md` frontmatter → `question`, `answer_summary`. |
| `revise(<page>)` | Output the commit body (if any); no source link. |
| `lint(<date>)` | Report as structural fix; no source link. |
| `revert(<target>)` | Run `git log --follow` on the page to find the original `ingest(...)` that first introduced the reverted-and-reintroduced content; include that SHA in the output. |
| `bootstrap:` | Report as scaffolding; no claim. |

### 3. Output format

Single-line blame:

```
Line <N>: added in <short-sha> (full: <full-sha>) on <YYYY-MM-DD> via <scope>.
Cites: raw/<path> ¶<N>:
  > "<exact passage>"
Source: <author>, <published>, <url>
```

Full-file or range blame:

```
<page>: <line-count> lines across <sha-count> commits
  - <N1> lines from <short-sha-1> <scope-1> — <date>
  - <N2> lines from <short-sha-2> <scope-2> — <date>
  ...
```

Followed by per-SHA detail blocks.

## Notes

- Short SHAs are 7 chars; full SHAs shown once per distinct commit for follow-up `git show`.
- Dates use author date (`%ai`), ISO 8601.
- Raw passages are quoted verbatim, never paraphrased.
