---
title: Python dataclass Explained
description: What @dataclass does, why it exists, and when to use it — explained through a real example from an AI agent codebase.
pubDatetime: 2026-04-13T09:27:29Z
tags: [python, dataclass, syntax-sugar, oop]
---

## What is `@dataclass`?

`@dataclass` is a decorator that automatically generates boilerplate methods for a class — primarily `__init__`, but also `__repr__` and `__eq__`.

It is **syntax sugar**. The two versions below are functionally identical:

```python
# With @dataclass
from dataclasses import dataclass, field
import threading

@dataclass
class ToolUseContext:
    messages: list
    cwd: str
    abort_signal: threading.Event = field(default_factory=threading.Event)
    options: dict = field(default_factory=dict)
```

```python
# Plain class — equivalent behaviour
import threading

class ToolUseContext:
    def __init__(self, messages, cwd):
        self.messages = messages
        self.cwd = cwd
        self.abort_signal = threading.Event()
        self.options = {}
```

Both let you write:

```python
ctx = ToolUseContext(messages=[], cwd="/home/user/project")
```

`@dataclass` just saves you from writing the `__init__` by hand.

## What does it buy you?

| Feature | What it generates |
|---|---|
| `__init__` | Constructor with one parameter per annotated field |
| `__repr__` | Readable string like `ToolUseContext(messages=[], cwd='.')` |
| `__eq__` | Equality comparison by field values (opt-in) |

For most use cases, `__init__` and `__repr__` are the only ones that matter.

## The `field()` function

When a field needs a **default value that is mutable** (a list, dict, or any object), you cannot write:

```python
# WRONG — one Event shared across ALL instances
abort_signal: threading.Event = threading.Event()
```

Python evaluates the default once at class definition time, so every instance would share the same object. Instead, `field(default_factory=...)` tells `@dataclass` to call the factory fresh for each new instance:

```python
abort_signal: threading.Event = field(default_factory=threading.Event)
```

This generates:

```python
self.abort_signal = threading.Event()  # new Event per instance
```

## A real example: `ToolUseContext`

In an AI agent, each user turn needs a shared execution environment — the current directory, the conversation history, a cancellation signal. Rather than passing these as separate arguments to every function:

```python
# Without ToolUseContext — signature bloat
def call(self, args, messages, cwd, abort_signal, options):
    ...
```

They are bundled into one object:

```python
# With ToolUseContext — stable signature
def call(self, args: dict, ctx: ToolUseContext) -> ToolResult:
    ...
```

When you later need to add a new field to the environment, you add it to `ToolUseContext` once. Every tool that needs it reads `ctx.whatever`. Tools that don't need it ignore it. The `call()` signature never changes.

`@dataclass` is the right tool here because `ToolUseContext` is a pure data holder — it has no behaviour of its own, just fields. That is exactly the case `@dataclass` was designed for.

## When to use `@dataclass`

✅ Use it when the class is a **data holder** — its purpose is to group related fields together, and it has little or no behaviour.

✅ Use it when you want clean `__repr__` output for debugging without writing it yourself.

❌ Don't use it when the class has complex initialisation logic that a generated `__init__` cannot express.

❌ Don't use it just because it looks shorter — a plain class is equally valid and sometimes clearer.

## Summary

`@dataclass` reads the class-level type annotations and generates an `__init__` (and `__repr__`) for you. It is syntax sugar — no new runtime behaviour, no magic. The plain-class equivalent is always available if you need more control. Use `field(default_factory=...)` any time a default value is mutable.
