---
title: Running Hermes Agent in Docker
description: A practical guide to running Nous Research's Hermes Agent from a Docker image with persistent config, TUI chat, gateway mode, backups, and skills.
pubDatetime: 2026-06-02T01:20:43Z
tags: [hermes-agent, docker, ai-agents, devops, skills]
---

Hermes Agent can run directly on a host machine, but Docker is a tidy way to keep the runtime isolated while still preserving configuration, memory, sessions, and skills across container runs.

The important idea is simple: the container is disposable, but the data directory is not. Mount a host directory into `/opt/data`, and Hermes stores its persistent state there.

## Official Links

- [Hermes Agent documentation](https://hermes-agent.nousresearch.com/docs/)
- [Hermes Docker guide](https://hermes-agent.nousresearch.com/docs/user-guide/docker/)
- [Docker Hub image](https://hub.docker.com/r/nousresearch/hermes-agent)
- [OpenRouter Hermes Agent app](https://openrouter.ai/apps/hermes-agent)

## Pull The Image

The normal image name is:

```text
nousresearch/hermes-agent:latest
```

Pull it with:

```bash
docker pull nousresearch/hermes-agent:latest
```

If Docker Hub access is slow or blocked, a registry proxy can be used, but that adds another party to trust. Direct Docker Hub is the cleaner path when it works.

## Create A Persistent Data Directory

Create a directory on the host to hold Hermes state:

```bash
mkdir -p ~/.hermes
```

In Docker commands, mount that directory into the container at `/opt/data`:

```bash
-v ~/.hermes:/opt/data
```

Hermes will use `/opt/data` for files such as:

```text
config.yaml
.env
skills/
sessions/
logs/
```

Treat this directory as sensitive. It can contain API keys, bot tokens, browser sessions, memories, and other private state.

## Run Setup

Run setup once:

```bash
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent setup
```

The flags mean:

| Flag or argument | Meaning |
|---|---|
| `docker run` | Start a new container from an image. |
| `-it` | Run interactively so the setup wizard can ask questions. |
| `--rm` | Delete the container after setup exits. |
| `-v ~/.hermes:/opt/data` | Store Hermes data on the host instead of inside the container. |
| `nousresearch/hermes-agent` | The Docker image to run. |
| `setup` | The Hermes command to execute inside the container. |

When setup finishes successfully, Hermes may print a message like `Ready to go!`. The container can stop afterward. That is expected because the setup command is complete.

## Start TUI Chat

After setup, start the terminal chat UI with the same data mount:

```bash
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent
```

Use this mode first when testing. It is easy to see startup errors, provider configuration issues, and model responses directly in the terminal.

## Start Gateway Mode

Gateway mode is the long-running process used for messaging integrations and background access:

```bash
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  -p 8642:8642 \
  nousresearch/hermes-agent gateway run
```

Check whether it is running:

```bash
docker ps
```

Stop and remove the gateway container with:

```bash
docker stop hermes
docker rm hermes
```

If configuration changes do not appear to take effect, restart the gateway container.

## Configure Hermes In Docker

Because Hermes is running in Docker, run configuration commands through Docker too, using the same mounted data directory.

View config:

```bash
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent config
```

Set a value:

```bash
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent config set <key> <value>
```

Rerun setup sections:

```bash
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent setup tools
```

```bash
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent setup gateway
```

```bash
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent setup terminal
```

Normal settings live in `config.yaml`. Secrets usually live in `.env`. Do not commit either file.

## Add A Skill

Hermes skills live under the persistent data directory:

```text
~/.hermes/skills/<category>/<skill-name>/SKILL.md
```

For example:

```text
~/.hermes/skills/custom/my-skill/SKILL.md
```

Create the directory and copy the skill file:

```bash
mkdir -p ~/.hermes/skills/custom/my-skill
cp /path/to/SKILL.md ~/.hermes/skills/custom/my-skill/SKILL.md
```

If the skill has helper scripts, templates, or references, copy the whole skill directory instead of only `SKILL.md`.

Then restart the active Hermes TUI or gateway so the new skill can be discovered.

## Back Up The Docker Image

If the image is already downloaded and needs to survive a machine snapshot restore, export it to a tar file:

```bash
docker save -o hermes-agent-latest.tar nousresearch/hermes-agent:latest
```

Move `hermes-agent-latest.tar` somewhere safe. Later, restore it with:

```bash
docker load -i hermes-agent-latest.tar
```

Verify:

```bash
docker image ls nousresearch/hermes-agent
```

This saves the Docker image only. It does not save Hermes state. Back up the mounted data directory separately:

```bash
tar -czf hermes-data.tar.gz ~/.hermes
```

## Understanding `docker tag` And Proxy Pulls

If a proxy mirror is used, the image may first be pulled under a proxy-prefixed name:

```bash
docker pull dockerproxy.example/nousresearch/hermes-agent:latest
```

Tagging gives that same local image the normal name:

```bash
docker tag dockerproxy.example/nousresearch/hermes-agent:latest nousresearch/hermes-agent:latest
```

Removing the proxy tag removes only the extra label:

```bash
docker rmi dockerproxy.example/nousresearch/hermes-agent:latest
```

The image data remains because it is still tagged as:

```text
nousresearch/hermes-agent:latest
```

## Permission Gotcha

Depending on how the container runs, the mounted data directory may be created with ownership that the host user cannot read. On the host, it can look empty even though Hermes has written files.

Symptoms include:

```text
Permission denied
```

or a directory listing like:

```text
drwx------ nobody nogroup hermes-data
```

Confirm contents with:

```bash
sudo ls -la ~/.hermes
```

This does not necessarily mean setup failed. If Hermes printed its success message and TUI or gateway mode starts correctly with the same mount, the setup is working. The issue is host-side file ownership and permissions.

## Practical Flow

For a clean first run:

```bash
mkdir -p ~/.hermes
docker pull nousresearch/hermes-agent:latest
docker run -it --rm -v ~/.hermes:/opt/data nousresearch/hermes-agent setup
docker run -it --rm -v ~/.hermes:/opt/data nousresearch/hermes-agent
```

After interactive chat works, start the gateway:

```bash
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  -p 8642:8642 \
  nousresearch/hermes-agent gateway run
```

That is the durable Docker pattern: disposable containers, persistent `/opt/data`, and the same mount reused for setup, chat, gateway, config, backups, and skills.
