# wiki-bootstrap — reference

Stub content lives as individual files under `${CLAUDE_PLUGIN_ROOT}/`:

| File written to user's wiki | Source in plugin |
| --- | --- |
| `.gitignore` | `stubs/gitignore` |
| `.llm-wiki.yaml` | `stubs/llm-wiki.yaml` |
| `CLAUDE.md` | `stubs/CLAUDE.md` |
| `OBSIDIAN.md` | `OBSIDIAN.md` (plugin root — same file used for GitHub browsing) |
| `README.md` | `stubs/README.md` |
| `index.md` | `stubs/index.md` |

## Why `OBSIDIAN.md` is at the plugin root, not under `stubs/`

It serves two audiences with identical content:

1. Someone browsing the plugin on GitHub before install — the repo-root path is where they'll look.
2. Users inside a wiki — `/wiki-init` copied it to their wiki's root.

Single source prevents the two copies from drifting out of sync. Every other stub is wiki-only so lives under `stubs/`.

## Editing stubs

Change the files directly under `${CLAUDE_PLUGIN_ROOT}/stubs/` or `${CLAUDE_PLUGIN_ROOT}/OBSIDIAN.md`. No restart of the bootstrap skill is needed — it reads the files at invocation time.

Schema changes to `.llm-wiki.yaml` require a `schema_version` bump and a migration note for existing wikis.
