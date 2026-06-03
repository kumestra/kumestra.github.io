---
title: Debugging React JSX in Chrome
description: A beginner-friendly guide to how JSX becomes browser JavaScript, how React updates the DOM, and how Chrome DevTools maps bugs back to your original code.
pubDatetime: 2026-06-03T11:20:22Z
tags: [react, javascript, frontend, debugging]
---

React can feel indirect when you first learn it.

You write JSX:

```jsx
function App() {
  return <h1 className="title">Hello React</h1>;
}
```

But Chrome does not understand JSX directly. Chrome understands HTML, CSS, and JavaScript. So before React code runs in the browser, a build tool transforms your JSX into plain JavaScript.

That is the key idea:

```text
JSX source code -> build tool -> JavaScript bundle -> Chrome runs it
```

This is similar to how C++ source code becomes machine code. Developers write one form, but the computer runs another form. The important difference is that frontend tools usually provide source maps, so Chrome can show your original JSX file while debugging.

## JSX Does Not Become JSON

JSX looks like HTML inside JavaScript, but it is not HTML and it is not JSON.

This JSX:

```jsx
const element = <button onClick={handleClick}>Save</button>;
```

is compiled into JavaScript similar to this:

```js
const element = jsx("button", {
  onClick: handleClick,
  children: "Save",
});
```

Older examples may show this form:

```js
const element = React.createElement(
  "button",
  { onClick: handleClick },
  "Save"
);
```

Both forms create a React element: a JavaScript object that describes what should appear on the screen.

It may look a little like JSON:

```js
{
  type: "button",
  props: {
    onClick: handleClick,
    children: "Save"
  }
}
```

But it is not real JSON because it can contain JavaScript values such as functions. For example, `handleClick` is a function, and JSON cannot store functions.

## What the Build Step Does

In a React project, the build tool may be Vite, Webpack, Parcel, Next.js, Remix, or another tool. The exact tool can differ, but the job is usually similar.

A build tool commonly does four things:

| Step | What happens |
| --- | --- |
| Transform | JSX, TypeScript, and newer JavaScript syntax become browser-ready JavaScript |
| Bundle | Many files are combined into fewer files the browser can load |
| Optimize | Production builds may minify code, remove unused code, and split code into chunks |
| Map | Source maps may be created so generated code can point back to original files |

In development, the generated JavaScript is usually easier to debug and source maps are commonly enabled.

In production, the generated JavaScript is often smaller and harder to read:

```js
function a(){return r.jsx("h1",{children:"Hello React"})}
```

This is good for performance, but bad for human eyes. Source maps are what make that compiled code traceable.

## What Chrome Actually Runs

Chrome runs JavaScript. It does not run JSX.

When Chrome loads your React app, it downloads JavaScript files from the dev server or from your production server. Those files include:

- Your compiled component code
- React library code
- React DOM code
- Other libraries your app imports

When your component runs, it returns React elements. React then uses those elements to decide what the real DOM should look like.

The flow is:

```text
component function runs
  -> returns React elements
  -> React compares old tree and new tree
  -> React updates the browser DOM
  -> Chrome paints the page
```

For example:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

When the button is clicked:

```text
click event -> setCount runs -> component renders again -> React updates DOM text
```

React does not usually create CSS by itself. CSS can come from normal CSS files, CSS modules, Tailwind classes, inline styles, styled-components, or another styling system. React mostly connects styles to elements through `className`, `style`, and component logic.

## Why Chrome Can Show Your JSX File

If Chrome is running compiled JavaScript, how can DevTools show `App.jsx`?

The answer is source maps.

A source map is a file that connects generated code back to original code.

It tells Chrome something like:

```text
generated bundle line 50231, column 18
came from
src/App.jsx line 12, column 4
```

So when an error happens in a bundle, Chrome DevTools can show the original file and line number.

Without source maps, an error might look like this:

```text
TypeError: Cannot read properties of undefined
    at a (main.8f31c9.js:1:42108)
```

With source maps, it can look like this:

```text
TypeError: Cannot read properties of undefined
    at ProductCard (src/ProductCard.jsx:18:12)
```

That second version is much more useful. It points you back to the component you wrote.

## Debugging React in Chrome

Chrome DevTools is the main debugging tool for frontend developers.

The most useful areas are:

| Tool | What it helps with |
| --- | --- |
| Console | Read errors, warnings, logs, and stack traces |
| Sources | Set breakpoints in original JSX or JavaScript files |
| Network | Check API calls, image loading, CSS loading, and failed requests |
| Elements | Inspect the real DOM and applied CSS |
| React Developer Tools | Inspect React components, props, state, hooks, and renders |

Chrome DevTools shows the browser side. React Developer Tools shows the React side.

That distinction is important:

```text
Elements tab -> real DOM
React DevTools -> React component tree
```

The DOM tells you what Chrome currently has on the page. The React tree tells you which component produced it and what props and state were involved.

## A Practical Debugging Flow

When a bug happens, a useful flow is:

1. Reproduce the bug clearly.
2. Open Chrome DevTools.
3. Read the Console error and stack trace.
4. Look for the first line that points to your own source file.
5. Open that file in the Sources tab.
6. Set a breakpoint near the suspicious line.
7. Reproduce the bug again.
8. Inspect variables, props, state, and function arguments.
9. Use React Developer Tools to check component props and hooks.
10. Use the Network tab if the bug depends on data from an API.

Do not stop at the first React internal line in the stack trace. Many errors pass through React internals. Usually you want the first stack frame that points to your app code.

For example:

```text
TypeError: Cannot read properties of undefined
    at ProductCard (src/ProductCard.jsx:18:12)
    at renderWithHooks (react-dom...)
    at updateFunctionComponent (react-dom...)
```

The useful line is usually:

```text
ProductCard (src/ProductCard.jsx:18:12)
```

That is where your investigation should begin.

## Common React Bugs and How They Appear

Many React bugs are not Chrome bugs. They are app state, props, rendering, or event bugs.

### Reading a Missing Value

```jsx
function ProductCard({ product }) {
  return <h2>{product.name}</h2>;
}
```

If `product` is `undefined`, Chrome may show:

```text
Cannot read properties of undefined
```

The fix is to understand why the value is missing. Maybe data is still loading, maybe a parent component passed the wrong prop, or maybe the API response shape changed.

### Calling an Event Handler Too Early

This is usually wrong:

```jsx
<button onClick={handleSave()}>Save</button>
```

It calls `handleSave` while rendering.

This is usually correct:

```jsx
<button onClick={handleSave}>Save</button>
```

If you need to pass an argument:

```jsx
<button onClick={() => handleSave(id)}>Save</button>
```

### Updating State from Old State

This can be fragile when the next value depends on the previous value:

```jsx
setCount(count + 1);
```

The safer form is:

```jsx
setCount((previousCount) => previousCount + 1);
```

This is especially useful when multiple state updates happen close together.

### Looking at the DOM Instead of State

In plain JavaScript, you might debug by checking the DOM and changing it directly.

In React, the DOM is the result. State and props are usually the cause.

So ask:

```text
What state value caused this UI?
What props did this component receive?
Which event changed the state?
Did the API return the data I expected?
```

That mindset makes React debugging easier.

## When DevTools Shows Only the Bundle

Sometimes Chrome shows compiled bundle code instead of your original JSX.

Common causes include:

- Source maps are disabled in Chrome DevTools.
- The build tool is not generating source maps.
- The app is running a production build without public source maps.
- The error comes from a third-party library.
- The deployed code does not match the source code you are looking at.

In development, source maps are usually enabled by default. In production, teams often hide source maps from the public and upload them privately to error tracking tools such as Sentry. That lets developers map production errors back to source code without exposing readable source files to every visitor.

## Is It Ever Really a Chrome Bug?

Real Chrome bugs exist, but they are less common than app bugs.

Before assuming Chrome is wrong, try to isolate the issue:

- Does it happen in a small example?
- Does it happen in another browser?
- Does it happen without React?
- Does it happen with browser extensions disabled?
- Does the Console show an app error first?

Most of the time, the problem is in application code, build configuration, dependency behavior, or CSS. If the issue only happens in one browser and you can reproduce it with a tiny example, then it may be a browser-specific issue.

## Final Mental Model

The most useful beginner model is:

```text
You write JSX
  -> build tool compiles JSX into JavaScript
  -> Chrome runs the JavaScript
  -> React creates element descriptions
  -> React compares old and new UI trees
  -> React updates the real DOM
  -> Chrome displays the page
```

When something breaks, debugging goes in the opposite direction:

```text
Chrome error
  -> source map
  -> original JSX file
  -> component props/state/events
  -> fix the source code
```

That is why frontend developers can work comfortably even though the browser is running transformed code. The build step creates the code Chrome needs, and source maps help developers find their way back to the code they actually wrote.
