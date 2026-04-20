---
name: wiki-bootstrap
description: Use when the user runs /wiki-init, says "set up a new wiki", "initialize a wiki here", "scaffold a wiki", or invokes any llm-wiki command from a directory without .llm-wiki.yaml. Creates the wiki skeleton and makes the first commit. Refuses on unsafe preconditions; otherwise fully automatic.
---

# wiki-bootstrap

Scaffolds a new llm-wiki at CWD. Every other skill assumes the structure this skill creates.

## Refuse conditions

Abort and report reason. Do not mutate anything.

1. `.llm-wiki.yaml` exists at CWD or any ancestor — already inside a wiki.
2. `git --version` fails — git unavailable.
3. CWD is `$HOME`, `/`, `/tmp`, `/Users`, `/var`, or any directory not specifically created for a wiki.
4. CWD is non-empty and contains any path in the write-set (`.gitignore`, `.llm-wiki.yaml`, `CLAUDE.md`, `OBSIDIAN.md`, `README.md`, `index.md`, `wiki/`, `raw/`). Collision with existing content is a hard refuse.

A non-empty CWD that does not collide with the write-set proceeds silently. A pre-existing `.git` is reused; otherwise `git init` runs.

## Write set

Create, in order:

1. `git init` if no `.git`.
2. Directories: `wiki/entities/`, `wiki/concepts/`, `wiki/sources/`, `wiki/topics/`, `wiki/queries/`, `raw/`.
3. Files, from the verbatim stubs in `reference.md`:
   - `.gitignore`
   - `.llm-wiki.yaml`
   - `CLAUDE.md`
   - `OBSIDIAN.md`
   - `README.md`
   - `index.md`
4. Stage only the files listed above (explicit paths; never `git add .`).
5. Commit with subject `bootstrap: initialize llm-wiki skeleton`.

## Output

Single line on success: `Wiki initialized at <pwd>. Next: /llm-wiki:ingest <path-or-url>.`
