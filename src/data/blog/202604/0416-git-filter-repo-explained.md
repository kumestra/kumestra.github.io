---
title: git-filter-repo — History Rewriting Done Right
description: What git-filter-repo is, how it works, why git doesn't include it, and where it fits among other popular third-party git tools.
pubDatetime: 2026-04-16T00:53:12Z
tags: [git, git-filter-repo, developer-tools, version-control]
---

## The Problem It Solves

Git has a built-in tool for rewriting history called `git filter-branch`. It works, but it has a bad reputation:

- **Slow** — rewrites history by checking out every commit one by one
- **Complex** — the API is confusing and easy to misuse
- **Dangerous** — mistakes can silently corrupt a repo

`git filter-repo` was written as a drop-in replacement that is faster, safer, and far easier to reason about. The git project now officially recommends it over `git filter-branch` — the `git filter-branch` man page was updated to warn users away from it and point to `git filter-repo` instead.

---

## What git-filter-repo Does

`git filter-repo` rewrites git history by walking every commit and applying a transformation. It does three things:

1. **Rewrites file paths** — renames or moves files across all commits in history
2. **Rewrites or drops commits** — keeps only commits relevant to the filter you applied, drops the rest entirely
3. **Removes all remotes** — intentionally, as a safety measure to prevent you from accidentally pushing rewritten history back to the original remote and corrupting it

The remote removal comes with a notice:

```
NOTICE: Removing 'origin' remote; see 'Why is my origin removed?'
        in the manual if you want to push back there.
        (was https://github.com/original/repo.git)
```

After running it, you add a new remote manually and push to the intended destination.

---

## The `--subdirectory-filter` Flag

The most common use case is extracting a subdirectory into its own standalone repo.

```bash
git filter-repo --subdirectory-filter src/sqlite
```

This command:

- **Keeps** only commits that touched something inside `src/sqlite/`
- **Rewrites** those commits so `src/sqlite/` becomes the repo root — a file at `src/sqlite/server.py` becomes `server.py`
- **Drops** all commits that only touched files outside `src/sqlite/`

```
Before                              After
──────────────────────────          ──────────────────
src/sqlite/                    →    (root)
  pyproject.toml               →    pyproject.toml
  src/mcp_server_sqlite/       →    src/mcp_server_sqlite/
  README.md                    →    README.md
src/github/                    ←    gone
src/postgres/                  ←    gone
... (other directories)        ←    gone
```

### What happens to mixed commits?

If a commit modified files both inside and outside the target subdirectory, the commit is **kept but trimmed** — only the changes inside `src/sqlite/` survive, the rest are silently dropped from that commit. The commit message stays unchanged, so it may reference files that no longer appear in the diff. This is a minor cosmetic oddity and does not affect functionality.

---

## Running It Without Installing

`git filter-repo` is not a built-in git command — it must be installed separately. If you have `uv` available, you can run it without touching your OS:

```bash
uvx git-filter-repo --subdirectory-filter src/sqlite
```

`uvx` runs the tool in a temporary isolated virtual environment. It does not install anything into your system Python and leaves no packages behind after the command finishes. The only thing it modifies is the git repo directory you run it in.

To install it permanently:

```bash
# Ubuntu/Debian
sudo apt install git-filter-repo

# or via pip
pip install git-filter-repo
```

---

## A Complete Subdirectory Extraction Workflow

Here is the full workflow for extracting a subdirectory into its own repo with preserved history:

```bash
# 1. Copy the original repo (never modify the original)
cp -r /path/to/original-repo /path/to/new-repo-name

# 2. Run the filter
cd /path/to/new-repo-name
uvx git-filter-repo --subdirectory-filter src/target-dir

# 3. Add the new remote (filter-repo removed the old one)
git remote add origin git@github.com:username/new-repo-name.git

# 4. Push
git push -u origin main
```

> ⚙️ Always work on a **copy** of the original repo. `git filter-repo` modifies history in place.

---

## Why git Doesn't Include It

`git filter-repo` is widely recommended but ships separately. A few reasons:

- **Python dependency** — `git filter-repo` is a Python script. Git's core is written in C and has historically avoided requiring Python to keep it portable across systems where Python may not be available.
- **Third-party origins** — it was written by an individual contributor (Elijah Newren), not the core git team. Git is conservative about pulling in external tools.
- **Inertia** — `git filter-branch` already exists as a built-in. Replacing it would break existing scripts that depend on it.
- **Scope philosophy** — git keeps its core minimal and lets the ecosystem handle higher-level workflows.

---

## Other Popular Git Tools Outside Core git

`git filter-repo` is not alone. Several widely-used git tools live outside the core for the same reasons:

| Tool | What it does |
|---|---|
| **git-lfs** | Stores large binary files outside the repo — common in game dev and data science |
| **git-absorb** | Automatically figures out which commit a staged change belongs to and amends it |
| **git-delta** | A better diff viewer with syntax highlighting |
| **git-branchless** | Suite of tools for stacked commits / stacked PRs workflow |
| **gh** | GitHub CLI — opening PRs, reviewing issues, managing releases |
| **lazygit** | Full-featured terminal UI for git |
| **tig** | Lightweight terminal UI for browsing history |

The pattern is consistent: useful, widely adopted, but external because they are third-party, have extra runtime dependencies, or fall outside git's deliberately narrow core scope.

---

## References

- [git-filter-repo on GitHub][filter-repo-gh] — source code, documentation, and the "Why is my origin removed?" FAQ

[filter-repo-gh]: https://github.com/newren/git-filter-repo
