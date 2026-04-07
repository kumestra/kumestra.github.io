---
title: "Jekyll, Static Site Generators, and Static Site Hosting"
description: "A concept note covering static site generators, static site hosting services, and why Jekyll with GitHub Pages remains the simplest way to publish a website."
pubDatetime: 2026-04-07T01:59:33Z
tags: [jekyll, github-pages, static-site-generator, web-hosting]
---

## What Is a Static Site Hosting Service?

GitHub Pages (`github.io`) is GitHub's free static site hosting service. It serves websites directly from a GitHub repository at `<username>.github.io` URLs.

Other popular static site hosting services include:

| Service | Notable Feature |
|---|---|
| **Netlify** | Built-in CI/CD, form handling, serverless functions |
| **Vercel** | Made by the Next.js team, strong for JS-based SSGs |
| **Cloudflare Pages** | Fast global CDN, generous free tier |
| **AWS S3 + CloudFront** | More manual setup, but highly scalable |
| **Firebase Hosting** | Google's offering, integrates well with Firebase services |
| **Render** | Also supports dynamic apps alongside static sites |

Most of these offer free tiers for static sites, custom domains, and HTTPS — similar to GitHub Pages.

## What Is a Static Site Generator?

A static site generator (SSG) takes plain text files — Markdown, HTML templates, config — and compiles them into a complete static website (plain HTML/CSS/JS) ready to serve.

The workflow is straightforward:

```mermaid
flowchart LR
    A["✍️ Write\n(Markdown / templates)"] --> B["⚙️ Generate\n(Jekyll, Hugo, etc.)"]
    B --> C["🌐 Host\n(GitHub Pages, Netlify, etc.)"]
```

Popular SSGs beyond Jekyll include Hugo, Next.js, Eleventy, and Astro. Each requires either a local dev environment or a CI/CD pipeline (like GitHub Actions) to build the site before hosting.

## Code and Hosting Are Separate Concerns

When people choose a hosting service other than GitHub Pages, their source code typically still lives on GitHub. For example, a repo on GitHub can be connected to Netlify — Netlify watches for pushes, runs the build, and serves the result on its own CDN.

GitHub is used for **version control**; the hosting service is used for **serving the site**. You can mix and match.

## Why Jekyll + GitHub Pages Is the Simplest Combo

Jekyll is the most popular SSG specifically for GitHub Pages because of one key advantage: **GitHub Pages runs Jekyll natively** — no build tooling required on your end. No Node, no Ruby, no CI config.

The entire setup:

1. Create a repo named `<username>.github.io`
2. Add Markdown files with the right structure and front matter
3. Push
4. Site is live ✅

Every other SSG + hosting combo requires either a local dev environment or CI/CD configuration. Jekyll + GitHub Pages is the only combination where the hosting service does the build automatically with zero configuration.

The tradeoff is that Jekyll is less flexible and slower to build than modern SSGs — but for blogs and simple sites, the simplicity is hard to beat.

## Jekyll Folder Structure

A typical Jekyll project:

```
.
├── _config.yml          # Site settings (title, URL, theme, etc.)
├── _posts/              # Blog posts (named YYYY-MM-DD-title.md)
│   └── 2026-04-07-my-first-post.md
├── _layouts/            # HTML templates
│   ├── default.html
│   └── post.html
├── _includes/           # Reusable HTML snippets (nav, footer)
├── _data/               # YAML/JSON data files
├── _sass/               # Sass stylesheets
├── assets/              # Static files (CSS, JS, images)
├── _drafts/             # Unpublished posts (no date prefix)
├── index.md             # Homepage
└── about.md             # Other pages
```

The `_` prefix is a Jekyll convention — those folders are processed by Jekyll but not copied directly to the output. Everything else gets copied as-is.

A **minimal setup** needs only three things:

```
.
├── _config.yml
├── _posts/
│   └── 2026-04-07-hello.md
└── index.md
```

### Key Concepts

- **`_config.yml`** — site-wide settings (title, theme, URL)
- **`_posts/`** — blog posts as Markdown with date-based filenames
- **`_layouts/`** — HTML templates that wrap your content
- **Front matter** — YAML metadata at the top of each file (between `---` lines) that tells Jekyll how to process it
- **Liquid** — a templating language (e.g., `{{ page.title }}`, `{% for post in site.posts %}`) used in layouts

## Setting Up Jekyll: Local vs. GitHub-Only

There are two approaches:

### Option 1: Let GitHub Handle It (No Local Ruby)

Create the folder structure, write Markdown with front matter, push to GitHub. GitHub Pages runs Jekyll automatically and serves the result. The downside is you can't preview locally — you push and wait.

### Option 2: Local Ruby Setup

Install Ruby and Jekyll on your machine, run `jekyll serve` to preview at `localhost:4000`, iterate quickly, then push when ready.

Many people start with Option 1 and only set up Ruby locally when they want faster feedback.

## Bootstrapping Without Local Tools

For the "no local tooling" approach, there are several ways to get the initial folder structure:

- **GitHub template repos** — clean starting points designed to be forked/copied, usually minimal with placeholder content
- **Fork a theme** — copy someone's complete, styled Jekyll site and replace their content (more files, but more visual polish out of the box)
- **Manually** — just create the folders and files yourself; the minimal structure is only three items
- **GitHub repo Settings → Pages** — choose a theme directly and GitHub sets up the Jekyll structure for you
