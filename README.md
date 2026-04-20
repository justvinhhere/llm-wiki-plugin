# llm-wiki

A Claude Code plugin for building a compound, LLM-maintained knowledge base as git-versioned markdown. Every wiki mutation is a commit, so `git blame` on any line traces the claim back to the ingest that introduced it and the raw source it cites — **provenance for free, no inline citations to maintain.**

Inspired by Andrej Karpathy's gist [*"On LLM Wikis"*](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — this plugin is one concrete take on the compound-knowledge-base-as-git-repo idea sketched there.

Works excellent with [Obsidian](./OBSIDIAN.md) as a viewer, but Obsidian is not required.

## Install

```
/plugin marketplace add https://github.com/justvinhhere/llm-wiki-plugin
/plugin install llm-wiki@llm-wiki-marketplace
```

Restart Claude Code.

## Quickstart

```
mkdir my-wiki && cd my-wiki
/llm-wiki:wiki-init          # scaffold skeleton + first commit
/llm-wiki:ingest https://... # read a source, write pages, commit as ingest(<slug>)
/llm-wiki:query "what does the wiki say about X"
```

All commands also work as short forms (`/wiki-init`, `/ingest`, `/query`, …) when there's no name collision with other installed plugins.

## Commands

| Command | Purpose |
| --- | --- |
| `/llm-wiki:wiki-init` | Scaffold a new wiki in the current directory + first commit. |
| `/llm-wiki:ingest <path-or-url>` | Read a source, write all affected pages, commit as `ingest(<slug>)`. |
| `/llm-wiki:query <question>` | Search + synthesize from wiki content; synthesis answers auto-file as query pages. |
| `/llm-wiki:review` | Audit + commit any uncommitted wiki edits with a conventional subject. |
| `/llm-wiki:blame <page> [line]` | Trace a claim to its commit, ingest scope, and the raw source passage. |
| `/llm-wiki:history <page> [--diffs]` | Narrated timeline of a single page's commits. |
| `/llm-wiki:lint [--since <ref>]` | Structural audit — orphans, dead links, stale pages, contradictions. Read-only. |
| `/llm-wiki:revert [<sha>]` | Undo a wiki-mutation commit via `git revert`. Defaults to the most recent ingest. |

## How git is used

The plugin treats git as the substrate for three specific guarantees:

- **Provenance without inline citations.** Every commit subject is scoped to its source (`ingest(<slug>)`, `query(<slug>)`, `revise(<page>)`). `git blame` on any wiki line returns the commit that introduced it; the scope points to the source page; the source page's `raw_path:` points to the original material. No `(source: X)` bookkeeping in prose required.
- **Full diff-level history.** Every commit records the complete before/after of every touched page. `/llm-wiki:history <page>` narrates a page's evolution; `/llm-wiki:blame <page>` surfaces the introducing commit; `git log --grep "^ingest"` lists every ingest across the wiki's life.
- **Recoverable mistakes.** Bad ingest? `/llm-wiki:revert` runs `git revert` and commits the undo. History stays linear; the reverted change is itself greppable.

The plugin is deliberately thin above git — no shipped scripts, no hooks, no MCP. Skills are prose guidance; the agent composes concrete `git` and filesystem calls at runtime.

## Automation model

Every skill is **automation-first**: no interactive prompts. `/llm-wiki:ingest` reads, writes, and commits in one invocation. Safety lives in refuse-conditions (schema mismatch, dirty wiki tree, collision on init, merge conflict on revert) — skills abort on unsafe preconditions instead of asking.

Humans audit through git after the fact (`git log`, `git show`, `/llm-wiki:review` for uncommitted hand-edits, `/llm-wiki:blame` for any line, `/llm-wiki:lint` for structural health). The wiki is designed so an agent can maintain it unattended while a human periodically checks.

## Configuration

Each wiki carries a `.llm-wiki.yaml` at its root. Defaults cover every tunable parameter. Commonly edited keys:

- `lint.stale_days` — age threshold for flagging stale pages (default 180).
- `query.top_k_candidates` — max pages read in full when answering a query (default 8).
- `lint.orphan_allowlist` — glob patterns whose pages are never flagged as orphans.
- `ingest.raw_layout` — where each source type lands under `raw/` (article / paper / transcript / note).

## With Obsidian

Obsidian is an excellent viewer for a llm-wiki — `[[wikilinks]]` are native, the graph view visualizes the page network, Dataview lets you query frontmatter dynamically. Open the wiki repo as an Obsidian vault.

See [OBSIDIAN.md](./OBSIDIAN.md) for setup, recommended plugins, required settings, and the Obsidian-plus-Claude-Code workflow.

Obsidian is **not** a dependency — you can edit the wiki in any editor, and every `/llm-wiki:` command runs without it.

## Example wiki

```
my-wiki/
├── .git/
├── .gitignore
├── .llm-wiki.yaml
├── CLAUDE.md
├── OBSIDIAN.md
├── README.md
├── index.md                  # catalog, plugin-maintained
├── raw/
│   ├── articles/
│   └── papers/
└── wiki/
    ├── entities/
    ├── concepts/
    ├── sources/
    ├── topics/
    └── queries/
```

## What this is not

- Not a Wikipedia replacement — no permissions, no multi-user conflict resolution.
- Not enterprise-scale — targets personal / small-team corpora (10²–10³ sources).
- Not a RAG killer — at scale you still want vector search; the plugin can later shell out to `qmd` or similar.
- Not a deterministic knowledge system — the LLM still makes mistakes; git ensures they're visible and reversible.

## License

MIT — see [LICENSE](LICENSE).
