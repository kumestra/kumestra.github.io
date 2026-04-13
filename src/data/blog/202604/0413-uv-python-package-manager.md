---
title: "uv: The Modern Python Package and Project Manager"
description: A practical overview of uv — what it is, why it's fast, and how it simplifies daily Python development workflows.
pubDatetime: 2026-04-13T07:07:07Z
tags: [python, uv, tooling, package-manager, developer-tools]
---

`uv` is a fast Python package and project manager written in Rust, developed by Astral — the same team behind `ruff`. It replaces a fragmented set of Python tools with a single, unified binary.

## What uv Replaces

| Traditional tool | `uv` equivalent |
|---|---|
| `pip` | `uv pip install` |
| `pip-tools` | `uv pip compile` |
| `virtualenv` / `venv` | `uv venv` |
| `pyenv` | `uv python install` |
| `poetry` / `hatch` | `uv` (project management) |

## Why It's Fast

### Package installs

`uv` is 10–100x faster than `pip` due to its Rust implementation and aggressive use of a shared global cache (`~/.cache/uv`). Packages are not re-downloaded across projects.

### Python version downloads

When a project requires a Python version not present on the system, `uv` downloads it automatically — in seconds, not minutes. This is because `uv` uses **pre-built binaries** from the [python-build-standalone](https://github.com/indygreg/python-build-standalone) project rather than compiling from source.

| Method | What happens | Time |
|---|---|---|
| Compile from source (`pyenv` default) | Download source → `./configure` → `make` | 5–15 minutes |
| `uv python install` | Download pre-built binary → extract | Seconds |

This means you don't need `pyenv` or any other Python version manager — `uv` handles it transparently.

## Automatic Virtual Environment Management

When you run `uv sync` or `uv run` in a project directory, `uv` automatically creates and manages a `.venv` in the project root.

```
myproject/
├── .venv/          ← auto-created by uv
├── pyproject.toml
├── uv.lock
└── src/
```

- No manual `virtualenv` or `venv` creation needed
- No manual activation needed — `uv run` handles it
- Each project gets its own isolated environment
- The shared global cache keeps disk usage reasonable

## Daily Workflow

For most day-to-day work, three commands cover everything:

```bash
uv add requests      # need a new package
uv remove requests   # don't need it anymore
uv run pytest        # run anything in the project env
```

`uv sync` is occasionally useful — mainly after pulling changes where teammates added or removed dependencies. But even that can be skipped since `uv run` auto-syncs before running.

### One rule to follow

Always run `uv` commands from the **project root** — the directory containing `pyproject.toml`. This is how `uv` finds the correct config, Python version, and dependencies.

```
myproject/          ← run uv commands here
├── pyproject.toml
├── uv.lock
├── .venv/
└── src/
    └── main.py
```

## Other Useful Commands

### Project setup
```bash
uv init myproject        # create a new project
uv sync                  # install all deps, create/update .venv
uv python pin 3.12       # pin a Python version for the project
```

### Python version management
```bash
uv python install 3.13   # install a specific Python version
uv python list           # see available and installed versions
```

### One-off tools with uvx
```bash
uvx ruff check .         # run ruff without installing it
uvx black .
uvx mypy src/
```

### Inspecting state
```bash
uv tree                  # show dependency tree
uv pip list              # list installed packages
```

## Distribution for Apps (Not PyPI)

For apps that are not published to PyPI — internal tools, open source projects — `uv` enables a clean distribution model. Users only need to install `uv` once, then clone and run:

```bash
# Install uv (one-time)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Use the app
git clone <repo>
uv run main.py
```

`uv` handles the rest: downloads the right Python version, creates the virtual environment, installs dependencies, and runs the app.

### Python vs compiled languages

Unlike C, C++, Rust, or even Next.js, Python apps require no build or compile step. The source repo is the artifact.

| Language | What you ship |
|---|---|
| C / C++ / Rust | Compile → ship binary |
| Next.js | `npm run build` → ship `dist/` |
| Python | Ship source repo as-is |

The tradeoff is that source code is exposed — but for internal tools and open source this is rarely a concern.

## Open Source Projects

For open source Python projects, the experience is completely symmetrical between developers and users:

**Developer:**
```bash
git clone <repo>
uv run main.py
# make changes, push
```

**User or contributor:**
```bash
git clone <repo>
uv run main.py
```

The `uv.lock` file committed to the repo ensures everyone — users and contributors alike — gets identical dependency versions. No special packaging, deployment pipeline, or build artifacts required.

## Summary

`uv` is largely invisible in practice. It auto-creates environments, auto-downloads Python versions, and auto-syncs dependencies. The only habit to build is running commands from the project root. Everything else just works.
