---
title: How to include translation in SillyTavern replies
description: A practical guide to making SillyTavern keep the original AI response while appending a Chinese translation after it.
pubDatetime: 2026-06-01T02:06:59Z
tags: [sillytavern, llm, roleplay, translation, prompt-engineering]
---

SillyTavern is flexible because it is mostly a prompt assembly tool. Every time you send a message, it gathers the character card, chat history, system prompts, Author's Note, World Info, presets, and formatting instructions, then sends that combined prompt to the model.

That means a bilingual response format is usually not a code change. You can ask the model to write the normal reply first, then append a translation below it.

The goal is:

```text
Normal AI response, in the usual style.

中文翻译：
The same response translated into Simplified Chinese.
```

This is useful when you want to keep the original English roleplay, tone, and formatting, but also want a readable Chinese version immediately under the original message.

## The best place to put the instruction

For a single chat, use **Author's Note**. For every chat using the same preset, use **Post-History Instructions**.

Author's Note is convenient because it is attached to the current chat and easy to turn on or off. Post-History Instructions are stronger because they are placed near the end of the prompt that SillyTavern sends to the model.

Use Author's Note first. If the model ignores it, move the same instruction to Post-History Instructions.

## Recommended Author's Note setup

Open the Author's Note panel and use:

```text
Placement: In-chat
Depth: 0
Role: System
Insertion Frequency: 1
```

What these mean:

| Setting | Meaning | Recommended value |
|---|---|---|
| Author's Note | Hidden instruction text injected into the prompt | Put the translation rule here |
| In-chat | Insert the note into the chat-history area | Use this for per-chat behavior |
| Depth | Where the note appears in chat history | `0`, meaning near the latest message |
| Role | How the note is presented to the model | `System` |
| Insertion Frequency | How often the note is inserted | `1`, meaning every user message |

Depth matters because newer instructions tend to have stronger influence. For a formatting rule that should affect the next response, `Depth 0` is usually the right choice.

Insertion Frequency matters because a translation rule should apply every time. `0` disables insertion. `1` means always insert it.

## Prompt template

Use this short version first:

```text
[Final visible reply format:
First write {{char}}'s normal English roleplay response as usual.

Then append:

中文翻译：
A faithful Simplified Chinese translation of the English response above.

Do not put the translation in reasoning/thinking. Do not add new content in Chinese.]
```

The wording is intentionally strict about "final visible reply" because some reasoning models may put the Chinese translation inside the hidden or collapsible thinking block instead of the visible answer.

If the model still disobeys, use the stronger version:

```text
[FINAL VISIBLE MESSAGE FORMAT - MANDATORY

Write the final visible reply in this exact structure:

{{char}}'s normal English roleplay response, as usual.

中文翻译：
A faithful Simplified Chinese translation of the English response above.

Rules:
- The first visible part must be the normal English response.
- The Chinese translation must appear after the English response in the visible message body.
- Do not put the Chinese translation inside reasoning, thinking, analysis, or <think> blocks.
- The Chinese section must only translate the English response. Do not add new story content.]
```

## Token settings

Bilingual replies are longer. A response with the original English plus Chinese translation can easily require twice as many output tokens as the original response.

Do not set **Max Response Length** equal to **Context Size**. Context Size is the total room for prompt plus reply. Max Response Length is only the reserved room for the new answer. If both are nearly the same, SillyTavern has no room left for mandatory prompts, character information, chat history, or Author's Note.

Good starting points:

| Model context window | Context Size | Max Response Length |
|---|---:|---:|
| 4K context | `4096` | `768` to `1024` |
| 8K context | `8192` | `1024` to `1536` |
| 16K+ context | `16384` or higher | `1536` to `2048` |

If you see this warning:

```text
Mandatory prompts exceed the context size.
```

reduce Max Response Length, increase Context Size if the model supports it, or reduce the amount of prompt/history being sent.

## Reasoning models can cause a weird failure mode

Some models expose a reasoning or thinking block in SillyTavern, often displayed as something like "Thought for N seconds". If the translation instruction is not specific enough, the model may put the Chinese translation into that reasoning block and then produce only the normal English response afterward.

If that happens:

- Use the phrase "final visible reply" in the instruction.
- Tell the model not to put translation inside reasoning, thinking, analysis, or `<think>` blocks.
- Lower or disable reasoning effort if your backend supports it.
- Collapse or hide reasoning display in SillyTavern if you do not want to see it.
- Increase Max Response Length enough for both the original reply and translation.

This is not really a translation problem. It is a placement problem: the model followed part of the instruction, but placed the translation in the wrong output channel.

## When to use Post-History Instructions instead

Use Post-History Instructions if Author's Note keeps being ignored.

Post-History Instructions are useful for final formatting rules because they appear after the main chat history in the assembled prompt. That makes them a good place for global output structure, such as:

```text
Always answer with the original English response first, then append a Simplified Chinese translation under "中文翻译：".
```

The tradeoff is scope. Author's Note is better for one chat. Post-History Instructions are better for a preset or general habit.

## Troubleshooting checklist

- If the model answers only in Chinese, say "Do not replace the English response."
- If the Chinese appears before English, say "The first visible part must be English."
- If the translation appears in the thought block, say "final visible reply" and forbid reasoning/thinking placement.
- If the translation is cut off, increase Max Response Length or reduce reply length.
- If SillyTavern says mandatory prompts exceed context, lower Max Response Length or raise Context Size.
- If the model ignores Author's Note, move the instruction to Post-History Instructions.

## A compact working setup

For most chats, this is enough:

```text
Author's Note:
[Final visible reply format:
First write {{char}}'s normal English roleplay response as usual.

Then append:

中文翻译：
A faithful Simplified Chinese translation of the English response above.

Do not put the translation in reasoning/thinking. Do not add new content in Chinese.]

Placement: In-chat
Depth: 0
Role: System
Insertion Frequency: 1

Context Size: 8192, if the model supports it
Max Response Length: 1024-1536
```

The important idea is simple: keep the original response as the primary message, then append the translation as a second section. SillyTavern does not need a special translation plugin for this. It only needs a clear output-format instruction placed close enough to the next generation, with enough token budget for both versions of the reply.

## References

- [SillyTavern Author's Note documentation](https://docs.sillytavern.app/usage/core-concepts/authors-note/)
- [SillyTavern prompt management documentation](https://docs.sillytavern.app/usage/prompts/)
- [SillyTavern reasoning documentation](https://docs.sillytavern.app/usage/prompts/reasoning/)
