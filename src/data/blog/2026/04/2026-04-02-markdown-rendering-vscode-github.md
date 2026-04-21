---
title: Getting VS Code Markdown Preview to Match GitHub
description: How to render GitHub Flavored Markdown in VS Code without losing your editor theme.
pubDatetime: 2026-04-02T02:20:36Z
tags: [markdown, vscode, github, gfm]
---

## The Problem

GitHub renders Markdown with features that VS Code's built-in preview doesn't support out of the box:

| Feature | VS Code Built-in | GitHub |
|---------|-----------------|--------|
| YAML frontmatter as table | ❌ | ✅ |
| Mermaid diagrams | ❌ | ✅ |
| Footnotes | ❌ | ✅ |
| Emoji rendering | ❌ | ✅ |
| Task list checkboxes | ❌ | ✅ |

There's an extension pack that solves this — `bierner.github-markdown-preview` — but it comes with a catch.

## The Extension Pack

`bierner.github-markdown-preview` bundles 6 extensions:

1. **`bierner.markdown-preview-github-styles`** — applies GitHub's CSS to the preview
2. **`bierner.markdown-mermaid`** — Mermaid diagram rendering
3. **`bierner.markdown-footnotes`** — footnote support
4. **`bierner.markdown-yaml-preamble`** — renders frontmatter as a table
5. **`bierner.markdown-emoji`** — emoji rendering
6. **`bierner.markdown-checkbox`** — task list checkboxes

Installing the full pack gives you pixel-perfect GitHub rendering. But extension #1 (`markdown-preview-github-styles`) **overrides your editor theme** — if you use Solarized, Dracula, or any custom theme, the preview switches to GitHub's own light/dark colors.

The `colorTheme` setting offers some control:

```json
"markdown-preview-github-styles.colorTheme": "auto"  // "auto" | "light" | "dark"
```

But it only picks between GitHub Light and GitHub Dark — it won't use your editor theme's colors.

## The Solution

Skip the full pack. Install the 5 feature extensions individually, excluding the styling one:

```json
{
    "recommendations": [
        "bierner.markdown-mermaid",
        "bierner.markdown-footnotes",
        "bierner.markdown-yaml-preamble",
        "bierner.markdown-emoji",
        "bierner.markdown-checkbox"
    ]
}
```

This gives you all of GitHub's Markdown features (frontmatter tables, Mermaid, footnotes, emoji, checkboxes) while your editor theme controls the preview's colors.

The only difference from GitHub is the background color — which is a non-issue since the content renders identically.

## What is Frontmatter?

Frontmatter is metadata placed at the very top of a Markdown file, enclosed in triple dashes (`---`). It's parsed by the processing tool, not rendered as content.

```yaml
---
title: My Post
date: 2026-04-01
tags: [python, ai]
draft: false
---
```

Common formats:
- **YAML** (between `---`) — most common
- **TOML** (between `+++`)
- **JSON** (between `{}`)

Used by static site generators (Jekyll, Hugo, Astro), documentation tools, and AI tooling like Claude Code skills.

## What is GFM?

GitHub Flavored Markdown (GFM) is GitHub's extension of CommonMark. It adds:

- **Fenced code blocks** with language identifiers for syntax highlighting
- **Tables** using pipe (`|`) syntax
- **Task lists** with `- [x]` / `- [ ]` checkboxes
- **Strikethrough** with `~~text~~`
- **Autolinked URLs** — raw URLs become clickable
- **Emoji shortcodes** — `:rocket:` becomes 🚀 (GitHub only, not portable)

GFM is the de facto standard for Markdown on the web. Use GFM features when they make a document clearer, but plain Markdown suffices for simple content — don't force tables or task lists where a paragraph works fine.
