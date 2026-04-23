---
title: "Why apt Can't Install ping — Virtual Packages Explained"
description: Why `sudo apt install ping` fails with two candidates, what a virtual package is, and which ping implementation to actually install on Ubuntu.
pubDatetime: 2026-04-23T02:08:54Z
tags: [linux, ubuntu, apt, networking, ping, debian]
---

## The Error 🤔

```bash
$ sudo apt install -y ping
E: Package 'ping' has no installation candidate
```

Before the error, apt hints at what's going on:

```
Package ping is a virtual package provided by:
  inetutils-ping 2:2.5-3ubuntu4.1
  iputils-ping 3:20240117-1ubuntu0.1
You should explicitly select one to install.
```

Two real packages both claim to provide `ping`. apt refuses to choose between them automatically.

---

## What Is a Virtual Package? 📦

A **virtual package** is a name that represents a capability rather than a concrete piece of software. It says: *"I need something that can do X"* — not *"I need this specific package."*

Any real package can declare that it **Provides** a virtual package name. When apt resolves a dependency or an install request:

- **One provider** → apt selects it automatically.
- **Two or more providers** → apt stops and asks you to pick explicitly.

Virtual packages are common wherever multiple implementations of a standard tool exist: `ping`, `awk`, `mail-transport-agent`, `www-browser`, and so on.

---

## The Two ping Implementations 🔀

| Package | Project | Notes |
|---|---|---|
| `iputils-ping` | [iputils][iputils] | Default on Ubuntu and most modern Debian-based distros; actively maintained |
| `inetutils-ping` | [GNU inetutils][inetutils] | Older GNU toolset implementation; less common on desktop/server Ubuntu |

Both implement the same ICMP echo protocol and accept broadly similar flags, but `iputils-ping` is the de-facto standard on Ubuntu and is what most documentation assumes.

---

## The Fix ✅

Install the implementation explicitly:

```bash
sudo apt install -y iputils-ping
```

Verify:

```bash
ping -c 3 1.1.1.1
```

---

## Why apt Doesn't Just Pick One 🧠

apt's refusal to guess is intentional. Auto-selecting between two equally valid providers could silently install the "wrong" one for a given environment (e.g., a system that uses the GNU toolset throughout might prefer `inetutils-ping` for consistency). Making the choice explicit keeps the system state predictable and reproducible.

[iputils]: https://github.com/iputils/iputils
[inetutils]: https://www.gnu.org/software/inetutils/
