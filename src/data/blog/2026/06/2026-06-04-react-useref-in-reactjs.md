---
title: "React useRef Explained"
description: "A beginner-friendly guide to useRef in React, including DOM refs, mutable values, TypeScript typing, and when to choose useRef instead of useState."
pubDatetime: 2026-06-04T11:15:32Z
tags: [react, javascript, frontend, hooks]
---

`useRef` is a React Hook that gives a component a value it can remember between renders.

The value lives inside a small object with a `current` property:

```jsx
const myRef = useRef(initialValue);
```

If the initial value is `0`, then the ref starts like this:

```jsx
{
  current: 0
}
```

The important part is that `myRef.current` can change without causing React to re-render the component.

That makes `useRef` different from `useState`.

## `useRef` vs `useState`

Both `useRef` and `useState` let a component remember something, but they are used for different jobs.

| Hook | Best for | Does changing it re-render? |
|---|---|---|
| `useState` | Data that should update the UI | Yes |
| `useRef` | DOM elements or hidden mutable values | No |

For example, state is a good choice when the screen should show the latest value:

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

When `setCount` runs, React renders the component again so the button can show the new count.

A ref is different:

```jsx
import { useRef } from "react";

function CounterTracker() {
  const countRef = useRef(0);

  function increase() {
    countRef.current = countRef.current + 1;
    console.log(countRef.current);
  }

  return <button onClick={increase}>Increase hidden count</button>;
}
```

Here, `countRef.current` changes, but React does not re-render the component. The value is remembered, but it is not part of the visible UI.

## Real case 1: focusing an input

One of the most common uses for `useRef` is accessing a real DOM element.

For example, you might want a button that focuses an input field:

```jsx
import { useRef } from "react";

function SearchBox() {
  const inputRef = useRef(null);

  function focusInput() {
    inputRef.current?.focus();
  }

  return (
    <>
      <input ref={inputRef} placeholder="Search..." />
      <button onClick={focusInput}>Focus input</button>
    </>
  );
}
```

This line creates the ref:

```jsx
const inputRef = useRef(null);
```

At first, `inputRef.current` is `null`.

Then this JSX connects the ref to the input:

```jsx
<input ref={inputRef} />
```

After React creates the real input element in the browser, React stores that DOM element in:

```jsx
inputRef.current
```

So `inputRef.current` becomes the real input element, not the JSX text itself. That is why the code can call:

```jsx
inputRef.current?.focus();
```

The `?.` optional chaining is useful because the ref can be `null` before the element mounts or after it unmounts.

## Real case 2: storing a timer ID

Refs are not only for DOM elements. They can store any value that should survive renders without updating the screen.

A common example is a timer ID:

```jsx
import { useRef } from "react";

function TimerButton() {
  const timeoutRef = useRef(null);

  function startTimer() {
    timeoutRef.current = setTimeout(() => {
      console.log("Timer finished");
    }, 3000);
  }

  function cancelTimer() {
    if (timeoutRef.current !== null) {
      clearTimeout(timeoutRef.current);
      timeoutRef.current = null;
    }
  }

  return (
    <>
      <button onClick={startTimer}>Start</button>
      <button onClick={cancelTimer}>Cancel</button>
    </>
  );
}
```

The timer ID needs to be remembered so it can be canceled later. But showing the timer ID in the UI is not necessary, so `useRef` is a better fit than `useState`.

## Two ways `ref.current` can change

There are two common ways a ref gets a new value.

The first way is changing it manually:

```jsx
const valueRef = useRef(0);

valueRef.current = 10;
```

The second way is letting React set it through JSX:

```jsx
const inputRef = useRef(null);

return <input ref={inputRef} />;
```

In the second example, React sets `inputRef.current` to the real DOM input element after the input appears on the page.

When that element is removed, React sets the ref back to `null`.

## TypeScript and DOM refs

In TypeScript, DOM refs usually include the element type:

```tsx
import { useRef } from "react";

function SearchBox() {
  const inputRef = useRef<HTMLInputElement>(null);

  function focusInput() {
    inputRef.current?.focus();
  }

  return <input ref={inputRef} />;
}
```

This part tells TypeScript what kind of element the ref will hold:

```tsx
useRef<HTMLInputElement>(null)
```

It means:

```text
This ref starts as null, but it will point to an HTML input element later.
```

Because TypeScript knows the ref points to an `HTMLInputElement`, it understands methods like:

```tsx
inputRef.current?.focus();
inputRef.current?.select();
inputRef.current?.value;
```

You may also see this more explicit form:

```tsx
const inputRef = useRef<HTMLInputElement | null>(null);
```

For many React DOM refs, both forms are acceptable. The first style is common because React's TypeScript types understand that a DOM ref can start as `null`.

## What happens during render?

Consider this component:

```tsx
import { useRef } from "react";

function MyComponent() {
  const inputRef = useRef<HTMLInputElement>(null);

  console.log(inputRef.current);

  return <input ref={inputRef} />;
}
```

During render, `inputRef.current` is usually `null` because React has not attached the ref to the DOM element yet.

After React commits the input to the page, the ref points to the actual input element.

That is why ref-based DOM work is usually done in event handlers or effects:

```tsx
function focusInput() {
  inputRef.current?.focus();
}
```

## A simple rule for choosing

Use `useState` when changing a value should update what the user sees.

Use `useRef` when you need to remember a value but changing it should not update the UI.

Common `useRef` examples include:

- Focusing an input
- Reading the size or position of a DOM element
- Storing timer IDs
- Storing previous values
- Keeping a mutable flag that should survive re-renders

The shortest mental model is:

```text
useState changes the UI.
useRef remembers something quietly.
```
