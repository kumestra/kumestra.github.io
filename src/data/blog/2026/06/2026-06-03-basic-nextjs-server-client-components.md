---
title: "Basic Next.js Concepts: Server Components and Client Components"
description: A beginner-friendly explanation of what Next.js adds to React and how Server Components differ from Client Components.
pubDatetime: 2026-06-03T12:01:33Z
tags: [nextjs, react, server-components, client-components]
---

Next.js is a React framework for building full-stack web applications. React gives you components for describing user interfaces. Next.js adds the surrounding application structure: routing, server rendering, data fetching, optimization, and backend endpoints.

The easiest way to start thinking about Next.js is this:

```txt
React  = UI components
Next.js = React + routing + server rendering + data fetching + backend features
```

Modern Next.js apps usually use the **App Router**, where routes are defined by folders and special files inside the `app` directory.

## File-based routing

In a regular React app, you often install and configure a routing library. In Next.js, routes come from your file structure.

For example:

```txt
app/page.jsx             -> /
app/about/page.jsx       -> /about
app/blog/page.jsx        -> /blog
app/blog/[slug]/page.jsx -> /blog/my-first-post
```

A `page.jsx` file creates UI for a route. A `layout.jsx` file creates shared UI that wraps pages, such as a navigation bar, sidebar, footer, or app shell.

```jsx
// app/layout.jsx
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <nav>My Site</nav>
        <main>{children}</main>
      </body>
    </html>
  )
}
```

```jsx
// app/page.jsx
export default function Page() {
  return <h1>Hello Next.js</h1>
}
```

That small structure already gives you a working route at `/` with a shared layout.

## The key idea: server and client components

In the App Router, React components can run in two different places:

| Component type | Runs where? | Best for |
| --- | --- | --- |
| Server Component | Server | Fetching data, reading files, querying databases, rendering mostly static UI |
| Client Component | Browser | Click handlers, state, effects, browser APIs, interactive widgets |

By default, components in the `app` directory are **Server Components**. If a component needs browser interactivity, you mark it as a **Client Component** with the `"use client"` directive.

## Server Components

A Server Component runs on the server. A useful first mental model is: **a Server Component is a little like PHP-style server-rendered UI, but written as React and JavaScript**.

That means a Server Component can fetch data, call server-side functions, read from a database, and use secrets that should never be sent to the browser.

```jsx
// app/blog/page.jsx
import { getPosts } from "@/lib/posts"

export default async function BlogPage() {
  const posts = await getPosts()

  return (
    <main>
      <h1>Blog</h1>
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </main>
  )
}
```

This component can be `async` because it runs on the server. The browser receives the rendered result, not the database code.

Server Components are good for:

- Loading data before rendering UI
- Keeping sensitive code and credentials on the server
- Reducing the amount of JavaScript sent to the browser
- Rendering pages, layouts, lists, articles, dashboards, and other mostly non-interactive UI

Server Components cannot use browser-only behavior:

- No `useState`
- No `useEffect`
- No `onClick` handlers
- No `window`
- No `localStorage`

If the component needs those things, it should be a Client Component.

## Client Components

A Client Component runs in the browser. This is closest to the React code many people already know from standard client-side React apps.

To create one, put `"use client"` at the very top of the file:

```jsx
// app/components/Counter.jsx
"use client"

import { useState } from "react"

export default function Counter() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  )
}
```

Client Components are good for:

- Buttons and click handlers
- Forms with live state
- Tabs, menus, modals, and accordions
- Browser APIs such as `window`, `document`, or `localStorage`
- React hooks such as `useState`, `useEffect`, and `useRef`

A Client Component is still part of a Next.js app. It can use Next.js features such as navigation helpers and optimized routing. The main difference is that its JavaScript is sent to the browser so it can become interactive.

## Using both together

The usual pattern is to keep pages and layouts as Server Components, then move only the interactive pieces into smaller Client Components.

```jsx
// app/page.jsx
import Counter from "./components/Counter"

export default async function Page() {
  const message = "Rendered on the server"

  return (
    <main>
      <h1>{message}</h1>
      <Counter />
    </main>
  )
}
```

In this example, `Page` is a Server Component. It renders the page structure and can fetch server-side data. `Counter` is a Client Component because it needs state and an `onClick` handler.

That gives you a clean split:

```txt
Server Component: prepare data and render UI on the server
Client Component: handle interaction in the browser
```

## A practical rule of thumb

Start with a Server Component. Switch to a Client Component only when the component needs browser interactivity.

Use a Server Component when you need to:

- Fetch data
- Read from a database
- Access backend-only code
- Render content without local browser state

Use a Client Component when you need to:

- Track changing UI state
- Respond to clicks
- Run effects after rendering
- Access browser-only APIs

This keeps your app fast because less JavaScript needs to be downloaded and executed in the browser.

## Final mental model

Next.js is React with a server-aware application framework around it.

Server Components are like server-rendered React views. They are great for data and structure.

Client Components are like normal interactive React components. They are great for browser behavior.

The power of Next.js comes from combining both: render as much as possible on the server, then use client-side React exactly where interactivity is needed.
