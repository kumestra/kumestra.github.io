---
title: "Hermes Agent Interfaces: TUI, Bots, and Web Frontends"
description: A comparison of Hermes Agent's native interfaces and general chat frontends, with a practical rule for choosing where tools and skills should live.
pubDatetime: 2026-06-02T05:54:01Z
tags: [hermes-agent, ai-agents, tui, telegram, llm-frontends]
---

Hermes Agent has a strong backend story: it can run as an agent with tools, skills, memory, provider configuration, and gateway integrations. The harder question is not whether Hermes can do useful work. It is which interface should sit in front of it.

There are three common choices:

- the Hermes terminal UI
- bot interfaces such as Telegram or Discord
- general LLM web frontends such as Open WebUI, LobeChat, or LibreChat

Each one optimizes for a different thing. The terminal UI is closest to the agent. Bot interfaces are convenient and mobile-friendly. Web frontends are visually polished, but they usually understand Hermes only as an OpenAI-compatible API.

That difference matters because modern agents are not just text generators. They have tools, skills, files, approvals, memories, and intermediate state. A frontend that cannot represent those things directly can still be useful, but it is not the same as a native agent UI.

## The three layers

It helps to separate the system into layers:

```text
User interface -> Agent runtime -> Model provider
```

For Hermes, the agent runtime is the important part. Hermes can sit between the user interface and the raw model provider:

```text
Frontend -> Hermes Agent -> model provider
```

In that shape, the frontend should mostly be responsible for display and input. Hermes should own the agent behavior.

That is different from a normal OpenAI-compatible model setup:

```text
Frontend -> raw model API
```

In the normal setup, the frontend often owns tools. It sends a tool list to the model, receives tool calls, executes them, and sends tool results back. With Hermes, there is already an agent runtime in the middle. If the frontend also tries to act as the tool owner, the system can become confusing.

## Hermes TUI

The Hermes TUI is the most direct way to use Hermes. It is close to the runtime, so it is usually the best place to test configuration, providers, tools, and skills.

The strength of the TUI is fidelity. It is not trying to make Hermes look like a generic chat model. It is the native surface where Hermes-specific behavior is most likely to work as intended.

That makes it useful for:

- initial setup
- testing tools and skills
- diagnosing provider or gateway problems
- understanding how Hermes behaves without another UI layer in the way
- using Hermes locally with minimal infrastructure

The weakness is user experience. A terminal is not a modern chat app. Markdown rendering is limited. Rich media is awkward. Image, video, and audio workflows are not naturally pleasant. The TUI can be powerful, but it is not the interface most people want to live in all day.

The TUI is the best tool for truth. It is not always the best tool for comfort.

## Telegram and Discord bots

Bot interfaces solve a different problem: access. A Telegram or Discord bot is easy to use from a phone, easy to reach remotely, and familiar as a conversational surface.

For Hermes, that can be valuable. A bot can expose native agent behavior more directly than a generic OpenAI-compatible web frontend, especially if the bot integration was designed around Hermes itself.

Bots are good for:

- quick remote access
- mobile chat
- notifications
- casual agent interaction
- simple commands
- running Hermes without opening a dedicated web app

But chat platforms were not designed specifically for LLM agents. They have their own message formatting rules, rate limits, threading models, attachment behavior, and streaming limitations.

That means bots can feel imperfect for serious LLM use:

- markdown may render inconsistently
- streaming may not feel as smooth as a native LLM app
- long responses can be split awkwardly
- tool traces may be hard to display
- approvals and intermediate agent state may not map cleanly to the platform
- images, files, and rich media are constrained by the bot platform

Telegram and Discord are useful delivery channels. They are not a full replacement for a purpose-built agent client.

## General web frontends

General LLM web frontends such as Open WebUI, LobeChat, and LibreChat have the opposite tradeoff.

They are often excellent as chat applications. They can provide:

- polished markdown rendering
- conversation history
- user accounts
- model switching
- file upload interfaces
- image display
- browser-based access
- mobile-friendly layouts
- settings pages

For day-to-day chat UX, this can be much better than a terminal or a bot.

The catch is that these frontends are usually built around model APIs, not Hermes-native agent semantics. They can connect to Hermes through an OpenAI-compatible endpoint, but that does not automatically make them understand Hermes-specific features.

The frontend may see Hermes as something like this:

```text
OpenAI-compatible base URL + API key + model name
```

But Hermes is actually doing more:

```text
OpenAI-compatible endpoint -> Hermes Agent -> tools, skills, memory, providers
```

That can work well if the frontend behaves like a clean display pane. It can work poorly if the frontend also tries to run its own tools, memory, web search, RAG, or function calling on top of Hermes.

## One tool owner

The most important design rule is simple:

```text
Use one tool owner.
```

If Hermes is the agent backend, Hermes should own tools and skills.

That means a web frontend connected to Hermes should usually disable its own:

- web search
- tools and plugins
- frontend-side function calling
- frontend memory
- automatic RAG injection
- prompt enhancement features

Then the flow stays clean:

```text
Web frontend -> Hermes Agent -> Hermes tools and skills -> model provider
```

If both layers try to use tools, problems can appear:

| Issue | What happens |
|---|---|
| Duplicate search | The frontend searches the web, then Hermes searches again. |
| Conflicting context | The frontend injects one set of facts while Hermes retrieves another. |
| Unclear authority | It becomes hard to know which layer should decide tool use. |
| Extra latency | Two systems may plan, retrieve, summarize, or call tools. |
| Messy traces | Tool progress may not display cleanly in the web UI. |
| Security confusion | Frontend tools and Hermes tools may have different permissions. |

This is not always fatal, but it is usually unnecessary. A clean architecture is easier to reason about.

## The missing ideal client

The ideal Hermes interface would combine both sides:

```text
Hermes-native behavior + modern ChatGPT-style UX
```

That would mean a web or mobile client designed around Hermes from the beginning, with first-class support for:

- markdown rendering
- smooth response streaming
- images
- audio
- video
- file uploads and downloads
- tool progress
- skill invocation
- approvals
- memory visibility
- agent state
- conversation history
- mobile ergonomics

That is different from simply pointing a generic OpenAI-compatible frontend at Hermes. A generic frontend can show the conversation, but a Hermes-native frontend could understand what Hermes is doing.

This is the real gap: Hermes can be a capable agent runtime, but the richest user experience requires a client that knows how to represent agent behavior.

## Practical recommendation

Use each interface for what it is best at:

| Interface | Best use |
|---|---|
| Hermes TUI | Native testing, setup, tool and skill debugging, local operation. |
| Telegram or Discord bot | Convenient remote and mobile access. |
| Open WebUI, LobeChat, LibreChat | Polished browser chat UI connected to Hermes as an agent backend. |
| Raw model provider frontend | Cases where the frontend should own tools instead of Hermes. |

For a generic web frontend connected to Hermes, use this mental model:

```text
Frontend = chat window
Hermes = agent
Model provider = raw intelligence
```

So the recommended configuration is:

```text
Frontend tools: off
Frontend web search: off
Frontend memory: off
Frontend RAG: off, unless intentionally needed
Hermes tools and skills: on
```

There can be exceptions. A frontend document system may be useful if the goal is to inject a specific knowledge base before Hermes sees the prompt. But that should be an intentional design choice, not an accidental combination of two agent systems.

## Conclusion

The Hermes TUI is likely the most faithful interface. Telegram and Discord bots are convenient native channels. General web frontends provide the best visual chat experience, but they are usually not full Hermes clients.

The best current compromise is to let Hermes do the real agent work and let the web frontend act as a polished display surface. That gives a clean separation:

```text
good UI outside, real agent inside
```

The best future would be a Hermes-native web and mobile client: something with the polish of modern LLM apps and the full semantics of Hermes tools, skills, memory, files, and approvals.
