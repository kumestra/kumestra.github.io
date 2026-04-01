---
title: "Understanding All Frontmatter Fields in AstroPaper"
description: "A complete guide to every supported frontmatter field in AstroPaper — what each one does, when to use it, and what happens if you leave it out."
pubDatetime: 2026-04-01T10:00:00Z
modDatetime: 2026-04-01T12:00:00Z
author: "Evanara Kumestra"
featured: true
draft: false
tags: ["astropaper", "guide", "blogging"]
ogImage: ""
canonicalURL: ""
hideEditPost: false
timezone: "Asia/Bangkok"
---

Frontmatter is the block of metadata at the top of every blog post, enclosed between `---` lines. AstroPaper uses it to control how your post is displayed, indexed, and shared. Here is a breakdown of every supported field.

## Table of contents

## Required Fields

These fields must be present in every post or the build will fail.

### `title`

```yaml
title: "My Post Title"
```

The title of your post. Shown in the browser tab, post header, post cards, and OG images.

### `description`

```yaml
description: "A short summary of what this post is about."
```

A brief summary of the post. Used in SEO meta tags, OG/social preview cards, and post listings.

### `pubDatetime`

```yaml
pubDatetime: 2026-04-01T10:00:00Z
```

The publication date and time in ISO 8601 format. Posts are sorted by this field — newest first. Scheduled posts (set in the future) are hidden until their time arrives, within a `scheduledPostMargin` window defined in `config.ts`.

## Optional Fields

These fields are optional. Omitting them uses a default value or simply disables the feature.

### `author`

```yaml
author: "Evanara Kumestra"
```

The name of the post's author. Defaults to `SITE.author` from `src/config.ts` if not set. Useful when a blog has multiple contributors.

### `modDatetime`

```yaml
modDatetime: 2026-04-01T12:00:00Z
```

The date the post was last modified. When set, AstroPaper displays a "Updated" timestamp alongside the original publish date. Set to `null` to explicitly clear it.

### `featured`

```yaml
featured: true
```

When `true`, the post appears in the **Featured** section on the homepage, above the recent posts list. Only a handful of posts should be featured at once.

### `draft`

```yaml
draft: true
```

When `true`, the post is excluded from production builds and does not appear anywhere on the site. It is still visible during local development. Use this for work-in-progress posts.

### `tags`

```yaml
tags: ["guide", "astropaper", "blogging"]
```

An array of tags for the post. Tags are used to group posts and generate tag pages at `/tags/<tag>`. Defaults to `["others"]` if omitted.

### `ogImage`

```yaml
ogImage: "my-custom-image.jpg"
```

A custom image used when the post is shared on social media (Open Graph). Can be a path to a file in `public/` or an imported image. If omitted and `dynamicOgImage` is enabled in `config.ts`, an OG image is generated automatically from the post title.

### `canonicalURL`

```yaml
canonicalURL: "https://example.com/original-post"
```

Overrides the canonical URL in the page's `<head>`. Use this when republishing content that originally appeared elsewhere, to tell search engines where the original lives.

### `hideEditPost`

```yaml
hideEditPost: true
```

When `true`, hides the "Edit page" link that normally appears at the bottom of the post. The global edit link is configured in `config.ts` under `editPost`. Use this to hide it on specific posts where editing doesn't make sense.

### `timezone`

```yaml
timezone: "Asia/Bangkok"
```

The IANA timezone used to interpret `pubDatetime` and `modDatetime` for this post. Overrides the global `SITE.timezone` from `config.ts`. Useful when publishing from a different timezone or scheduling posts precisely. A full list of valid values can be found at [Wikipedia — List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

## Quick Reference

| Field | Required | Default |
|---|---|---|
| `title` | yes | — |
| `description` | yes | — |
| `pubDatetime` | yes | — |
| `author` | no | `SITE.author` |
| `modDatetime` | no | `null` |
| `featured` | no | `false` |
| `draft` | no | `false` |
| `tags` | no | `["others"]` |
| `ogImage` | no | auto-generated |
| `canonicalURL` | no | current page URL |
| `hideEditPost` | no | `false` |
| `timezone` | no | `SITE.timezone` |
