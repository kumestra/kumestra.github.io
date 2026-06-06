---
title: "CSS Basics and Specifications"
description: "A beginner-friendly guide to what CSS is, who maintains it, and where frontend developers should learn and reference it."
pubDatetime: 2026-06-06T06:52:38Z
tags: [css, frontend, web-standards]
---

CSS stands for Cascading Style Sheets. It is the language browsers use to style structured documents like HTML pages. If HTML describes what is on the page, CSS describes how that content should look and flow.

A simple mental model:

| Layer | Job |
| --- | --- |
| HTML | Content and structure |
| CSS | Presentation and layout |
| JavaScript | Behavior and interaction |

For example, HTML might define a heading and a paragraph:

```html
<h1>Hello CSS</h1>
<p>This page is styled with CSS.</p>
```

CSS can then choose those elements and apply visual rules:

```css
h1 {
  color: blue;
  font-size: 2rem;
}

p {
  line-height: 1.6;
  margin-block-start: 1rem;
}
```

The browser combines the HTML structure with the CSS rules and paints the final result on screen.

## Who created CSS?

CSS was first proposed by Håkon Wium Lie in 1994 while he was working at CERN. Bert Bos soon joined the effort, and together they became central figures in shaping CSS into a web standard.

The first official CSS Recommendation, CSS Level 1, was published by the World Wide Web Consortium in 1996. Since then, CSS has grown from one specification into a family of modules covering selectors, layout, color, fonts, animations, transforms, and many other parts of styling.

## Who maintains CSS now?

CSS is maintained by the W3C Cascading Style Sheets Working Group, usually called the CSS Working Group or CSSWG. It is not maintained by one person.

The CSSWG develops new CSS features, maintains existing CSS Recommendations, handles errata, and coordinates with browser vendors and other web standards groups. The work is public, and much of the technical discussion happens through GitHub issues and editor drafts.

That matters because CSS is a web platform standard. A feature only becomes dependable when the specification, browser implementations, tests, and developer documentation begin to line up.

## The core concepts

Most CSS learning starts with a few ideas that show up everywhere.

## Selectors

Selectors tell the browser what elements a rule applies to.

```css
p {
  color: #333;
}

.title {
  font-weight: 700;
}

#app {
  min-height: 100vh;
}
```

In this example, `p` selects all paragraph elements, `.title` selects elements with `class="title"`, and `#app` selects the element with `id="app"`.

## Declarations

Inside each rule, declarations describe the style changes. A declaration is a property and a value.

```css
button {
  background-color: black;
  color: white;
  padding: 0.75rem 1rem;
}
```

Here, `background-color`, `color`, and `padding` are properties. `black`, `white`, and `0.75rem 1rem` are values.

## The cascade

The cascade is the rule system browsers use when more than one CSS rule tries to style the same thing.

```css
p {
  color: blue;
}

.intro {
  color: red;
}
```

If a paragraph has `class="intro"`, both rules could apply. The browser decides which declaration wins using importance, specificity, source order, and cascade layers. This is why CSS sometimes feels surprising at first: you are not only writing styles, you are participating in a priority system.

## Specificity

Specificity is part of the cascade. It describes how targeted a selector is.

As a rough beginner model:

| Selector type | Example | Specificity |
| --- | --- | --- |
| Element | `p` | Low |
| Class | `.intro` | Medium |
| ID | `#hero` | High |

This is not the whole algorithm, but it is enough to explain many early CSS questions.

## Inheritance

Some CSS properties inherit from parent elements to child elements. Text-related properties often inherit.

```css
body {
  color: #222;
  font-family: system-ui, sans-serif;
}
```

Most text inside the page will inherit that color and font unless another rule overrides it. Not every property inherits: layout properties like `margin`, `padding`, and `border` usually do not.

## The box model

Every element is treated like a box. The box has content, padding, border, and margin.

```text
margin
  border
    padding
      content
```

The content is the actual text, image, or child elements. Padding creates space inside the border. Border surrounds the padding and content. Margin creates space outside the element.

The box model is one of the most important CSS ideas because spacing, sizing, and layout all depend on it.

## Layout

CSS has several layout systems. The most common ones for modern frontend work are normal flow, flexbox, and grid.

Normal flow is the default. Block elements stack vertically, and inline elements flow inside lines of text.

Flexbox is useful for arranging items in one direction:

```css
.toolbar {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}
```

Grid is useful for two-dimensional layout:

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 1rem;
}
```

You do not need to use flexbox or grid for everything. Good CSS often starts with normal flow and adds layout tools only where they clarify the page.

## MDN versus the CSS specification

For most frontend developers, MDN Web Docs is the best daily reference. MDN explains CSS in developer-friendly language, gives practical examples, and includes browser compatibility tables.

The CSS specifications are more formal. They are written for browser implementers, standards authors, tool authors, and people who need exact behavior. Specs answer questions like "what must the browser do in this edge case?"

A good workflow is:

1. Learn the concept from MDN or a practical course.
2. Build small examples in HTML and CSS.
3. Check browser compatibility before using newer features.
4. Read the formal spec when the practical documentation is unclear or when exact behavior matters.

In short: MDN is usually better for learning and day-to-day frontend reference. The specification is the source of truth for how browsers should implement CSS.

## Related URLs

| Resource | Best for | URL |
| --- | --- | --- |
| MDN CSS documentation | Learning and daily frontend reference | <https://developer.mozilla.org/en-US/docs/Web/CSS> |
| MDN CSS reference | Looking up properties, selectors, at-rules, and values | <https://developer.mozilla.org/en-US/docs/Web/CSS/Reference> |
| MDN Getting Started With CSS | Beginner tutorial | <https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Getting_started> |
| web.dev Learn CSS | Structured CSS course | <https://web.dev/learn/css> |
| W3C CSS current work | Overview of completed specs and drafts | <https://www.w3.org/Style/CSS/current-work.en.html> |
| CSSWG editor drafts | Latest in-progress specification drafts | <https://drafts.csswg.org/> |
| W3C CSSWG publications | Official W3C CSS publications by status | <https://www.w3.org/groups/wg/css/publications/> |
| W3C CSS Working Group charter | Scope and maintenance responsibility of the CSSWG | <https://www.w3.org/2025/03/css-wg.html> |

## A practical way to learn CSS

Do not start by memorizing every property. Start with the mental model.

First, learn selectors and declarations. Then learn the cascade, specificity, inheritance, and the box model. After that, practice layout with normal flow, flexbox, and grid. Once those ideas feel familiar, newer features become much easier to place.

CSS is not just decoration. It is the browser's language for turning document structure into visual interface. The better you understand its few core systems, the less mysterious the rest of it becomes.
