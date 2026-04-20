# llm-wiki

A Claude Code plugin for building a compound, LLM-maintained knowledge base as git-versioned markdown. Every wiki mutation is a commit, so `git blame` on any line traces the claim back to the ingest that introduced it and the raw source it cites — **provenance for free, no inline citations to maintain.**

## Install

```
/plugin marketplace add https://github.com/justvinhhere/llm-wiki-plugin
/plugin install llm-wiki
```

Restart Claude Code.

## Quickstart

```
mkdir my-wiki && cd my-wiki
/wiki-init                 # scaffold skeleton + first commit
/ingest https://...        # read a source, file it, commit on approval
/query "what does the wiki say about X"
```

## Commands

| Command | Purpose |
| --- | --- |
| `/wiki-init` | Scaffold a new wiki in the current directory. |
| `/ingest <path-or-url>` | Read a source, propose page touches, commit on approval. |
| `/query <question>` | Search + synthesize from wiki content; auto-files synthesis answers. |
| `/review` | Narrate uncommitted changes, suggest a commit message. |
| `/blame <page> [line]` | Trace a claim to its commit, ingest scope, and raw source passage. |
| `/history <page>` | Narrated timeline of a single page's evolution. |
| `/lint` | Structural health check (orphans, dead links, stale pages, contradictions). |
| `/revert` | Undo a bad ingest via `git revert`. |

## Why git

Karpathy's [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) pattern treats provenance as a hard problem. This plugin's bet is that `git` already solved three of the hardest parts:

- **Provenance** → `git blame` + scoped commit subjects (`ingest(karpathy-llm-wiki)`) give line-level citations with no bookkeeping.
- **Reviewability** → the write path is always _draft → diff → approve → commit_; no silent wiki mutations.
- **Recoverability** → `git revert` is a one-command undo for any ingest.

The plugin layers thin LLM-driven workflows on top of those primitives and stays out of git's way otherwise. No hooks, no MCP, no shipped scripts — the agent composes concrete `git` and filesystem calls at runtime based on prose guidance in the skills.

## Configuration

Each wiki carries a `.llm-wiki.yaml` at its root. Defaults cover all tunable parameters. Keys users commonly edit:

- `lint.stale_days` — age threshold for flagging stale pages (default 180).
- `query.top_k_candidates` — max pages read in full when answering a query (default 8).
- `lint.orphan_allowlist` — glob patterns whose pages are never flagged as orphans.
- `ingest.raw_layout` — where each source type lands under `raw/`.

## Example wiki

```
my-wiki/
├── .git/
├── .gitignore
├── .llm-wiki.yaml
├── CLAUDE.md
├── README.md
├── index.md                  # catalog, LLM-maintained
├── raw/
│   ├── articles/
│   │   └── karpathy-llm-wiki.md
│   └── papers/
└── wiki/
    ├── entities/karpathy.md
    ├── concepts/llm-wiki.md
    ├── concepts/memex.md
    ├── sources/karpathy-llm-wiki.md
    ├── topics/compound-knowledge.md
    └── queries/rag-vs-wiki.md
```

## What this is not

- Not a Wikipedia replacement — no permissions, no multi-user conflict resolution.
- Not enterprise-scale — targets personal / small-team corpora (10²–10³ sources).
- Not a RAG killer — at scale you still want vector search; the plugin can later shell out to `qmd` or similar.
- Not a deterministic knowledge system — the LLM still makes mistakes; git ensures they're visible and reversible.

## License

MIT — see [LICENSE](LICENSE).
