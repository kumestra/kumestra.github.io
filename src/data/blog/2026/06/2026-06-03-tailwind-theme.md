---
title: Understanding @theme in Tailwind CSS
description: A practical guide to how Tailwind CSS uses @theme variables to create utility classes such as bg-background.
pubDatetime: 2026-06-03T04:16:25Z
tags: [tailwind-css, css, frontend]
---

Tailwind CSS has moved more of its configuration into CSS itself. One important part of that shift is the `@theme` directive.

At first glance, this can look like normal CSS:

```css
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --font-sans: var(--font-geist-sans);
  --font-mono: var(--font-geist-mono);
}
```

But `@theme` is not a standard browser CSS keyword. It is a Tailwind-specific at-rule. Tailwind reads it at build time and uses it to understand which design tokens should become utility classes.

## What `@theme` means

`@theme` defines values that belong to Tailwind's design system.

Those values are usually written as CSS custom properties:

```css
@theme {
  --color-brand: #2563eb;
  --spacing-card: 24px;
  --font-display: Inter, sans-serif;
}
```

Each variable name tells Tailwind what kind of token it is. For example:

| Theme variable | Token type | Example utilities |
| --- | --- | --- |
| `--color-brand` | Color | `bg-brand`, `text-brand`, `border-brand` |
| `--spacing-card` | Spacing | `p-card`, `m-card`, `gap-card` |
| `--font-display` | Font family | `font-display` |

So the developer writes a named design value once, and Tailwind makes that value available through utility classes.

## What is a design token?

A design token is a named value for a design decision.

Instead of repeating raw values throughout a project:

```css
.card {
  background: #ffffff;
  color: #171717;
  padding: 24px;
}
```

we give those values names:

```css
:root {
  --background: #ffffff;
  --foreground: #171717;
  --spacing-card: 24px;
}
```

Those names make the design easier to reuse and change. If the background color changes later, the update can happen in one place instead of across many components.

`@theme` connects those named values to Tailwind's utility system.

## How `--color-background` becomes `bg-background`

Consider this theme block:

```css
@theme inline {
  --color-background: var(--background);
}
```

Tailwind sees the variable name `--color-background` and understands two things:

1. The `--color-` prefix means this is a color token.
2. The remaining name, `background`, becomes the color's name in Tailwind.

That allows Tailwind to generate utilities like:

```css
.bg-background {
  background-color: var(--background);
}

.text-background {
  color: var(--background);
}

.border-background {
  border-color: var(--background);
}
```

In practice, Tailwind usually only emits utilities it finds in your source files. If your app uses this class:

```html
<div class="bg-background text-foreground">
  Content
</div>
```

then Tailwind knows to include the matching generated CSS.

The relationship looks like this:

```text
--color-background
        ↓
Tailwind color token named "background"
        ↓
bg-background utility class
        ↓
background-color: var(--background)
```

## What `inline` changes

The `inline` keyword changes how Tailwind writes the generated utility.

With this CSS:

```css
:root {
  --background: #ffffff;
}

@theme inline {
  --color-background: var(--background);
}
```

Tailwind can generate:

```css
.bg-background {
  background-color: var(--background);
}
```

Without `inline`, the generated CSS would be closer to:

```css
.bg-background {
  background-color: var(--color-background);
}
```

So the difference is:

```text
without inline:
bg-background -> var(--color-background) -> var(--background)

with inline:
bg-background -> var(--background)
```

`inline` removes one layer of variable indirection. It is especially useful when a Tailwind theme token points to another CSS variable.

## Why this is useful for dark mode

A common pattern is to define ordinary CSS variables on `:root`, then change them in dark mode:

```css
:root {
  --background: #ffffff;
  --foreground: #171717;
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
}

@media (prefers-color-scheme: dark) {
  :root {
    --background: #0a0a0a;
    --foreground: #ededed;
  }
}
```

Now components can use stable Tailwind classes:

```html
<body class="bg-background text-foreground">
  ...
</body>
```

The class names stay the same, while the underlying values can change between light and dark mode.

## The main idea

`@theme` is Tailwind's way of turning CSS variables into design tokens that utility classes can use.

In short:

```text
Developer defines a theme variable in @theme
        ↓
Tailwind reads that variable during build
        ↓
Tailwind makes matching utility classes available
        ↓
Components use those utilities in markup
```

So when you see this:

```css
@theme inline {
  --color-background: var(--background);
}
```

read it as:

> Define a Tailwind color token named `background`, and when utilities use it, point directly to the `--background` CSS variable.

That is why a class like `bg-background` can exist even though it was not handwritten as a normal CSS class.
