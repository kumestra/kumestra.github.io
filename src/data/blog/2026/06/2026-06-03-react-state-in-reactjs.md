---
title: React State Explained
description: A beginner-friendly guide to component state, useState, setter functions, and functional updates in React.
pubDatetime: 2026-06-03T09:27:10Z
tags: [react, javascript, frontend, state]
---

State is one of the most important ideas in React. It is the data a component owns and remembers while the app is running.

If props are data passed into a component, state is data managed inside a component.

For example, state can store:

- The number in a counter
- The text typed into an input
- Whether a menu is open or closed
- The currently selected tab
- A loading status
- A list of items fetched from an API

When state changes, React re-renders the component so the screen can show the new value.

## Creating state with `useState`

In a function component, state is commonly created with the `useState` hook:

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

This line is the key:

```jsx
const [count, setCount] = useState(0);
```

It means:

| Part | Meaning |
|---|---|
| `count` | The current state value |
| `setCount` | The function used to update the state |
| `0` | The initial value used on the first render |

The variable name and setter name are up to you, but React code usually follows this pattern:

```jsx
const [value, setValue] = useState(initialValue);
```

## Updating state

You should not change state directly:

```jsx
// Wrong
count = count + 1;
```

Instead, call the setter function:

```jsx
// Right
setCount(count + 1);
```

The argument passed to the setter becomes the next state value.

```jsx
setCount(5);
```

After React applies that update, `count` becomes `5`, and the component renders again with the new value.

## State can hold different types

State is not limited to numbers. It can hold strings, booleans, arrays, objects, or other values.

```jsx
const [name, setName] = useState("");
const [isOpen, setIsOpen] = useState(false);
const [items, setItems] = useState([]);
const [user, setUser] = useState({ name: "Alex", age: 25 });
```

Choose the initial value based on what kind of data the component needs to remember.

## Example: input state

Here is a simple example where state stores the text typed by the user:

```jsx
import { useState } from "react";

function NameForm() {
  const [name, setName] = useState("");

  return (
    <>
      <input
        value={name}
        onChange={(event) => setName(event.target.value)}
      />

      <p>Hello, {name}</p>
    </>
  );
}
```

Every time the user types, the `onChange` event runs. The new input value is passed to `setName`, React updates the `name` state, and the paragraph displays the latest text.

This pattern is common in React forms:

```text
User types -> event handler runs -> setter updates state -> component re-renders
```

## When to use a functional update

Sometimes the new state depends on the previous state.

This works in many simple cases:

```jsx
setCount(count + 1);
```

But React can batch state updates, which means multiple updates may be grouped together for performance. When the next value depends on the previous value, a safer pattern is:

```jsx
setCount((previousCount) => previousCount + 1);
```

The setter receives a function. React passes the latest previous state into that function, and the return value becomes the next state.

This is especially useful when updating state more than once in a row:

```jsx
setCount((previousCount) => previousCount + 1);
setCount((previousCount) => previousCount + 1);
setCount((previousCount) => previousCount + 1);
```

Each update is based on the latest value, so the count increases by `3`.

## Updating object state

When state is an object, avoid mutating it directly:

```jsx
// Wrong
user.name = "Sam";
setUser(user);
```

Instead, create a new object:

```jsx
setUser({
  ...user,
  name: "Sam",
});
```

The spread syntax copies the existing fields, then replaces the `name` field. React works best when state is treated as immutable data: create a new value instead of changing the old one.

## Updating array state

The same idea applies to arrays.

Instead of mutating the array:

```jsx
// Wrong
items.push("New item");
setItems(items);
```

Create a new array:

```jsx
setItems([...items, "New item"]);
```

For removing an item:

```jsx
setItems(items.filter((item) => item.id !== idToRemove));
```

For changing an item:

```jsx
setItems(
  items.map((item) =>
    item.id === idToUpdate
      ? { ...item, title: "Updated title" }
      : item
  )
);
```

## Props vs state

Props and state both affect what appears on the screen, but they are not the same.

| Concept | Meaning |
|---|---|
| Props | Data received from a parent component |
| State | Data owned and updated by the component |

A component should use props when another component gives it data. It should use state when it needs to remember and change data itself.

```jsx
function Greeting({ name }) {
  return <h1>Hello, {name}</h1>;
}
```

In this example, `name` is a prop. The `Greeting` component does not own it.

```jsx
function Toggle() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <button onClick={() => setIsOpen((open) => !open)}>
      {isOpen ? "Open" : "Closed"}
    </button>
  );
}
```

In this example, `isOpen` is state. The `Toggle` component owns and updates it.

## A simple mental model

Think of a React component as a function that describes the UI for the current data.

```text
state + props -> UI
```

When state changes, React calls the component again and produces updated UI.

You do not manually edit the page. You update state, and React makes the interface match that state.

## Summary

State is data owned by a component. In React, the most common way to create state is:

```jsx
const [state, setState] = useState(initialValue);
```

To change the value, call the setter:

```jsx
setState(newValue);
```

If the next value depends on the previous value, use a functional update:

```jsx
setState((previousState) => nextState);
```

The main ideas are:

- State lets components remember changing data.
- State should be updated with its setter function.
- Changing state causes React to re-render the component.
- Object and array state should be updated by creating new values.
- Props are passed into a component; state is owned by a component.

Once this clicks, React becomes much easier to understand: your UI is just a reflection of your current state.
