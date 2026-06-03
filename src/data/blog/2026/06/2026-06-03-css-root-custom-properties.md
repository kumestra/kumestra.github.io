---
title: Understanding CSS :root Custom Properties
description: A beginner-friendly explanation of how CSS variables are declared on :root and used with var().
pubDatetime: 2026-06-03T03:34:37Z
tags: [css, web-development, frontend]
---

## The Code

```css
:root {
  --background: #ffffff;
  --foreground: #171717;
}
```

This small CSS block defines two reusable values for the page: one for a background color and one for a foreground color. It may look unusual at first if you remember CSS mainly as properties like `color`, `margin`, or `background`, because these lines are not directly styling an element. They are defining CSS custom properties, often called CSS variables.

## What `:root` Means

`:root` is a CSS selector. In an HTML document, it selects the top-level element of the page, which is the `<html>` element.

That means this rule is roughly similar to writing:

```css
html {
  --background: #ffffff;
  --foreground: #171717;
}
```

Developers often use `:root` for global CSS variables because it places the variables at the highest level of the document. Values defined there can be inherited and used throughout the page.

## What The Variables Are

These two declarations create CSS custom properties:

```css
--background: #ffffff;
--foreground: #171717;
```

A CSS custom property name starts with two hyphens, like `--background`. The value after the colon is what the variable stores.

In this example:

| Variable | Value | Meaning |
| --- | --- | --- |
| `--background` | `#ffffff` | White |
| `--foreground` | `#171717` | Very dark gray |

The semicolon at the end finishes each declaration, just like ordinary CSS.

## Does This Change The Page Visually?

Not by itself.

This block selects the root element and stores two values there, but it does not directly set a visual CSS property like `background`, `color`, `border`, or `font-size`.

For example, this defines a variable:

```css
:root {
  --background: #ffffff;
}
```

But this actually uses the variable to style something:

```css
body {
  background: var(--background);
}
```

The `var()` function means: use the value stored in this CSS variable. So `var(--background)` becomes `#ffffff`.

## Why CSS Variables Are Declared Inside Selectors

In many programming languages, variables can be declared directly:

```js
let background = "#ffffff";
```

CSS works differently. CSS is built around selectors and declarations. Styles are attached to elements, and custom properties follow the same model.

So instead of defining a variable in a separate global memory space, CSS defines the variable on an element:

```css
:root {
  --background: #ffffff;
}
```

Because `:root` is the top-level element, this behaves like a page-level default. It is similar to a global variable, but with an important CSS difference: it can be overridden by the cascade.

## Overriding The Variable

A custom property defined at `:root` can be replaced in a smaller part of the page:

```css
:root {
  --background: white;
}

.dark-section {
  --background: black;
}

.card {
  background: var(--background);
}
```

A `.card` outside `.dark-section` would use `white`. A `.card` inside `.dark-section` would use `black`.

That is the main reason CSS variables are powerful: they are reusable like variables, but they still participate in normal CSS inheritance and overriding.

## The Short Version

```css
:root {
  --background: #ffffff;
  --foreground: #171717;
}
```

This code selects the root HTML element and defines two reusable CSS variables. It does not directly change the page's appearance. The page changes only when another rule uses those variables with `var(...)`, such as `background: var(--background)` or `color: var(--foreground)`.
