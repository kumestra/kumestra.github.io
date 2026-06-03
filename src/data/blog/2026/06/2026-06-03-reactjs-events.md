---
title: ReactJS Events for Beginners
description: A beginner-friendly guide to understanding how events work in ReactJS, how they differ from HTML and JavaScript events, and how they commonly update state.
pubDatetime: 2026-06-03T10:15:04Z
tags: [react, javascript, frontend]
---

## ReactJS Events for Beginners

Events are how a React app responds to user actions. When someone clicks a button, types in an input, submits a form, presses a key, or moves the mouse, React can run a function in response.

A simple way to think about it is:

```text
user action -> event handler runs -> state may change -> React updates the UI
```

## Basic Event Syntax

In React, events are written on JSX elements like this:

```jsx
<htmlTag eventName={functionName}>
```

For example:

```jsx
function App() {
  function handleClick() {
    alert("Button clicked");
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

Here, `onClick` is the event name, and `handleClick` is the function that runs when the button is clicked.

## React Uses camelCase Event Names

In normal HTML, event names are lowercase:

```html
<button onclick="alert('Clicked')">Click me</button>
```

In React, event names use camelCase:

```jsx
<button onClick={handleClick}>Click me</button>
```

Common React events include:

| Event | When it runs |
| --- | --- |
| `onClick` | When an element is clicked |
| `onChange` | When an input value changes |
| `onSubmit` | When a form is submitted |
| `onMouseEnter` | When the mouse enters an element |
| `onKeyDown` | When a keyboard key is pressed |

## Pass a Function, Do Not Call It Immediately

This is correct:

```jsx
<button onClick={handleClick}>Click</button>
```

This is usually wrong:

```jsx
<button onClick={handleClick()}>Click</button>
```

The difference matters. `handleClick` passes the function to React so React can call it later. `handleClick()` calls the function immediately while the component is rendering.

If you need to pass a value to the function, use an arrow function:

```jsx
<button onClick={() => handleClick("hello")}>Click</button>
```

## Events Often Change State

In many React components, event handlers update state. When state changes, React re-renders the component and updates the screen.

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

The flow is:

```text
button clicked -> handleClick runs -> setCount updates state -> React shows new count
```

Event handlers do not always change state, but state updates are one of their most common jobs.

They can also log information, call an API, validate a form, focus an input, or trigger another action.

## Using the Event Object

React passes an event object to event handler functions. This object contains information about what happened.

For example, with an input field:

```jsx
import { useState } from "react";

function NameInput() {
  const [name, setName] = useState("");

  function handleChange(event) {
    setName(event.target.value);
  }

  return (
    <>
      <input value={name} onChange={handleChange} />
      <p>Hello, {name}</p>
    </>
  );
}
```

Here, `event.target.value` gives the current value typed into the input.

## Handling Form Submit Events

Forms need special care because browsers normally reload the page when a form is submitted. In React, you usually stop that default behavior with `preventDefault()`.

```jsx
function SignupForm() {
  function handleSubmit(event) {
    event.preventDefault();
    console.log("Form submitted");
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" placeholder="Email" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

Without `event.preventDefault()`, the browser may refresh the page and reset your React app state.

## HTML Events vs JavaScript Events vs React Events

| Feature | HTML Event | JavaScript Event | React Event |
| --- | --- | --- | --- |
| Example | `onclick="..."` | `addEventListener("click", fn)` | `onClick={fn}` |
| Where it is written | Inside HTML | In JavaScript | Inside JSX |
| Naming style | lowercase | lowercase string | camelCase |
| Function format | String code | Function reference | Function reference |
| DOM selection | Not needed | Usually needed | Not needed |
| UI update style | Manual | Manual | Usually through state |

In plain JavaScript, you often select an element and change it directly:

```js
const title = document.getElementById("title");
title.textContent = "Clicked";
```

In React, you usually update state and let React update the UI:

```jsx
const [title, setTitle] = useState("Hello");

<button onClick={() => setTitle("Clicked")}>Click</button>;
```

This is one of the biggest mindset shifts in React. You do not usually control the DOM directly. You describe what the UI should look like for the current state, and React handles the update.

## Final Mental Model

React events are functions attached to JSX elements. They respond to user actions and often update state.

The most useful beginner pattern is:

```jsx
function handleEvent() {
  setSomething(newValue);
}

return <button onClick={handleEvent}>Click</button>;
```

Once that idea feels natural, React events become much easier to understand. A user does something, your handler runs, state changes, and React refreshes the UI to match the new state.
