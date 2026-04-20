---
name: wiki-query
description: Use when the user runs /query <question>, says "what do we know about X", "summarize our view on Y", "compare A vs B in the wiki", "search the wiki for Z", "which pages link to [[W]]", "when did we first mention Q", or asks any question answerable from wiki content. Classifies the question as lookup or synthesis; runs four internal searches (content / frontmatter / graph / git-history); answers with citations. Synthesis answers auto-file as query pages with a query(<slug>) commit; lookups answer in chat only.
---

# wiki-query

Answers questions against the current wiki and its git history. Never fetches the web.

## Refuse conditions

1. No `.llm-wiki.yaml` in CWD or ancestors.
2. `.llm-wiki.yaml:schema_version != 1`.
3. Synthesis filing would require a dirty working tree to merge — caller must `/review` first.

## Workflow

### 1. Classify

| Kind | Signal | Filing |
| --- | --- | --- |
| Lookup | Asks for pointers / list / counts. "which pages link to X", "list pages tagged Y", "when did Z first appear". | Skip. |
| Synthesis | Asks for comparison, summary, explanation, cross-page connection. Answer is new prose. | File. |

Ambiguous → synthesis (filing is cheap, reversible via `/revert`).

If the raw question text contains `"don't file"` or `"just answer"` (case-insensitive, substring match), force skip regardless of classification.

### 2. Candidate discovery (parallel)

- **Content.** `git grep -n -i "<keywords>" -- 'wiki/**/*.md'`. Keywords: question tokens minus stopwords.
- **Frontmatter.** Parse YAML of all wiki pages; filter by `type`, `tags`, `status` where the question names them.
- **Graph.** For every proper-noun in the question, `git grep -l "\[\[<slug>\]\]" -- 'wiki/**/*.md'`.
- **Historical** (only if the question contains `when`, `first`, `last`, `recently`, `over time`). `git log -S "<phrase>" --oneline -- 'wiki/**/*.md'`.

### 3. Rank and read

Rank union: exact-title-match > frontmatter-tag-match > backlink-count > content-frequency. Read top `query.top_k_candidates` pages in full. If fewer candidates, read all.

### 4. Synthesize

Inline citations:
- `(see [[page-slug]])` for general support.
- `(see [[page-slug]] ¶ "phrase")` for passage-level.
- `(introduced in <sha>, ingest(<slug>))` for temporal claims.

If the wiki has no relevant content, output `Wiki has no material on this. Try /ingest <source>.` and stop — no file written.

### 5. Dedupe against existing query pages

If `query.dedupe_against_queries` is true: `git grep -l -i "<question-keywords>" -- 'wiki/queries/**/*.md'`. If an existing query page answers the same question (same `question:` frontmatter or >80% keyword overlap with its `answer_summary:`), output `Already answered at [[queries/<slug>]]` and stop.

### 6. File, if synthesis

- Slug: question text → stopword strip → general slug rule (`skills/wiki-ingest/reference.md`).
- Copy `templates/query.md`, fill frontmatter: `title`, `question`, `asked: <today>`, `answer_summary`, `cites_pages`, `sources` (union of cited pages' `sources`).
- Append entry to `index.md` under `## Queries`.

### 7. Commit

```
git add wiki/queries/<slug>.md index.md
git commit -m "query(<slug>): file synthesized answer"
```

## Output

- **Lookup:** the answer list.
- **Synthesis:** the answer body + `Filed as [[queries/<slug>]] in commit <short-sha>.`
- **Skip (dedupe or explicit):** pointer line only.
