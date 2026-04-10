---
title: "Line Length Conventions in Code: Focus on Bash"
description: A quick reference on recommended maximum line lengths for code, with specific guidance for Bash scripts.
pubDatetime: 2026-04-10T01:03:02Z
tags: [bash, coding-style, conventions, shell-scripting]
---

## Common Line Length Standards

Different languages and communities have settled on slightly different maximums:

| Characters | Usage |
|---|---|
| **80** | Traditional standard — Python (PEP 8), Bash, many linters' default |
| **100** | Modern practical middle ground |
| **120** | Google style guides, IntelliJ default, common in Java and Go |

For most projects, **80–100 characters** strikes a good balance. It keeps code readable in split-pane editors, diffs, and code reviews without excessive wrapping.

Consistency within a project matters more than the exact number — follow whatever the project's linter or formatter enforces.

## Bash: Stick to 80 Characters

Bash scripts have a stronger case for the **80-character** limit than most languages:

- Bash is often read in **terminals**, which default to 80 columns
- Scripts are frequently edited over **SSH** or in minimal editors
- Long pipelines and flag-heavy commands are common and benefit from explicit wrapping

## Line Continuation with `\`

When commands exceed the limit, use backslash (`\`) continuation to break them across lines:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}' \
  "https://example.com/api"
```

This keeps each flag or argument on its own line, making the command easier to scan and modify.
