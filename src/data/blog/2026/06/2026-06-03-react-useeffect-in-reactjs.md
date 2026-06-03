---
title: React useEffect Explained
description: A beginner-friendly guide to running side effects in React with useEffect and dependency arrays.
pubDatetime: 2026-06-03T09:37:07Z
tags: [react, javascript, frontend, hooks]
---

`useEffect` is a React Hook for running code after a component renders.

That code is usually called a side effect. A side effect is something that happens outside the normal job of rendering JSX to the screen.

Common side effects include:

- Fetching data from an API
- Updating the page title
- Starting or stopping a timer
- Subscribing to browser events
- Reading from or writing to local storage
- Connecting to some external system

The basic idea is simple:

```jsx
useEffect(() => {
  // code to run after render
}, [dependencies]);
```

`useEffect` has two main arguments:

| Argument | Meaning |
|---|---|
| First argument | A function React runs after rendering |
| Second argument | An array of values React watches |

The second argument is usually called the dependency array.

## The first argument: the effect function

The first argument is a function.

```jsx
useEffect(() => {
  console.log("The effect ran");
});
```

React runs this function after the component renders. This is different from normal code inside the component body, which runs while React is calculating what the UI should look like.

An effect is useful when the component needs to synchronize with something outside React.

For example, this effect changes the browser tab title:

```jsx
import { useEffect } from "react";

function PageTitle() {
  useEffect(() => {
    document.title = "Hello React";
  });

  return <h1>Hello React</h1>;
}
```

The JSX controls what appears inside the page. The effect controls something outside the JSX: the browser tab title.

## The second argument: the dependency array

The second argument tells React when the effect should run again.

```jsx
useEffect(() => {
  console.log("count changed");
}, [count]);
```

This means:

1. Run the effect after the component first appears.
2. Run the effect again when `count` changes.

If the array contains more than one value, the effect runs again when any one of them changes.

```jsx
useEffect(() => {
  console.log("count or name changed");
}, [count, name]);
```

This effect runs:

- After the first render
- Again when `count` changes
- Again when `name` changes

React does not wait for every dependency to change. One changed dependency is enough.

## Three common dependency patterns

The dependency array changes how often the effect runs.

| Code | When it runs |
|---|---|
| `useEffect(() => {})` | After every render |
| `useEffect(() => {}, [])` | After the first render only |
| `useEffect(() => {}, [value])` | After the first render and whenever `value` changes |

These three patterns are very important.

## No dependency array

If you do not pass a dependency array, the effect runs after every render.

```jsx
useEffect(() => {
  console.log("Runs after every render");
});
```

This can be useful sometimes, but it is not the most common beginner use case. If the effect updates state every time it runs, it can accidentally create a render loop.

```jsx
useEffect(() => {
  setCount(count + 1);
});
```

This is dangerous because:

```text
Effect runs -> state changes -> component renders -> effect runs again
```

When an effect changes state, think carefully about the dependency array.

## Empty dependency array

An empty dependency array means the effect runs after the component first appears.

```jsx
useEffect(() => {
  console.log("Component appeared");
}, []);
```

This is commonly used for setup work, such as fetching initial data.

```jsx
import { useEffect, useState } from "react";

function UsersList() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    async function loadUsers() {
      const response = await fetch("/api/users");
      const data = await response.json();
      setUsers(data);
    }

    loadUsers();
  }, []);

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

The component first renders with an empty `users` array. Then the effect runs, fetches the data, updates state, and React renders again with the loaded users.

## Dependency array with values

If the effect depends on a value, put that value in the dependency array.

```jsx
import { useEffect, useState } from "react";

function SearchResults({ searchTerm }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    async function search() {
      const response = await fetch(`/api/search?q=${searchTerm}`);
      const data = await response.json();
      setResults(data);
    }

    search();
  }, [searchTerm]);

  return (
    <ul>
      {results.map((result) => (
        <li key={result.id}>{result.title}</li>
      ))}
    </ul>
  );
}
```

Here, the effect should run when `searchTerm` changes because the API request depends on `searchTerm`.

The flow looks like this:

```text
searchTerm changes -> component renders -> effect runs -> data loads -> state updates
```

The dependency array keeps the effect connected to the values it uses.

## Cleanup functions

Some effects need cleanup.

For example, if an effect starts a timer, the component should stop the timer when it no longer needs it.

```jsx
import { useEffect, useState } from "react";

function Clock() {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const timerId = setInterval(() => {
      setTime(new Date());
    }, 1000);

    return () => {
      clearInterval(timerId);
    };
  }, []);

  return <p>{time.toLocaleTimeString()}</p>;
}
```

The returned function is the cleanup function.

```jsx
return () => {
  clearInterval(timerId);
};
```

React runs cleanup when the component is removed. React also runs cleanup before running the effect again if the dependencies changed.

That means this pattern:

```jsx
useEffect(() => {
  // setup

  return () => {
    // cleanup
  };
}, [value]);
```

means:

```text
value changes -> old cleanup runs -> new setup runs
```

Cleanup is important for timers, subscriptions, event listeners, and connections.

## Example: window event listener

Here is a common cleanup example with a browser event listener:

```jsx
import { useEffect, useState } from "react";

function WindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    function handleResize() {
      setWidth(window.innerWidth);
    }

    window.addEventListener("resize", handleResize);

    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  return <p>Window width: {width}</p>;
}
```

The effect adds the event listener. The cleanup removes it.

Without cleanup, the browser could keep old event listeners after the component is gone.

## Do you always need useEffect?

No. This is one of the most important things to learn.

Use `useEffect` when the component needs to synchronize with something outside rendering. Do not use it just because you want to calculate a value.

For example, this does not need an effect:

```jsx
function FullName({ firstName, lastName }) {
  const fullName = `${firstName} ${lastName}`;

  return <p>{fullName}</p>;
}
```

This value can be calculated during render, so `useEffect` is unnecessary.

Also, user actions usually belong in event handlers:

```jsx
function SaveButton() {
  function handleClick() {
    console.log("Save the form");
  }

  return <button onClick={handleClick}>Save</button>;
}
```

The click action is caused by the user clicking the button, so it belongs in the click handler.

## A useful mental model

A helpful way to think about `useEffect` is:

```text
Render the UI first.
Then synchronize with the outside world.
```

The dependency array tells React which values the synchronization depends on.

If the effect uses `count`, include `count`.

```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);
```

If the effect uses `userId`, include `userId`.

```jsx
useEffect(() => {
  fetch(`/api/users/${userId}`);
}, [userId]);
```

This keeps the effect honest: when the values used by the effect change, React runs the effect again.

## About Strict Mode in development

In React development mode, Strict Mode may run an effect setup and cleanup an extra time. This helps reveal bugs in effects that do not clean up correctly.

If you see an effect run twice while developing, it does not always mean your production app will do the same thing. It may be React checking that the effect is written safely.

The best response is usually not to fight Strict Mode. Instead, make sure the effect can clean up after itself.

## Summary

`useEffect` runs code after React renders a component.

The first argument is the function to run:

```jsx
useEffect(() => {
  // effect code
});
```

The second argument is the dependency array:

```jsx
useEffect(() => {
  // effect code
}, [value]);
```

The effect runs after the component first appears. Then it runs again when one or more dependencies in the array change.

Use `useEffect` for side effects like fetching data, timers, subscriptions, browser APIs, and other external systems. For values that can be calculated during render, keep the calculation in the component body instead.
