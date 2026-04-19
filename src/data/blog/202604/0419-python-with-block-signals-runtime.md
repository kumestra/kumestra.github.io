---
title: Python with Block, Signals, and the Runtime — When Does __exit__ Run?
description: A concept note on when Python's with block guarantees cleanup, how Unix signals (SIGINT, SIGTERM, SIGKILL) affect it, and what the Python runtime actually is.
pubDatetime: 2026-04-19T09:51:08Z
tags: [python, context-manager, signals, unix, runtime, cleanup]
---

A common question when using `with` blocks: is cleanup *always* guaranteed? The answer depends on what stops the process — and understanding that requires knowing about Unix signals and what the Python runtime actually is.

## The `with` block guarantee

`with` is syntax sugar for `try/finally`:

```python
with open("file.txt") as f:
    data = f.read()
```

Exactly equivalent to:

```python
cm = open("file.txt")
f = cm.__enter__()
try:
    data = f.read()
finally:
    cm.__exit__(None, None, None)
```

The `finally` means `__exit__` always runs for normal Python control flow — exceptions, `return`, `break`, `continue`. Whatever happens inside the block, cleanup is guaranteed.

```python
with open("file.txt") as f:
    raise ValueError("oops")   # __exit__ still runs
    return                      # __exit__ still runs
    break                       # __exit__ still runs
```

---

## Three tiers of process termination

Whether `__exit__` runs depends entirely on *how* the process stops.

### 🟢 Python exceptions — cleanup always runs

Any exception propagating through a `with` block triggers `__exit__`. `KeyboardInterrupt` (from `Ctrl+C`) is just an exception — it propagates up the call stack and every `with` block it passes through gets cleaned up.

```python
with open("file.txt") as f:
    time.sleep(100)  # user hits Ctrl+C
# __exit__ runs — file is closed
```

### 🟡 OS signals — depends on the signal

Unix signals are a mechanism for the OS to notify a process of an event. Three matter here:

| Signal | Number | Sent by | Catchable? | Python effect |
|---|---|---|---|---|
| `SIGINT` | 2 | `Ctrl+C` | Yes | Raises `KeyboardInterrupt` |
| `SIGTERM` | 15 | `kill <pid>`, Docker, systemd | Yes | Can install a handler |
| `SIGKILL` | 9 | `kill -9 <pid>` | **No** | Process deleted immediately |

**`SIGINT`** — Python converts it to `KeyboardInterrupt`. Since it's an exception, `__exit__` runs. ✅

**`SIGTERM`** — the polite shutdown request. Docker `stop`, Kubernetes pod termination, and `systemctl stop` all send this first. Python can catch it with a signal handler and run cleanup before exiting. ✅ (if you handle it)

**`SIGKILL`** — the OS removes the process from memory immediately. No user code runs at all. `__exit__`, `finally`, signal handlers — none of it executes. ❌

### 🔴 Physical — cleanup never runs

Power cut, kernel panic, hardware failure. The process simply vanishes. No software can handle this at the language level — true for all languages, not just Python.

---

## SIGTERM vs SIGKILL — the polite/forced distinction

The key difference:

- **`SIGTERM`** — "please shut down". The process decides how and when. It can clean up, flush buffers, close connections, then exit.
- **`SIGKILL`** — the OS pulling the power plug in software. Not a request. The process gets no control.

Most tools use a two-step shutdown sequence:

```
1. send SIGTERM  → give the process a chance to clean up
2. wait N seconds
3. send SIGKILL  → force kill if it didn't exit
```

Docker `stop`, Kubernetes, and `systemctl stop` all follow this pattern. The waiting period is configurable (Docker default: 10 seconds).

### `Ctrl+C` edge case

A single `Ctrl+C` sends `SIGINT` → `KeyboardInterrupt` → `__exit__` runs. Safe.

Some terminals escalate to `SIGKILL` if you hit `Ctrl+C` twice rapidly. In that case, the second one may skip cleanup.

---

## What is the Python runtime?

The Python runtime is everything that runs your code — the engine underneath your script.

It includes:

- **Interpreter** — reads bytecode and executes it (CPython is the standard implementation, written in C)
- **Memory manager** — allocates and frees objects, runs the garbage collector
- **Built-in types** — `int`, `str`, `list`, `dict` implemented in C
- **Standard library** — `os`, `json`, `asyncio` etc.
- **Call stack** — tracks which function is currently executing
- **GIL** — (CPython) a lock ensuring only one thread runs Python bytecode at a time

The simplest mental model: **the Python runtime is a running Python process**. When you run `python script.py`, a process starts. That process IS the runtime — it holds the interpreter, your objects in memory, the call stack, everything.

When `SIGKILL` hits, the OS deletes the process. The runtime is gone. No more instructions execute — which is why no cleanup happens.

---

## Summary

```
Cause                   | __exit__ runs?
------------------------|---------------
Python exception        | ✅ always
Ctrl+C (SIGINT)         | ✅ yes (KeyboardInterrupt)
SIGTERM (kill)          | ✅ if handled
SIGKILL (kill -9)       | ❌ no
Power cut / crash       | ❌ no
```

The rule: `with` guarantees cleanup **within the Python runtime**. Anything that bypasses the runtime — OS force kill, hardware failure — cannot be handled at the language level.
