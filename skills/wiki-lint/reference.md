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

## Commit format for manual follow-up

When the user fixes findings, suggest these subjects:

- Single-page fix → `revise(<page-slug>): <short why>`.
- Bulk structural fix following a lint pass → `lint(<yyyy-mm-dd>): fix N orphan(s), M dead link(s)`.

Using `lint(<date>)` for the bulk fix makes the audit-and-fix pair greppable: `git log --grep "^lint"`.
