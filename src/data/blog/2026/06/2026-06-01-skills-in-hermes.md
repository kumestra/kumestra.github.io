---
title: Skills in Hermes Agent
description: A practical guide to how Hermes Agent skills work, how to create one, and how to expose repo-managed skills through symlinks.
pubDatetime: 2026-06-01T09:09:54Z
tags: [hermes-agent, skills, ai-agents, automation, telegram]
---

Hermes Agent supports skills: small, discoverable instruction packages that teach the agent how to handle a particular workflow.

A skill is usually a directory containing a `SKILL.md` file. That file includes frontmatter metadata and step-by-step instructions. It can also include helper scripts, templates, references, or other files that the agent can use when the skill is invoked.

Skills are useful when a workflow is repeated often enough that it should not live only in conversation memory. They turn a pattern into a reusable tool.

## What a skill looks like

A minimal Hermes skill can be as small as this:

```markdown
---
name: current-time
description: "Return the current time for locations such as London, New York, Tokyo, and UTC."
version: 1.0.0
author: dotfiles
license: MIT
dependencies: []
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [time, timezone, utility, test-skill]
---

# Current Time

Use this skill when the user asks for the current time in one or more cities,
countries, or time zones.

Run the helper script:

```bash
python3 scripts/current_time.py london "new york" tokyo
```
```

The frontmatter tells Hermes how to identify and search for the skill. The body tells the agent what to do after the skill is loaded.

## Skill directory layout

A normal skill directory looks like this:

```text
~/.hermes/skills/<category>/<skill-name>/SKILL.md
```

For example:

```text
~/.hermes/skills/utility/current-time/SKILL.md
~/.hermes/skills/utility/current-time/scripts/current_time.py
```

The category is mostly organizational. The skill name and description matter more for discovery.

## Two ways to trigger a skill

Hermes can use a skill in two main ways.

### 1. Explicit invocation

The strongest trigger is naming the skill directly:

```text
/current-time london beijing
```

or:

```text
Use current-time for london and beijing
```

This tells Hermes exactly which skill to load. Explicit invocation is the best path when testing a new skill.

### 2. Semantic invocation

Hermes can also choose a skill from the request itself:

```text
What time is it now in London and Beijing?
```

If the skill metadata says it handles time, timezone, or location-based clock questions, Hermes may load it automatically.

Semantic invocation is convenient, but it is not as deterministic as naming the skill. The better the `name`, `description`, and `tags`, the more likely Hermes is to pick the right skill.

## A practical test skill: current time

A good first skill is a current-time utility. It is simple, easy to verify, and demonstrates the full flow:

- Hermes discovers the skill.
- The skill loads from `SKILL.md`.
- Hermes follows the instructions.
- Hermes runs a helper script.
- The result is returned to the user.

The helper can use Python's standard `zoneinfo` module:

```python
from datetime import datetime
from zoneinfo import ZoneInfo

print(datetime.now(ZoneInfo("Europe/London")).strftime("%Y-%m-%d %H:%M %Z"))
```

For a real helper, keep a small alias table:

| Alias | IANA timezone |
|---|---|
| London | `Europe/London` |
| New York | `America/New_York` |
| Beijing | `Asia/Shanghai` |
| Tokyo | `Asia/Tokyo` |
| UTC | `UTC` |

This lets users type natural city names while the script still uses precise time zone IDs.

## Repo-managed skills with symlinks

One useful pattern is to keep custom skills in a dotfiles or config repository, then symlink them into Hermes.

Repository copy:

```text
repo/hermes/skills/utility/current-time/
```

Hermes live path:

```text
~/.hermes/skills/utility/current-time
```

Symlink:

```bash
mkdir -p ~/.hermes/skills/utility
ln -s /path/to/repo/hermes/skills/utility/current-time \
  ~/.hermes/skills/utility/current-time
```

This keeps the skill version-controlled while still letting Hermes discover it from its normal skills directory.

For a single-file skill that already exists elsewhere, make a Hermes skill directory and symlink only the `SKILL.md`:

```bash
mkdir -p ~/.hermes/skills/productivity/blog
ln -s /path/to/repo/codex/skills/blog/SKILL.md \
  ~/.hermes/skills/productivity/blog/SKILL.md
```

That pattern is handy when sharing a skill between multiple agents or toolchains.

## Verifying a skill

After adding or linking a skill, check whether Hermes can see it:

```bash
hermes skills list
```

Look for a row like:

```text
current-time    utility    local    local    enabled
```

Then invoke it in Hermes:

```text
/current-time london beijing
```

If Hermes loads the skill and runs its helper script, the skill is working.

## Skills in Telegram

If Hermes is connected to Telegram through `hermes gateway`, the same skills can be used from Telegram.

For example:

```text
/current-time london beijing
```

or:

```text
Use current-time for london and beijing
```

One detail: Telegram's command menu is not the same as Hermes' skill list. The menu may only show registered bot commands, not every skill under `~/.hermes/skills`.

Also, Telegram command names usually prefer underscores. A Hermes skill named `current-time` can still work when typed manually, but it may not appear cleanly in Telegram autocomplete. If Telegram command-menu ergonomics matter, consider a skill name like:

```text
current_time
```

For pure Hermes usage, hyphenated names are readable and work well.

## Restarting after changes

If a newly added skill does not appear immediately:

```bash
hermes skills list
```

Then restart the active Hermes chat or gateway:

```bash
hermes
hermes gateway
```

For Telegram, stop and restart the gateway process:

```bash
Ctrl+C
hermes gateway
```

This gives Hermes a fresh chance to discover newly linked files.

## What belongs in a skill

A skill is a good fit for:

- Repeated workflows
- Project-specific procedures
- CLI recipes
- Blog or documentation generation
- API workflows
- Formatting rules
- Tool-specific instructions
- Small helper scripts

A skill is not a good fit for secrets. Do not put API keys, tokens, user IDs, passwords, or private account details in `SKILL.md`.

Use environment variables or private config files for secrets, and keep the skill focused on instructions.

## Final checklist

To add a custom Hermes skill:

- Create `SKILL.md` with a clear `name` and `description`.
- Put it under `~/.hermes/skills/<category>/<skill-name>/`.
- Add scripts or references if the workflow needs them.
- Run `hermes skills list`.
- Test explicit invocation first.
- Test semantic invocation after that.
- Restart `hermes gateway` if using Telegram.
- Keep the source in a repo and symlink it if you want version control.

The most important part is the description. Hermes uses it to decide when the skill is relevant. A short, specific description often works better than a clever one.
