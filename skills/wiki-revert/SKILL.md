---
name: wiki-revert
description: Use when the user runs /revert [sha], says "undo the last ingest", "roll back commit X", or identifies a bad wiki-mutation commit that needs to be backed out. Runs git revert on the target SHA (or the most recent ingest if no SHA given) and commits with a revert(<original-scope>) subject. Refuses on merge conflicts; otherwise fully automatic.
---

# wiki-revert

Backs out a wiki-mutation commit via `git revert`. Produces a new commit; history stays linear.

## Refuse conditions

1. No `.llm-wiki.yaml` in CWD or ancestors.
2. `.llm-wiki.yaml:schema_version != 1`.
3. Working tree dirty (`git status --porcelain` non-empty).
4. Target SHA is the bootstrap commit.
5. Target SHA does not exist or is outside the current branch's history.
6. `git revert --no-commit <sha>` produces merge conflicts. Abort the revert (`git revert --abort`) and refuse; conflict resolution is a human task that requires `/review`.

## Workflow

### 1. Resolve target SHA

- If the caller passed a SHA, use it.
- Else, pick the most recent wiki-mutation commit matching the scope grep:
  ```
  git log --oneline --extended-regexp --grep "^(ingest|query|revise|lint)" -n 1
  ```
  — that SHA is the default.

### 2. Gather context

- `git show --stat <sha>` → file list, +/- stats, original subject.
- Extract original scope from subject prefix.
- Descendant check: for each file the commit touched, run `git log --oneline <sha>..HEAD -- <file>`. Record files with later commits.

### 3. Run the revert

```
git revert --no-commit <sha>
```

On conflict: `git revert --abort` and refuse (see refuse conditions).

### 4. Commit

```
git commit -m "revert(<original-scope>): undo <original-subject-truncated-to-60-chars>"
```

If the descendant check found later commits on the same files, append the list to the commit body:

```
Descendant commits on same files (may need follow-up revision):
- <short-sha> <scope>: <subject>
- ...
```

If the target commit was itself a revert (scope `revert(...)`), the new commit's scope becomes `revert(revert-<target>)` so `/history` narrates the chain.

## List mode

When the caller runs `/revert list` (or `/revert --list`), skip to a listing-only run:

```
git log --oneline --extended-regexp --grep "^(ingest|query|revise|lint)" -n <revert.recent_commits_window>
```

Output the list with columns `SHA · scope · relative-date · file-count · subject`. No revert is executed.

## Output

```
Reverted <short-sha> (<original-scope>) as commit <new-short-sha>. Files affected: N. Descendants noted: M.
```
