---
title: Git Commit Message Format and Standards
description: A practical guide to git commit message conventions, focusing on Conventional Commits — the most widely adopted standard today.
pubDatetime: 2026-04-18T05:44:14Z
tags: [git, conventional-commits, semver, best-practices, version-control]
---

Good commit messages are one of the highest-leverage habits in software development. They make `git log` useful, enable automation, and communicate intent to future maintainers (including yourself). This post covers the major standards and dives deep into Conventional Commits — the de facto standard today.

## Popular Standards at a Glance

| Standard | Format | Best For |
|---|---|---|
| **Conventional Commits** | `type(scope): description` | Automated changelogs, semver |
| **Angular Commit Guidelines** | `type(scope): subject` + body | Angular projects, CI/CD |
| **50/72 Rule** | Plain subject + body | General open source |
| **Gitmoji** | `✨ description` | Visual, expressive style |
| **Semantic Versioning commits** | Tied to `feat`/`fix`/`BREAKING CHANGE` | Library/package authors |

### Which is most widely used?

**Conventional Commits** is the most widely adopted standard today. It's used by Angular, Vue, Electron, Node.js, Vite, Prettier, ESLint, and thousands more projects. It has the richest tooling ecosystem (`commitlint`, `husky`, `semantic-release`, `release-please`) and is machine-readable for automation.

The **50/72 plain style** remains common in older open source projects (Linux kernel, Git itself, PostgreSQL) but has no machine-readable structure.

---

## Conventional Commits

The official specification is at **[conventionalcommits.org](https://www.conventionalcommits.org/en/v1.0.0/)**, currently at v1.0.0 (stable since 2019).

### Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### A full example

```
feat(auth)!: redesign OAuth2 login flow

Previously users were redirected to a separate page. Now login
happens inline via a modal, reducing drop-off during onboarding.

BREAKING CHANGE: /login/password endpoint has been removed.
Closes #42
Co-authored-by: Jane <jane@example.com>
```

---

## Type

The **type** is the first part of the commit — it categorizes what kind of change was made.

| Type | When to use |
|---|---|
| `feat` | New feature for the user |
| `fix` | Bug fix for the user |
| `docs` | Documentation changes |
| `style` | Formatting, whitespace — no logic change |
| `refactor` | Restructured code, no feature or fix |
| `test` | Adding or fixing tests |
| `chore` | Tooling, dependencies, config |
| `perf` | Performance improvement |
| `ci` | CI/CD pipeline changes |
| `revert` | Reverting a previous commit |

The spec **only mandates** that `feat` and `fix` carry SemVer meaning. The other types are widely adopted conventions but not strictly defined by the spec.

---

## Scope (optional)

The **scope** narrows down *which part of the codebase* changed. It goes in parentheses after the type.

```
feat(auth): add OAuth2 login
fix(api): return 404 when user not found
chore(deps): upgrade react to v19
```

Scope is freeform — your team defines what scopes make sense. Common patterns:

- **By module**: `auth`, `api`, `db`, `ui`
- **By feature**: `payment`, `search`, `cart`
- **By file type**: `config`, `deps`, `ci`

Omit scope entirely when the change is general or the project is small — both are valid per the spec.

---

## Body (optional)

Any commit can have a body. Separate it from the subject with a **blank line**.

```
fix(auth): reject expired tokens

Previously expired tokens were silently accepted.
Now returns 401 with a clear error message.
```

**Rule**: subject says *what*, body says *why*. The diff already shows *how*.

Write a body when:
- The reason for the change is non-obvious
- There's a workaround or subtle constraint future devs need to know
- Context would otherwise be lost

Skip the body when the subject line is self-explanatory.

---

## Footer (optional)

Footers appear after a blank line following the body. They provide **metadata** about the commit.

```
feat(payment): add Stripe checkout

Closes #88
Co-authored-by: Jane <jane@example.com>
BREAKING CHANGE: PayPal integration has been removed
```

### Common footer tokens

| Token | Purpose |
|---|---|
| `Closes #123` | Links and closes a GitHub/GitLab issue |
| `Fixes #123` | Same as Closes |
| `Refs #123` | References an issue without closing it |
| `BREAKING CHANGE: <desc>` | Declares a breaking change (triggers MAJOR bump) |
| `Reviewed-by: Name` | Credits a reviewer |
| `Co-authored-by: Name <email>` | Credits a co-author |

Footer format is `token: value` or `token #value` for issue refs. `BREAKING CHANGE` is the only token with special meaning in the spec — it's required to trigger a major version bump alongside the `!` shorthand.

---

## Relationship to Semantic Versioning (SemVer)

SemVer is a versioning standard with the format `MAJOR.MINOR.PATCH` (e.g., `2.1.3`).

| Part | When to increment | Example |
|---|---|---|
| **MAJOR** | Breaking change — not backward compatible | `1.0.0` → `2.0.0` |
| **MINOR** | New feature — backward compatible | `1.0.0` → `1.1.0` |
| **PATCH** | Bug fix — backward compatible | `1.0.0` → `1.0.1` |

Conventional Commits maps directly to SemVer:

```
feat      →  MINOR bump
fix       →  PATCH bump
BREAKING  →  MAJOR bump
```

This mapping powers automated release tools. Given this commit history:

```
fix: correct null check        → 1.0.1
feat: add export button        → 1.1.0
feat!: redesign API response   → 2.0.0
```

Tools like `semantic-release` or `release-please` read the commits and **automatically determine the next version** — no manual version bumping required.

The official SemVer spec is at **[semver.org](https://semver.org)**.

---

## The 50/72 Rule (classic style)

Still used in Linux kernel, Git, PostgreSQL, and other traditional open source projects:

- **Subject**: 50 chars max, imperative mood, no trailing period
- **Blank line** separating subject from body
- **Body**: wrap at 72 chars, explain what and why

```
Add user authentication via OAuth2

Previously all endpoints were unprotected. This adds OAuth2 token
validation middleware so that only authenticated clients can access
the API.

Closes #42
```

No machine-readable structure, so it doesn't integrate with automation tools the way Conventional Commits does.

---

## Key Principles (both standards agree)

1. **Imperative mood** — "Add feature" not "Added" or "Adding"
2. **Subject explains what, body explains why** — the diff shows how
3. **One logical change per commit** — makes `git bisect` and `git revert` reliable
4. **Reference issues** in the footer

### What to avoid

- `fix stuff`, `WIP`, `misc changes` — too vague
- Bodies that just restate the subject line
- Giant commits mixing unrelated changes

---

## Tooling

If you adopt Conventional Commits, these tools integrate with it:

| Tool | Purpose |
|---|---|
| `commitlint` | Lints commit messages against the spec |
| `husky` | Git hooks to run commitlint pre-commit |
| `semantic-release` | Automated versioning and changelog from commits |
| `release-please` | Google's alternative to semantic-release |
| `changelogen` | Changelog generator for monorepos |
