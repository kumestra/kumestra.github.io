---
title: "推理模型时间线：从 OpenAI o1 到默认 Thinking 能力"
description: "梳理 2024 年 o1 之后推理模型从单独品类走向开源破圈、混合模式和 agent 工具使用的演进脉络。"
pubDatetime: 2026-06-11T03:23:27Z
tags: [ai, llm, reasoning, thinking-models, timeline]
---

今天说“推理模型”，通常不是指模型会在回答里写一大段解释，而是指它会在回答前投入额外的推理预算：多走几步、检查中间结果、必要时调用工具，并用更长的 test-time compute 换取更高的正确率。

这条路线在研究上可以追溯到 scratchpad、Chain-of-Thought、self-consistency、ReAct、Tree of Thoughts 等方法。但如果只看大众产品和模型发布节奏，真正的分水岭是 2024 年 9 月的 OpenAI o1-preview。o1 之后，“thinking” 从提示词技巧变成了模型产品的一等能力。

下面这条时间线按公开发布或公开可用时间整理，重点放在 o1 之后的节点。截至时间：2026-06-11。

## 先给一个判断框架

推理模型的发展可以分成四个阶段：

1. **品类定义阶段**：OpenAI o1 把“回答前花时间思考”产品化。
2. **开放权重扩散阶段**：QwQ、DeepSeek-R1 等让推理模型不再只属于闭源 API。
3. **混合推理阶段**：模型开始支持 Thinking / Non-Thinking、reasoning effort、adaptive reasoning。
4. **agent 化阶段**：推理不再只用于解数学题，而是和搜索、代码执行、文件、浏览器、工具调用、多步任务结合。

## 主时间线

| 时间 | 模型或事件 | 关键意义 |
| --- | --- | --- |
| 2024-09 | OpenAI 发布 **o1-preview / o1-mini** | 推理模型第一次被大规模产品化。OpenAI 明确把它描述为会在回答前花更多时间思考，面向科学、数学和代码等复杂任务。来源：[Introducing OpenAI o1-preview](https://openai.com/index/introducing-openai-o1-preview/) |
| 2024-11 | Qwen 发布 **QwQ-32B-Preview** | 开放权重阵营较早的 o1-like 推理模型，强调自我质疑、反思和深度推理。来源：[QwQ-32B-Preview](https://qwenlm.github.io/blog/qwq-32b-preview/) |
| 2024-12 | OpenAI 发布正式版 **o1** 和 ChatGPT Pro | o1 从 preview 走向正式产品，并出现更高推理预算的 Pro 形态。来源：[OpenAI o1](https://openai.com/index/introducing-openai-o1-preview/) |
| 2024-12 | Google 推出 Gemini 2.0 体系和 Flash Thinking | Google 开始把 multi-step reasoning、agentic models、Deep Research 等能力放进 Gemini 产品线。来源：[Gemini 2.0](https://blog.google/technology/google-deepmind/google-gemini-ai-update-december-2024/) |
| 2024-12 | Qwen 发布 **QvQ-72B-Preview** | 推理模型进入多模态开放权重阶段，重点是视觉、图表、物理和数学问题的 step-by-step visual reasoning。来源：[QvQ-72B-Preview](https://qwenlm.github.io/blog/qvq-72b-preview/) |
| 2025-01 | DeepSeek 发布 **DeepSeek-R1** | 开源推理模型真正破圈。R1 以 MIT 许可开放模型和技术报告，并发布多个蒸馏模型，性能对标 OpenAI o1。来源：[DeepSeek-R1 Release](https://api-docs.deepseek.com/news/news250120) |
| 2025-01 | OpenAI 发布 **o3-mini** | 推理模型开始降成本、降延迟，并提供 low / medium / high reasoning effort。来源：[OpenAI o3-mini](https://openai.com/index/openai-o3-mini/) |
| 2025-02 | xAI 发布 **Grok 3 Think** | Grok 进入推理模型赛道，推出 Grok 3 Think 和 Grok 3 mini Think，强调强化学习和 test-time compute。来源：[Grok 3](https://x.ai/news/grok-3) |
| 2025-02 | Anthropic 发布 **Claude 3.7 Sonnet** | Anthropic 提出 hybrid reasoning：同一个模型既能快速回答，也能 extended thinking。来源：[Claude 3.7 Sonnet](https://www.anthropic.com/news/claude-3-7-sonnet) |
| 2025-03 | Google 发布 **Gemini 2.5 Pro** | Google 明确称 Gemini 2.5 是 thinking model，并表示后续会把 thinking 能力内建到所有模型中。来源：[Gemini 2.5](https://blog.google/technology/google-deepmind/gemini-model-thinking-updates-march-2025/) |
| 2025-03 | Qwen 发布 **QwQ-32B** | Qwen 用强化学习扩展 32B 推理模型，开放权重并对比 DeepSeek-R1。来源：[QwQ-32B](https://qwenlm.github.io/blog/qwq-32b/) |
| 2025-04 | OpenAI 发布 **o3 / o4-mini** | 推理模型和工具使用开始深度结合，包括搜索、Python、文件、图像等。来源：[o3 and o4-mini](https://openai.com/index/introducing-o3-and-o4-mini/) |
| 2025-04 | Qwen 发布 **Qwen3** | 开源模型进入混合思考阶段，支持 Thinking Mode 和 Non-Thinking Mode，并允许按任务控制思考预算。来源：[Qwen3](https://qwenlm.github.io/blog/qwen3/) |
| 2025-05 | Anthropic 发布 **Claude Opus 4 / Sonnet 4** | Claude 把 extended thinking 和 tool use 结合起来，推向代码、agent 和长任务。来源：[Claude 4](https://www.anthropic.com/news/claude-4) |
| 2025-06 | Mistral 发布 **Magistral Small / Medium** | 欧洲主要模型公司进入 reasoning 赛道，开源 Magistral Small，并强调多语言推理。来源：[Magistral](https://mistral.ai/news/magistral/) |
| 2025-07 | xAI 发布 **Grok 4 / Grok 4 Heavy** | Grok 继续扩大 RL 和工具使用能力，把 reasoning 与实时搜索、代码解释器等结合。来源：[Grok 4](https://x.ai/news/grok-4) |
| 2025-08 | OpenAI 发布 **gpt-oss-120b / gpt-oss-20b** | OpenAI 发布开放权重推理模型，支持 reasoning effort、工具使用和完整 chain-of-thought 输出。来源：[gpt-oss](https://openai.com/index/introducing-gpt-oss/) |
| 2025-08 | OpenAI 发布 **GPT-5** | OpenAI 从“手动选择推理模型”转向统一系统：快速模型、GPT-5 thinking 和实时路由器。来源：[GPT-5](https://openai.com/index/introducing-gpt-5/) |
| 2025-08 | DeepSeek 发布 **DeepSeek-V3.1** | DeepSeek 也进入混合推理阶段，一个模型支持 Think 和 Non-Think 两种模式。来源：[DeepSeek-V3.1](https://api-docs.deepseek.com/news/news250821) |
| 2025-09 | Anthropic 发布 **Claude Sonnet 4.5** | Claude 强化 coding、agents、computer use，同时提升 reasoning 和 math。来源：[Claude Sonnet 4.5](https://www.anthropic.com/news/claude-sonnet-4-5) |
| 2025-11 | OpenAI 发布 **GPT-5.1 Instant / Thinking** | GPT-5 系列进一步分化为 Instant 和 Thinking；Instant 也能在复杂问题上自适应启用推理。来源：[GPT-5.1](https://openai.com/index/gpt-5-1/) |
| 2025-11 | Anthropic 发布 **Claude Opus 4.5** | Opus 4.5 强调 coding、agents、computer use 和复杂工作流中的持续推理。来源：[Claude Opus 4.5](https://www.anthropic.com/news/claude-opus-4-5) |
| 2025-12 | DeepSeek 发布 **DeepSeek-V3.2 / V3.2-Speciale** | DeepSeek 称其为 reasoning-first models built for agents，并把 thinking 直接融入 tool-use。来源：[DeepSeek-V3.2](https://api-docs.deepseek.com/news/news251201) |
| 2025-12 | Google 推出 **Gemini 3 Deep Think** | Deep Think 强调 parallel reasoning，同时探索多个假设，面向复杂数学、科学和逻辑问题。来源：[Gemini 3 Deep Think](https://blog.google/products/gemini/gemini-3-deep-think/) |
| 2026-04 | DeepSeek 发布 **DeepSeek-V4 Preview** | V4-Pro / V4-Flash 支持 1M 上下文和 Thinking / Non-Thinking 双模式，继续推进开源 reasoning 与 agent 能力。来源：[DeepSeek-V4 Preview](https://api-docs.deepseek.com/news/news260424) |

## o1 做了什么

o1 的重要性不在于它是第一个“会推理”的模型。早在 o1 之前，研究者已经用 scratchpad、chain-of-thought、self-consistency、ReAct 等方法证明：让模型写中间步骤、采样多条路径、和工具交互，都会提升复杂任务表现。

o1 的真正变化在于产品形态。

它把“多花时间思考”变成一个明确的模型能力，而不是用户在 prompt 里写一句“let's think step by step”。OpenAI 也把它和 GPT-4o 这类通用快速模型区分开来：普通问题用快速模型，复杂数学、代码、科学问题用推理模型。

这就是推理模型被大众感知为一个新品类的原因。

## DeepSeek-R1 为什么是第二个大节点

如果 o1 是闭源产品化的起点，DeepSeek-R1 就是开放权重推理模型破圈的起点。

R1 的影响来自几件事叠加：

1. **性能叙事清楚**：官方直接对标 OpenAI o1。
2. **开放程度高**：模型和技术报告发布，许可相对宽松。
3. **蒸馏链条完整**：从 1.5B 到 70B 的蒸馏模型让更多人能在本地或低成本环境里实验推理模型。
4. **训练路线有启发性**：R1-Zero 展示了用强化学习激发长链推理的可能性。

所以 R1 不一定是时间上第一个开放权重推理模型，但它是第一个真正让全球开发者、投资人、研究者和普通用户同时注意到“开源推理模型”的大事件。

## 2025 年之后的主线：推理不再是单独模型

2025 年中后期开始，趋势变了。

早期的模式是：“普通模型”和“推理模型”分开。你需要在模型列表里手动选 o1、o3、R1、QwQ、Magistral。

后来的模式变成：“一个系统里自动决定要不要思考”。GPT-5 是典型代表：它不是单个模型，而是快速模型、GPT-5 thinking 和路由器组成的统一系统。GPT-5.1 又进一步让 Instant 也能做 adaptive reasoning。

Qwen3、DeepSeek-V3.1、DeepSeek-V4 走的是另一条相似路线：同一个模型支持 Thinking 和 Non-Thinking。用户或 API 可以根据任务决定要速度，还是要更深的推理。

Claude 3.7 之后的路线也类似，只是 Anthropic 更喜欢称它为 hybrid reasoning：一个模型既是普通 LLM，也是推理模型。

## 另一个主线：推理和工具合流

推理模型刚出现时，最常见的展示是数学题、竞赛代码、逻辑题。

但它真正改变产品体验，是从工具使用开始的：

- 搜索：模型自己判断要查什么、如何综合来源。
- Python / code execution：模型把计算、验证、实验交给工具。
- 文件与长上下文：模型能读代码库、PDF、表格、日志。
- 浏览器与计算机使用：模型能执行多步操作，而不只是给建议。
- agent 工作流：模型把任务拆成计划、执行、检查、修复。

OpenAI o3/o4-mini、Grok 4、Claude 4、DeepSeek-V3.2、Gemini 3 Deep Think 都在往这个方向走。推理模型的核心场景正在从“答难题”变成“做长任务”。

## 今天看推理模型，可以抓住四个指标

评估一个推理模型，不应该只看它是不是会输出长思考文本。更实用的指标是：

1. **推理预算是否可控**：有没有 reasoning effort、thinking budget、Think/Non-Think。
2. **推理是否高效**：多花 token 是否真的换来更高正确率，而不是空转。
3. **是否能用工具验证自己**：代码执行、搜索、文件分析、函数调用是否融入推理过程。
4. **是否能完成长任务**：代码库修改、研究报告、复杂表格、跨工具 workflow 是否稳定。

这也是为什么后来的模型越来越少强调“我能想很久”，越来越多强调“我能用工具把事情做完”。

## 小结

推理模型的发展可以用一句话概括：

**o1 把 thinking model 变成产品类别；DeepSeek-R1 把开放权重推理模型推到全球视野；GPT-5、Qwen3、Claude、Gemini 和 DeepSeek V 系列则把推理变成默认能力。**

接下来，“推理模型”这个词可能会越来越少被单独提起。不是因为它不重要，而是因为它会像上下文窗口、工具调用、多模态一样，变成大模型的基本配置。
