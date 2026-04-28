---
title: Package Managers on Windows — winget, Chocolatey, and Scoop Explained
description: A concept note covering the main Windows package managers, how they work under the hood, who uses them, and which one to pick.
pubDatetime: 2026-04-28T08:35:17Z
tags: [windows, package-manager, winget, chocolatey, scoop, developer-tools]
---

## The State of Package Managers on Windows

Unlike Linux (where `apt` or `dnf` is just how you install things) or macOS (where Homebrew is the de facto developer standard), Windows grew up with a GUI-first culture. Downloading `.exe` installers from official websites *is* the norm for most users — and it still is.

Package managers on Windows are mostly used by **developers and sysadmins**, not general users.

---

## The Main Options

### winget — Windows Package Manager

Microsoft's official CLI package manager. Ships with Windows 10 and 11 — no installation needed.

```cmd
winget install Spotify.Spotify
winget install Google.Chrome
winget install Valve.Steam
winget upgrade --all
```

### Chocolatey

The most popular third-party option, launched in 2011. Predates winget by nearly a decade, with a large package repository and strong enterprise adoption.

```cmd
choco install spotify
choco install googlechrome
choco install steam
```

### Scoop

Installs apps into your user directory (`~\scoop\apps\`) in a portable style — no admin rights required. Focused on developer tools rather than general consumer software.

---

## Comparison

| | winget | Chocolatey | Scoop |
|---|---|---|---|
| Made by | Microsoft | Community | Community |
| Launched | 2020 | 2011 | 2013 |
| Admin required | Sometimes | Yes (usually) | No |
| Focus | General apps | General + enterprise | Developer tools |
| Ships with Windows | ✅ Yes | ❌ No | ❌ No |
| Maturity | Newer | Mature | Mature |

---

## How They Work Under the Hood

These package managers don't host the software themselves. They store a *manifest* (a recipe) that knows where to download the official installer and how to run it silently.

When you run `winget install Spotify.Spotify`, it:

1. Looks up Spotify's manifest in the package repository
2. Downloads the official Spotify installer from Spotify's servers
3. Runs it silently in the background

The end result is the same app you'd get from downloading manually — just without the clicking.

---

## Does the Install Location or Config Differ?

For winget and Chocolatey — generally **no**. They run the official installer, so the result is identical:

- Same install location (`C:\Program Files\...`)
- Same config files and app behavior
- Same built-in update mechanism
- Appears in Add/Remove Programs the same way

**Scoop is the exception** — it installs into `~\scoop\apps\` in a portable style, which can occasionally cause differences for apps that expect to live in `Program Files`.

For apps with their own update managers (like Adobe Creative Cloud or Steam), the package manager just triggers the initial install — the app's own ecosystem takes over from there.

---

## The Real Win: `upgrade --all`

The main practical benefit over manual downloads is updating everything at once:

```cmd
winget upgrade --all
```

This updates Chrome, Spotify, Steam, VS Code — everything winget knows about — in one command, instead of clicking "update" inside each app individually.

---

## Who Uses What

| Audience | Likely choice |
|---|---|
| Casual developer setting up a machine | winget |
| Enterprise / IT automation | Chocolatey |
| Developer wanting no-admin portable installs | Scoop |
| General (non-developer) user | Downloads from website |

---

## Which Is Most Popular?

**Chocolatey** was dominant for years simply because it existed when nothing else did. **winget** is now catching up fast and likely overtaking it for new setups — it ships with Windows, requires no install step, and Microsoft actively promotes it in official documentation.

If you're starting fresh with no prior preference, **winget** is the practical default today.

---

## Why Windows Lags Behind Linux/macOS

On Linux, there's no "official website" culture for most software — the package manager *is* how you install things, so everyone uses it. On Windows, website installers came first and general users never felt the problem that package managers solve. The developer-only adoption on Windows reflects that history, not a flaw in the tools themselves.
