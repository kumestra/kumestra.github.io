---
title: Installing Hermes Agent with OpenRouter and Telegram
description: A practical walkthrough for installing Hermes Agent on Linux or WSL, configuring OpenRouter, and enabling Telegram gateway access.
pubDatetime: 2026-06-01T07:56:36Z
tags: [hermes-agent, openrouter, telegram, wsl, ai-agents]
---

Hermes Agent is an open source CLI AI agent from Nous Research. It can run locally, use external model providers, search files and the web, and connect to messaging platforms such as Telegram.

This note captures a practical install path for a Linux or WSL machine using an OpenRouter API key and Telegram bot access.

## Install Hermes

The official one-line installer is:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

A more inspectable version is:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh -o /tmp/hermes-install.sh
less /tmp/hermes-install.sh
bash /tmp/hermes-install.sh
```

During installation, Hermes may check for Python, `uv`, Git, Node.js, `ripgrep`, and `ffmpeg`.

## Optional Packages

Hermes may ask whether to install optional system packages:

```text
Install ripgrep for faster file search and ffmpeg for TTS voice messages? [Y/n]
```

For most local installs, answering `Y` is fine.

| Package | Purpose |
|---|---|
| `ripgrep` | Provides the `rg` command, a fast search tool for files and code. |
| `ffmpeg` | Handles audio conversion for voice and text-to-speech workflows. |

Hermes itself does not need root access, but installing these system packages can require `sudo`.

## Choose Full Setup for OpenRouter

If you have an OpenRouter API key, choose:

```text
Full setup - configure everything
```

Then select:

```text
OpenRouter
```

OpenRouter lets Hermes use many different hosted models through one account. In a manual OpenAI-compatible setup, the usual base URL is:

```text
https://openrouter.ai/api/v1
```

If Hermes offers a direct OpenRouter option, use that instead of manually entering an OpenAI-compatible endpoint.

## Terminal Backend

Hermes asks where terminal commands should run:

```text
Select terminal backend:
  Local
  Docker
  Modal
  SSH
  Daytona
  Singularity/Apptainer
  Keep current (local)
```

For WSL or a normal personal Linux machine, choose local:

```text
Keep current (local)
```

This means Hermes runs commands directly on the current machine. Docker or cloud sandboxes can be useful later, but local is the simplest working default.

## Telegram Messaging

To use Hermes through Telegram, create a Telegram bot and provide the bot token during setup. Hermes will also ask for an allowlist of user IDs. This restricts who can talk to the bot.

Hermes may ask:

```text
Use your user ID as the home channel? [Y/n]
```

For a direct Telegram bot chat, answer `Y`. The home channel is where Hermes sends notifications, cron results, and cross-platform messages.

## Image Generation, TTS, Vision, and Search

Hermes setup includes several optional provider screens.

| Screen | Good default |
|---|---|
| Image generation provider | Skip unless you already have an image API provider. |
| TTS provider | Microsoft Edge TTS is a simple free default. |
| Vision backend | OpenRouter with Gemini is a good choice if already using OpenRouter. |
| Search provider | DuckDuckGo is the easiest no-key option. |

These settings can be changed later with:

```bash
hermes setup
hermes config
hermes config edit
```

## Reload the Shell

After installation, reload the shell so the `hermes` command is on `PATH`:

```bash
source ~/.bashrc
```

Then start Hermes in the terminal:

```bash
hermes
```

If Hermes opens its chat UI and responds, the terminal side is working.

## Start the Telegram Gateway

On WSL, Hermes may fail to install the gateway as a `systemd` service:

```text
Systemd install failed. You can start manually: hermes gateway
```

That is not fatal. Open a second terminal or tmux pane and run:

```bash
source ~/.bashrc
hermes gateway
```

Keep that process running. It bridges Telegram messages to Hermes and also runs the cron scheduler.

## Verify the Install

A successful setup has three signs:

- `hermes` starts and replies in the terminal.
- `hermes gateway` starts without exiting.
- Sending a message to the Telegram bot produces a reply.

Some installs may show warnings such as:

```text
Auxiliary Nous client unavailable: no Nous authentication found
```

That does not necessarily mean the install failed. If OpenRouter is the main model provider, Hermes can still work without a Nous login. The warning only means optional Nous auxiliary services are unavailable.

## Useful Paths

Hermes stores its main files under the home directory:

```text
~/.hermes/config.yaml
~/.hermes/.env
~/.hermes/hermes-agent
~/.hermes/sessions
~/.hermes/logs
```

The `.env` file contains API keys and bot tokens. Treat it as secret.

## Useful Commands

```bash
hermes                 # Start terminal chat
hermes setup           # Reconfigure providers and settings
hermes config          # View configuration
hermes config edit     # Edit configuration in an editor
hermes gateway         # Start messaging gateway manually
hermes update          # Update Hermes
```

## Final State

For this setup, the install is complete when terminal chat works and Telegram replies through the gateway. The only remaining optional improvement is auto-starting the gateway. On WSL, keeping `hermes gateway` open in a second terminal or tmux pane is usually the simplest approach.
