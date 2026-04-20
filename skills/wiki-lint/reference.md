# wiki-lint — reference

Detection algorithms for each check. False-positive guards and scoping rules.

---

## Orphans

A page `P` is an orphan iff all hold:

- No page `Q` in `wiki/**/*.md` contains a wikilink that resolves to `P`.
- `P`'s path does not match any glob in `lint.orphan_allowlist`.
- `P`'s frontmatter does not set `allow_orphan: true`.

Default allowlist includes `index.md` and `wiki/topics/**`.

Implementation: build the backlink graph by scanning every `[[X]]` in every page and resolving X; add edge `Q → P`. A page with zero inbound edges fails the check unless allowlisted.

---

## Dead links

For every wikilink `[[X]]` in any page body, apply the resolution algorithm from `skills/wiki-ingest/reference.md`:

- Zero hits → `dead-link`.
- Multiple hits → `dead-link-ambiguous`.

### False-positive guards — skip matches inside:

- Fenced code blocks (```` ``` ````).
- Inline code (`` ` ``).
- YAML frontmatter blocks.
- Block quotes (lines starting with `>`).

---

## Stale

A page is stale iff:

- `frontmatter.status == 'stale'`, or
- `today() - frontmatter.updated > lint.stale_days` (default 180).

`status: deprecated` pages are never flagged as stale.

---

## Missing cross-references

For every entity / concept page, compute the set of mentionable forms:

- The slug.
- The frontmatter `title`.
- All entries in `aliases` (entities only).

Scan every other page body for bare occurrences of any mentionable form not wrapped in `[[…]]`.

### False-positive guards — skip matches inside:

- Fenced code blocks.
- Inline code.
- Frontmatter blocks.
- Block quotes.
- Existing wikilinks (`[[karpathy]]` must not self-flag).
- Markdown headings (bare text in `# Memex` is stylistic, not an error).

### Common-word collision list

Skip flagging when the slug matches any of: `api, net, query, wiki, memory, source, page, note, link, file, tag, type`. Users override via `.llm-wiki.yaml:lint.cross_ref_ignore_slugs` (future key).

---

## Unresolved contradictions

Any page with non-empty `contradictions:` frontmatter.

Entry shape (lint rejects other shapes):

```yaml
contradictions:
  - source: bush-as-we-may-think
    note: "claims associative linking was Bush's invention; Otlet-mundaneum page credits Otlet 1934"
```

Report each entry separately — one page with three contradictions produces three findings.

---

## Index drift

Two sub-checks on `index.md`:

**Missing:** every page in `wiki/**/*.md` (other than `index.md` itself) must have at least one `[[slug]]` reference under its category heading in `index.md`.

**Stale:** every `[[X]]` in `index.md` must resolve. Dead references are flagged.

Alphabetical ordering within categories is a soft warning, not a finding.

---

## Scoping with `--since`

`--since <ref>` or `lint.default_scope: <ref>`:

```
git diff --name-only <ref> HEAD -- 'wiki/**/*.md'
```

Orphan, dead-link, and cross-ref checks resolve against the full page index regardless of scope; the scope filter limits which pages can produce findings.

Index drift is always full-scope — the index is a single file, scoping is meaningless.

---

## Commit format

Every lint pass that applies at least one fix writes one commit:

```
lint(<yyyy-mm-dd>): fix N orphan(s), M dead link(s), K contradiction(s), …
```

List only categories with non-zero fixes. Singular / plural per count. Subject ≤72 chars — drop lower-priority categories from the subject before truncating mid-word; the body carries the full accounting.

Body (optional): one short line per resolved contradiction naming the page and the call, plus any non-obvious judgment calls from other categories (merges, page promotions, deletions). Mechanical fixes (typo'd slug, new index entry) need no body line. No `Co-Authored-By` trailer.

`git log --grep "^lint"` lists every lint pass.

---

## Healing principles

For each finding, the agent chooses the tactic that best preserves the wiki's invariants. These principles describe the target state, not the mechanism.

**Orphans.** Every non-allowlisted page should be reachable from the link graph. Reachability can come from a topic page, a related entity, `index.md`, or from promoting the page itself into a topic. Frame the choice around where a reader would actually discover this page.

**Dead links.** A `[[X]]` that doesn't resolve is either a typo (fix the slug), an alias that was renamed (point to the new slug), or a reference to a page that was never created (create it if the wiki substantively covers the subject; otherwise unwrap `[[X]]` to plain `X`). `dead-link-ambiguous` cases pick the contextually-correct target based on the surrounding paragraph.

**Stale pages.** Pages past `lint.stale_days` either still reflect the source material (bump `updated:` with no body change and note why in the commit body) or are genuinely out of date. For out-of-date, the agent may re-read cited sources and revise, or mark `status: stale` if a refresh needs material the wiki doesn't have. `status: deprecated` when the subject has been superseded — never flagged stale again.

**Missing cross-references.** A bare mention of an indexed entity / concept is a link opportunity, not a crime. Wrap it in `[[slug]]` when doing so aids navigation; leave it bare when the mention is incidental (figure caption, list of examples, paragraph already saturated with links). The agent's test is "would a reader benefit from clicking here."

**Contradictions.** `contradictions:` frontmatter entries represent claims the ingest skill parked for editorial adjudication. Resolve by reading both source pages (their `raw_path:` content) and reconciling: prefer the more recent + higher-quality source, or synthesize a one-paragraph note acknowledging the tension if both sides hold. Rewrite the affected claim in the page body. Clear the resolved entry from `contradictions:`. Record the call in the lint commit body.

**Index drift.** `index.md` is a catalog — every page belongs under its category, every `[[X]]` resolves. Regenerate entries as needed. Alphabetical within category is a soft guideline, not an invariant.
