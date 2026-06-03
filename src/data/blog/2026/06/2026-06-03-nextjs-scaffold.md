---
title: Scaffolding a Next.js App in an Empty Git Repo
description: A practical guide to creating a Next.js project in the current directory and choosing the setup prompts.
pubDatetime: 2026-06-03T00:55:32Z
tags: [nextjs, react, scaffolding, web-development]
---

Starting a Next.js project is usually a one-command job, but the first setup can feel surprisingly noisy. The `create-next-app` CLI asks about TypeScript, linting, Tailwind CSS, the App Router, import aliases, and more. Each choice shapes the project you will work in every day.

This guide walks through a clean scaffold for an empty Git repository and explains the common prompts in plain language.

## Create the App in the Current Directory

If you already have an empty Git repository and do not want `create-next-app` to create another folder, run the command with `.` as the project path:

```bash
npx create-next-app@latest .
```

That tells the CLI to generate the Next.js files in the current directory.

You can also use npm's `create` syntax:

```bash
npm create next-app@latest .
```

After the scaffold finishes, start the development server:

```bash
npm run dev
```

Then open:

```text
http://localhost:3000
```

## A Good Default Setup

For a modern Next.js app, these are sensible starter choices:

| Prompt | Recommended Choice | Why |
| --- | --- | --- |
| TypeScript | Yes | Adds type safety and better editor help. |
| Linter | ESLint | The standard ecosystem choice with strong Next.js support. |
| React Compiler | No | Useful optimization tool, but not needed for most new projects yet. |
| Tailwind CSS | Yes | Fast, practical styling for building UI quickly. |
| `src/` directory | Yes | Keeps application code organized away from root config files. |
| App Router | Yes | The modern, recommended Next.js routing system. |
| Import alias customization | No | Keeps the default `@/*` alias, which is already useful. |

The same setup can be written as a command:

```bash
npx create-next-app@latest . --ts --eslint --tailwind --app --src-dir --import-alias "@/*"
```

If the CLI asks whether to customize the import alias, choosing `No` does not disable aliases. It keeps the default alias:

```tsx
import Button from "@/components/Button";
```

That is much cleaner than long relative paths:

```tsx
import Button from "../../../components/Button";
```

## ESLint or Biome?

Both are valid choices.

ESLint is the safer default for a typical Next.js project. It has the largest plugin ecosystem, broad tutorial support, and Next.js-specific rules through `@next/eslint-plugin-next`.

Biome is newer and faster. It combines linting and formatting, so it can replace a common ESLint plus Prettier setup for many projects. It is a good option when you want simpler configuration and speed, but its ecosystem is smaller.

If you are learning Next.js or expect to follow lots of examples, choose ESLint. If you value a compact all-in-one tool and are comfortable with a newer ecosystem, Biome is worth trying.

## What the App Router Means

The App Router is Next.js's modern file-system router. Routes live in the `app/` directory:

```text
src/app/page.tsx
src/app/about/page.tsx
src/app/layout.tsx
```

Those files map to routes:

```text
src/app/page.tsx        -> /
src/app/about/page.tsx  -> /about
```

The App Router also unlocks newer Next.js patterns such as layouts, loading UI, error UI, React Server Components, and route handlers. For a new app, it is the right default.

## What React Compiler Means

React Compiler is a build-time optimizer for React. It can automatically apply memoization-style optimizations that developers often write manually with APIs like `useMemo`, `useCallback`, or `React.memo`.

That can be powerful, but it is not necessary for a first scaffold. Start without it, get the application working, and enable it later when you have a real performance reason or want to experiment intentionally.

## Final Recommendation

For a new Next.js project inside an empty Git repository, start with:

```bash
npx create-next-app@latest . --ts --eslint --tailwind --app --src-dir --import-alias "@/*"
```

Or answer the prompts like this:

```text
TypeScript: Yes
Linter: ESLint
React Compiler: No
Tailwind CSS: Yes
src/ directory: Yes
App Router: Yes
Customize import alias: No
```

This gives you a clean, modern Next.js foundation without adding experimental complexity on day one.

## Further Reading

- [create-next-app CLI](https://nextjs.org/docs/app/api-reference/cli/create-next-app)
- [Next.js App Router](https://nextjs.org/docs/app)
- [React Compiler introduction](https://react.dev/learn/react-compiler/introduction)
