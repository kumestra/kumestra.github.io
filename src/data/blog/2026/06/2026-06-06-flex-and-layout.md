---
title: "Flex and Layout in CSS"
description: "A practical introduction to how browsers lay out elements by default, what flexbox changes, and when to use flex layout."
pubDatetime: 2026-06-06T04:06:50Z
tags: [css, flexbox, layout, tailwind]
---

When a browser renders HTML, every element participates in a layout system. Before you write any CSS, the browser already knows how common elements should behave. A `div` is block-level by default, so it usually takes the full available width and stacks with other block elements vertically. A `span` is inline by default, so it flows inside a line of text.

That default behavior is useful, but it is only one layout mode. CSS gives you several ways to tell an element how it should lay out itself and, in some cases, how it should arrange its direct children.

Flexbox is one of those layout modes.

## The default layout

Consider this markup:

```html
<div class="container">
  <div>One</div>
  <div>Two</div>
  <div>Three</div>
</div>
```

If you do not add layout CSS, the inner `div` elements use normal block flow. They stack from top to bottom:

```text
One
Two
Three
```

This is often called normal flow. It is the browser's default way of placing elements on the page. Block elements stack vertically. Inline elements flow horizontally inside text. Margins, width, height, and content all affect how much space each element takes.

Normal flow is not a mistake or a fallback. It is the baseline layout system of the web.

## Changing the layout mode

The CSS `display` property controls the display behavior of an element. Common values include:

```css
display: block;
display: inline;
display: inline-block;
display: flex;
display: grid;
```

When you write:

```css
.container {
  display: flex;
}
```

you are saying: make `.container` a flex container, and lay out its direct children as flex items.

The important phrase is direct children. Flexbox affects the immediate children of the element with `display: flex`. It does not automatically reach into every nested element below them.

```html
<div class="container">
  <section>Direct child</section>
  <section>
    Direct child
    <p>Nested child</p>
  </section>
</div>
```

In this example, the two `section` elements become flex items. The `p` element is not a flex item of `.container`; it is still laid out by whatever layout rules apply inside its parent `section`.

## What flex changes

By default, a flex container lays out its direct children in a row:

```text
One  Two  Three
```

That is already different from normal block flow, where the same `div` children would stack vertically.

Once an element is a flex container, flexbox properties become meaningful:

```css
.container {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  gap: 1rem;
}
```

These properties control the direct children:

| Property | What it controls |
| --- | --- |
| `flex-direction` | Whether items flow in a row, column, reversed row, or reversed column |
| `justify-content` | How items are distributed along the main axis |
| `align-items` | How items align along the cross axis |
| `gap` | Space between flex items |
| `flex-wrap` | Whether items stay on one line or wrap onto new lines |

The main axis depends on `flex-direction`. With the default `flex-direction: row`, the main axis is horizontal. With `flex-direction: column`, the main axis is vertical.

That means `justify-content` is not always "horizontal alignment" and `align-items` is not always "vertical alignment." They depend on the flex direction.

## Row and column

The default flex direction is row:

```css
.container {
  display: flex;
  flex-direction: row;
}
```

This places children side by side:

```text
One  Two  Three
```

If you want children to stack vertically but still use flexbox alignment tools, use column:

```css
.container {
  display: flex;
  flex-direction: column;
}
```

Now the children stack:

```text
One
Two
Three
```

This can look similar to normal block flow, but it is not the same. The parent is still a flex container, so flexbox controls like `gap`, `align-items`, and `justify-content` apply.

## Flex in Tailwind CSS

Tailwind's `flex` class is a shortcut for:

```css
display: flex;
```

So this:

```html
<div class="flex items-start justify-center">
  ...
</div>
```

means roughly:

```css
div {
  display: flex;
  align-items: flex-start;
  justify-content: center;
}
```

With the default row direction, `justify-center` centers the direct children along the horizontal main axis, while `items-start` aligns them at the start of the vertical cross axis.

If the container also has a minimum viewport height, the effect becomes easier to see:

```html
<main class="flex min-h-dvh items-start justify-center">
  <article>Content</article>
</main>
```

Here, the `main` element becomes a flex container. Its direct child is centered horizontally, aligned to the top vertically, and the container is at least as tall as the dynamic viewport.

## Flex changes the parent-child relationship

A useful mental model is:

> The parent chooses the layout system for its direct children.

If the parent uses normal block flow, block children stack.

If the parent uses `display: flex`, direct children become flex items.

If the parent uses `display: grid`, direct children become grid items.

So when you ask, "Do I need to make the container flex to change the layout of its children?" the precise answer is:

You need `display: flex` if you want to use flexbox to arrange that container's direct children.

But flexbox is not the only way to change layout. You can also use grid, normal flow, inline layout, positioning, margins, and other CSS tools. Flex is one strong option, especially when you want one-dimensional alignment.

## Flexbox versus grid

Flexbox is usually best for one-dimensional layout: a row or a column.

Grid is usually best for two-dimensional layout: rows and columns at the same time.

For example, use flexbox for:

- A navigation bar
- A row of buttons
- Centering one child in a parent
- A toolbar with icons and labels
- A vertical stack with consistent spacing

Use grid for:

- A photo gallery
- A dashboard layout
- A table-like card arrangement
- A page region with columns and rows

This is not a hard rule, but it is a good starting point.

## The key idea

CSS layout is not one thing. The browser starts with default layout behavior, and then CSS lets you opt into different layout modes.

`display: flex` does not simply "turn on alignment." It changes an element into a flex container. That container then lays out its direct children as flex items, making flexbox properties like `justify-content`, `align-items`, `gap`, and `flex-direction` available.

Once that clicks, Tailwind classes such as `flex`, `items-start`, `justify-center`, and `flex-col` become easier to read. They are not magic words; they are compact names for CSS layout decisions.

The parent chooses the layout mode. The direct children live inside that choice.
