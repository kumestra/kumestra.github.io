---
title: "理解 SillyTavern 里的 Preset"
description: "一篇用请求管线的角度解释 SillyTavern preset 心智模型、用途和寻找方式的笔记。"
pubDatetime: 2026-06-15T09:57:11Z
tags: [SillyTavern, preset, LLM, roleplay]
---

## 为什么 preset 容易让人困惑？

在 SillyTavern 里，**preset** 不是传统意义上的“唯一当前配置文件”。它更像是某个设置领域的一组已保存状态：你可以保存、选择、复用它，但它通常只负责整个生成流程中的一部分。

这也是它容易让人困惑的地方：多个 preset 可以同时处于生效状态，而且它们的影响范围有时会重叠。比如，一个 system prompt preset、一个 instruct template、一个 context template 都可能影响最终发给模型的提示词。它们并不总是像“一个文件负责一个完全独立的职责”那样整齐。

更清楚的心智模型是：

> preset 是一个已保存的设置束；真正生效的是多个设置束、聊天状态、角色卡、世界信息、persona 和后端配置共同组成的最终请求。

## 一个更实用的心智模型 🧭

不要把 preset 理解成“完整配置档案”。更适合的理解是：

> **preset = 某个控制面板的已保存快照。**

最终模型收到的请求，则是多个快照叠加之后组装出来的结果。

| 层级 | 常见 preset 类型 | 主要影响 |
|---|---|---|
| 生成行为 | textgen / OpenAI / Kobold / NovelAI preset | temperature、top-p、重复惩罚、最大长度等 |
| 提示词结构 | instruct template、context template | 用户、助手、系统消息如何被包裹和排列 |
| 提示词内容 | system prompt preset | 模型被要求遵守的行为指令 |
| 聊天数据 | 角色卡、persona、世界信息、聊天历史 | 通常不是同一种 preset，但会参与最终请求 |

因此，一个 sampler preset 和一个 system prompt preset 通常是相对独立的：前者改变模型如何采样，后者改变模型看到什么指令。

但 system prompt、instruct template、context template 之间可能都在塑造最终提示词，所以它们更像是**共同参与同一个请求管线**，而不是完全互不相干的配置文件。

## preset 能做什么？

一个 preset 可以保存并复用某一类设置。例如：

- 改变采样行为：temperature、top-p、top-k、重复惩罚、最大回复长度。
- 改变提示词格式：用户、助手、系统消息如何被包装。
- 改变 instruct 风格：例如 ChatML、Alpaca、Llama、Mistral 之类的格式。
- 改变 system prompt 内容：给模型的长期行为指令。
- 改变特定后端选项：例如 OpenAI、KoboldCpp、text-generation-webui、NovelAI 等后端相关设置。

可以把它想象成几个同时生效的旋钮面板：一个负责创作温度，一个负责消息格式，一个负责角色扮演行为。最终结果不是某一个 preset 单独决定的，而是这些面板一起决定的。

## 一个实际例子：角色扮演组合

假设你同时启用了这些 preset：

| preset | 作用 |
|---|---|
| `Creative Writing` text generation preset | 使用较高 temperature、较宽 top-p、更长输出长度，让回复更适合创作 |
| `ChatML` instruct preset | 把 system、user、assistant 消息包装成 ChatML 风格 |
| `Roleplay Assistant` system prompt preset | 要求模型保持角色、避免总结、不要代写用户行为 |

最终请求大致会像这样：

```text
[来自 instruct preset 的 ChatML 格式]

System:
[来自 system prompt preset 的角色扮演指令]

Context:
[角色描述、persona、世界信息、聊天历史]

User:
用户的最新输入

Generation settings:
temperature 0.9
top_p 0.95
max_tokens ...
```

这里没有一个 preset 是“唯一配置”。它们分别贡献格式、内容和采样参数，然后被 SillyTavern 组装成一次模型调用。

## 一个可用的 system prompt preset 示例 ✍️

下面是一个适合角色扮演聊天的 system prompt preset。它的职责很窄：让模型成为更稳定的角色扮演伙伴。

```json
{
  "name": "Grounded Roleplay Partner",
  "content": "You are writing {{char}}'s next reply in an ongoing fictional chat with {{user}}.\n\nStay grounded in the established character sheet, scenario, world info, and chat history. Treat those details as authoritative. Do not contradict them unless the story clearly establishes a reason.\n\nWrite in a natural, character-driven style. Prioritize emotional continuity, believable reactions, concrete sensory detail, and clear scene progression. Avoid summarizing the scene from a distance; instead, continue it through action, dialogue, internal reaction, and immediate consequences.\n\nKeep {{char}}'s voice distinct from {{user}}'s voice. Do not write {{user}}'s actions, thoughts, dialogue, or decisions unless the chat history explicitly requires it. Leave meaningful room for {{user}} to respond.\n\nDo not mention system prompts, character sheets, rules, tokens, or AI behavior in the reply.",
  "post_history": "Before replying, silently check: What just happened? What does {{char}} want right now? What would {{char}} naturally say or do next? Then write only {{char}}'s next reply."
}
```

它适合：

- 角色扮演聊天
- 长期剧情对话
- 需要保持角色卡和世界信息一致性的场景
- 情绪连续、场景推进、对话自然的创作型聊天

它不太适合：

- 普通问答助手
- 需要大纲、分析、总结的写作任务
- 故意追求混乱或超现实风格的聊天
- 需要模型同时书写用户和角色双方行为的场景

## 如何判断 preset 是否合理？

一个实用的检查方法是：不要只看 preset 名字，而要看它最终会影响哪一层。

- 它影响采样参数吗？
- 它影响 prompt 的文本内容吗？
- 它影响聊天格式吗？
- 它影响后端 API 行为吗？

如果两个 preset 都在写角色行为规则、回复格式、停止符、越狱文本或风格限制，就要小心重复或冲突。真正的判断依据不是 preset 的标题，而是最终组装出的 prompt/request 是否清晰、一致、没有互相打架。

## 去哪里找社区 preset？

可以从这些地方开始：

- **SillyTavern 内置的 Download Extensions & Assets**：适合找官方入口提供的社区资源。
- **官方 Discord**：适合寻找当前模型、当前后端、当前版本下的推荐设置。
- **r/SillyTavernAI**：适合搜索公开分享的 preset、prompt、模型经验。
- **官方 GitHub 默认 presets 目录**：适合参考 preset 的结构和分类。

搜索时可以用这些关键词：`preset`、`sampler`、`textgen`、`sysprompt`、`instruct`，或者直接搜索你正在用的模型名和后端名。

## 总结

理解 SillyTavern preset 的关键，是把它从“单一配置文件”这个概念里拿出来。

更准确的说法是：

> preset 是一组可复用的设置；多个 preset 可以同时参与最终请求；最终行为来自整个请求管线，而不是来自某一个 preset。

只要区分清楚“内容”“格式”“采样参数”“后端选项”这几层，preset 就会从一团重叠的配置，变成一套可以调试、组合和复用的工具。
