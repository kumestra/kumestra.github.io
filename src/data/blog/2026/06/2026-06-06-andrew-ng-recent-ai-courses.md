---
title: "吴恩达过去一年新课：从提示词到 Agentic AI 的学习路线"
description: "梳理吴恩达在过去 12 个月发布或参与的新课程，并把它们串成一条面向 AI 实用能力的学习路线。"
pubDatetime: 2026-06-06T09:20:21Z
tags: [ai, learning, andrew-ng]
---

截至 2026-06-06 往前看 12 个月，吴恩达在 DeepLearning.AI 上发布或参与了几门很有代表性的课程。它们不像传统机器学习课那样从模型、损失函数、优化算法讲起，而是更直接地围绕一个现实问题展开：普通人、开发者和数据工作者，怎样把当下的 AI 系统真正用起来？

这组课程大致可以看成四层能力：会提问、会用 AI 做应用、会在 Notebook 里协作写代码、会搭建 agentic workflow。

## 课程时间线

| 时间 | 课程 | 定位 | 官方链接 |
| --- | --- | --- | --- |
| 2025-10-08 | Agentic AI | 面向有 Python 基础的开发者，学习反思、工具调用、规划、多智能体等 agentic design patterns | [Course](https://www.deeplearning.ai/courses/agentic-ai/), [Announcement](https://www.deeplearning.ai/the-batch/check-out-our-course-on-how-to-build-ai-agents/) |
| 2025-11-06 | Jupyter AI: AI Coding in Notebooks | 吴恩达与 Brian Granger 合讲，面向想在 Jupyter Notebook 中使用 AI 辅助编码的人 | [Course](https://www.deeplearning.ai/courses/jupyter-ai-coding-in-notebooks/), [Announcement](https://community.deeplearning.ai/t/new-course-enroll-in-jupyter-ai-ai-coding-in-notebooks/882862) |
| 2026-01-09 | Build with Andrew | 面向零编程背景学习者，用自然语言和 AI 构建简单网页应用 | [Course](https://www.deeplearning.ai/courses/build-with-andrew/), [The Batch issue](https://www.deeplearning.ai/the-batch/issue-335/) |
| 2026-05-01 | AI Prompting for Everyone | 面向所有 AI 用户，学习搜索、深度研究、写作、多模态和简单应用构建中的提示方式 | [Course](https://www.deeplearning.ai/courses/ai-prompting-for-everyone/), [Announcement](https://www.deeplearning.ai/the-batch/learn-foundational-prompting-techniques-in-ai-prompting-for-everyone/) |

## 1. AI Prompting for Everyone：把提示词从小技巧变成基础素养

[AI Prompting for Everyone](https://www.deeplearning.ai/courses/ai-prompting-for-everyone/) 是这组课里最适合普通用户的一门。它的目标不是教人写复杂 prompt 模板，而是让人理解 2026 年的 AI 工具已经不只是“问一句答一句”的聊天框。

课程页显示，这是一门 beginner 课程，时长约 3 小时 4 分钟，由吴恩达主讲，包含 21 个视频课程和 3 个 graded assignments。课程覆盖三个常见场景：

- 找信息：什么时候依赖模型已有知识，什么时候启用 web search，什么时候使用 deep research。
- 思考和写作：如何提供足够上下文，让 AI 参与头脑风暴、批判性反馈和写作。
- 多媒体与代码：如何让 AI 理解图片、生成视觉内容、分析数据，并构建简单网站或应用。

这门课值得注意的地方在于，它把“会不会用 AI”从一个工具熟练度问题，提升成了信息判断和任务设计问题。真正的差别不只是 prompt 写得漂亮，而是知道什么时候该让模型检索、什么时候该让模型推理、什么时候该要求它提出反例和批判性反馈。

## 2. Build with Andrew：零代码人群的 AI 应用入门

[Build with Andrew](https://www.deeplearning.ai/courses/build-with-andrew/) 更像是一个非常短的入口课。课程页显示它是 beginner 级别，约 1 小时，包含 5 个视频课程和 2 个 graded assignments。吴恩达在 [2026-01-09 的 The Batch](https://www.deeplearning.ai/the-batch/issue-335/) 中介绍说，这门课面向从未写过代码的人，目标是在不到 30 分钟内，让学习者用 AI 把一个应用想法变成可运行的网页应用。

这个定位很有意思。它不是把非程序员训练成专业软件工程师，而是降低“我能不能做一个小工具”的心理门槛。课程中的典型练习包括做一个能在浏览器里运行、可以分享的互动生日祝福生成器，然后通过继续描述需求来迭代它。

如果说 AI Prompting for Everyone 教的是如何把 AI 当作通用助手，那么 Build with Andrew 教的是下一步：把想法压缩成足够清楚的规格，让 AI 直接产出一个可以运行的东西。

## 3. Jupyter AI: AI Coding in Notebooks：让 Notebook 变成协作界面

[Jupyter AI: AI Coding in Notebooks](https://www.deeplearning.ai/courses/jupyter-ai-coding-in-notebooks/) 是这组课里更偏开发和数据分析的一门短课。它由吴恩达和 Project Jupyter 联合创始人 Brian Granger 合讲。课程页显示它是 beginner short course，约 44 分钟，包含 5 个视频、3 个 code examples 和 1 个 graded assignment。

官方社区在 [2025-11-06 的新课公告](https://community.deeplearning.ai/t/new-course-enroll-in-jupyter-ai-ai-coding-in-notebooks/882862) 中说明，课程会教学习者把 Jupyter AI 当作 Notebook 里的编码伙伴：生成代码、解释代码、调试错误，把已有 cell 拖入对话提供上下文，以及围绕文件和 API 文档继续开发。

这门课的两个练习方向也很典型：

- 构建一个基于 Open Library API 的图书研究助手。
- 做一个股票数据分析工作流。

Notebook 的价值一直不只是“写代码”，而是把代码、数据、图表和解释放在同一个工作空间里。AI 进入 Notebook 后，真正改变的是上下文的组织方式：你不必把问题从 Notebook 搬去聊天窗口，再把答案搬回来；协作发生在工作现场。

## 4. Agentic AI：从单次回答走向可评估的工作流

[Agentic AI](https://www.deeplearning.ai/courses/agentic-ai/) 是这四门课中技术深度最高的一门。课程页显示它是 intermediate 级别，约 7 小时 45 分钟，包含 31 个视频、7 个 code examples 和 8 个 graded assignments。吴恩达在 [2025-10-08 的 The Batch](https://www.deeplearning.ai/the-batch/check-out-our-course-on-how-to-build-ai-agents/) 中称这门课会讲四个关键 agentic design patterns：

- Reflection：让系统检查和改进自己的输出。
- Tool use：让 LLM 驱动的应用调用数据库、API、搜索、代码执行等外部工具。
- Planning：把复杂任务拆成可执行步骤。
- Multi-agent collaboration：让多个专门化 agent 协作完成复杂任务。

我觉得这门课最重要的不是“多智能体”这个词，而是它强调 evals、error analysis 和 traces。很多 agent 项目卡住，不是因为模型不够新，而是因为团队不知道哪里坏了、该优化哪个组件、怎样判断一次修改是否真的变好。吴恩达把这部分放进课程里，说明 agentic AI 正在从 demo 文化走向工程文化。

## 这四门课连起来看

如果把它们按学习路线排列，我会这样理解：

1. 先学 [AI Prompting for Everyone](https://www.deeplearning.ai/courses/ai-prompting-for-everyone/)：建立现代 AI 工具的基本使用能力。
2. 再学 [Build with Andrew](https://www.deeplearning.ai/courses/build-with-andrew/)：把提示能力转成“构建小应用”的能力。
3. 如果经常做数据或 Python 分析，接着学 [Jupyter AI: AI Coding in Notebooks](https://www.deeplearning.ai/courses/jupyter-ai-coding-in-notebooks/)：把 AI 协作嵌进真实工作环境。
4. 如果想做更复杂的自动化和 AI 产品，再学 [Agentic AI](https://www.deeplearning.ai/courses/agentic-ai/)：理解工作流、工具调用、评估和部署。

这条路线也反映出吴恩达最近一年 AI 教育重心的变化：他不只是在教模型原理，而是在教“AI 能力的使用层”。从提示、构建、Notebook 协作，到 agentic workflow，这些课关注的是如何把强大的基础模型转化为个人生产力、团队工作流和可维护的软件系统。

对学习者来说，最实用的选择方式很简单：

- 完全没有技术背景：从 AI Prompting for Everyone 或 Build with Andrew 开始。
- 有一点 Python 或数据分析背景：直接看 Jupyter AI: AI Coding in Notebooks。
- 已经会写 Python，也想做智能体系统：看 Agentic AI。

过去几年，很多人学习 AI 的入口是“理解机器学习”。而这几门课传递出的新入口是：先学会把 AI 放进自己的工作里。这个变化很现实，也很吴恩达。

## Related URLs

- [AI Prompting for Everyone](https://www.deeplearning.ai/courses/ai-prompting-for-everyone/)
- [AI Prompting for Everyone announcement in The Batch](https://www.deeplearning.ai/the-batch/learn-foundational-prompting-techniques-in-ai-prompting-for-everyone/)
- [Build with Andrew](https://www.deeplearning.ai/courses/build-with-andrew/)
- [The Batch issue 335: Build with Andrew](https://www.deeplearning.ai/the-batch/issue-335/)
- [Jupyter AI: AI Coding in Notebooks](https://www.deeplearning.ai/courses/jupyter-ai-coding-in-notebooks/)
- [Jupyter AI course announcement in DeepLearning.AI Community](https://community.deeplearning.ai/t/new-course-enroll-in-jupyter-ai-ai-coding-in-notebooks/882862)
- [Agentic AI](https://www.deeplearning.ai/courses/agentic-ai/)
- [Agentic AI announcement in The Batch](https://www.deeplearning.ai/the-batch/check-out-our-course-on-how-to-build-ai-agents/)
