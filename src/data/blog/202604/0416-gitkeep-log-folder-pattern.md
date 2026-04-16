---
title: The .gitkeep Pattern — Tracking Empty Directories in Git
description: How modern repos use a logs/ folder with .gitkeep and .gitignore rules to track the directory structure without committing log files.
pubDatetime: 2026-04-16T09:51:50Z
tags: [git, gitignore, gitkeep, logging, project-structure]
---

## The Problem 📁

Git does not track empty directories. If your project needs a `logs/` folder to exist after cloning, you cannot just create the folder and commit it — git will ignore it completely.

At the same time, you never want actual log files committed. They are runtime artifacts, often large, always machine-specific.

---

## The Pattern

Modern repos solve this with two things together:

**1. An empty `.gitkeep` file inside the folder**

```
logs/
└── .gitkeep
```

`.gitkeep` is just an empty placeholder file. Git has no special knowledge of it — it is purely a community convention for "keep this empty directory". Because the file exists, git tracks the directory.

**2. `.gitignore` rules that ignore everything except `.gitkeep`**

```gitignore
logs/*
!logs/.gitkeep
```

- `logs/*` — ignore every file inside `logs/`
- `!logs/.gitkeep` — except `.gitkeep` itself (the `!` negates the rule)

---

## Result

```
git clone → logs/ directory exists ✅
log files generated at runtime → never committed ✅
.gitkeep → committed, keeps the folder tracked ✅
```

Everyone who clones the repo gets the `logs/` folder ready to use. No manual `mkdir logs` needed. No log files ever appear in `git status`.

---

## Why Not Just Add logs/ to .gitignore?

If you ignore the whole `logs/` directory:

```gitignore
logs/
```

The directory itself is ignored — git will not track it at all, and it will not exist after a fresh clone. Code that tries to write to `logs/something.log` will fail with a "directory does not exist" error.

The `logs/*` + `!logs/.gitkeep` combination ignores the *contents* while keeping the *directory*.

---

## The gitignore Rule Order Matters

`.gitignore` processes rules top to bottom. A negation rule (`!`) only works if it comes after the rule it is negating:

```gitignore
# CORRECT — ignore everything first, then un-ignore .gitkeep
logs/*
!logs/.gitkeep

# WRONG — the negation has no effect because logs/ is ignored before it is reached
logs/
!logs/.gitkeep
```

Use `logs/*` (contents), not `logs/` (directory), so the negation can work.

---

## Common Uses

This pattern appears wherever a project needs a directory to exist at runtime but should never commit its contents:

| Directory | What goes in it |
|---|---|
| `logs/` | Application log files |
| `tmp/` | Temporary files |
| `uploads/` | User-uploaded files |
| `cache/` | Cached data |
| `db/` | Local SQLite databases |

---

## Full Example

```
my-project/
├── logs/
│   └── .gitkeep       ← committed
├── src/
│   └── ...
└── .gitignore
```

`.gitignore`:

```gitignore
# Ignore all log files but keep the logs/ directory
logs/*
!logs/.gitkeep
```

After cloning, `logs/` exists and is ready. At runtime, log files written there never show up in `git status`.
