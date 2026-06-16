---
title: "Codex 中的 Client、Agent 和它们之间的数据结构"
description: "从技术角度解释 Codex Client、Agent 的职责边界，以及它们之间通过 Submission、Op、Event、EventMsg 等结构进行通信的方式。"
pubDatetime: 2026-06-16T02:39:40Z
tags: [Codex, Agent, 数据结构]
---

## 核心问题

在翻译 Codex 源码之前，需要先理解一个关键设计：Codex 把展示层和核心执行逻辑分开。

它们之间通过一组协议数据结构通信。

可以先用这个模型理解：

```text
Client -> Agent: Submission
Agent -> Client: Event
```

Client 发送请求，Agent 返回事件。

这组结构不是 OpenAI API 的请求结构，而是 Codex 内部不同层之间的通信协议。

## Client 是什么

在这个上下文里，Client 指的是 Codex 的前端、调用方或展示层。

它可以是：

- 命令行界面中的 TUI
- VS Code Extension
- macOS App
- app-server
- 测试工具或自动化调用方

Client 主要负责和用户交互。

它会处理这些事情：

- 收集用户输入
- 把用户输入发送给 Agent
- 渲染 Agent 消息
- 展示命令输出
- 展示审批请求
- 把用户的审批结果发回给 Agent
- 展示错误、进度、计划和 diff

所以，Client 更接近“表现层”和“交互层”。

## Agent 是什么

Agent 指的是 Codex 的核心执行逻辑。

它不是单纯的模型，也不是 UI。它是负责组织一次 Codex 会话的运行时逻辑。

Agent 主要负责：

- 接收用户输入
- 维护 session 和 turn 状态
- 构造模型请求
- 调用模型
- 决定何时调用工具
- 协调 shell 命令执行
- 协调 patch 应用
- 发起审批请求
- 处理上下文压缩
- 把执行过程中的事件流式发回 Client

因此，可以把 Agent 理解为 Codex 的逻辑核心。

## 为什么要分成 Client 和 Agent

这种设计可以把展示层和逻辑层分离。

粗略结构如下：

```text
CLI / TUI / VS Code / macOS App / Server
                  |
                  | Submission
                  v
             Codex Agent/Core
                  |
                  | Event
                  v
CLI / TUI / VS Code / macOS App / Server
```

这样做的好处是：

- 不同客户端可以共享同一套核心逻辑
- CLI、编辑器扩展、桌面应用不需要重复实现 Agent 行为
- UI 只需要关注渲染和交互
- 核心逻辑可以通过稳定协议对外输出状态
- 测试可以绕过真实 UI，直接驱动 Agent
- 事件流天然适合展示进度和中间状态

## Submission：Client 发给 Agent 的请求

`Submission` 表示 Client 发给 Agent 的一条请求。

它的核心字段是：

```text
id
op
client_user_message_id
trace
```

其中最重要的是：

- `id`：请求 ID，用来关联后续事件
- `op`：具体操作

可以这样理解：

```text
Client 说：这是请求 abc123，请处理这个用户输入。
```

## Op：请求的具体类型

`Op` 描述 Client 希望 Agent 做什么。

第一批翻译只关注最核心的几类：

```text
UserInput
Interrupt
ExecApproval
PatchApproval
```

含义分别是：

| Op | 含义 |
| --- | --- |
| `UserInput` | 用户发送了一条输入 |
| `Interrupt` | 用户希望中断当前任务 |
| `ExecApproval` | 用户批准或拒绝某个 shell 命令 |
| `PatchApproval` | 用户批准或拒绝某个代码 patch |

这些足够描述一个基础 Codex turn 的入口和审批回路。

## Event：Agent 发给 Client 的事件

`Event` 表示 Agent 发回给 Client 的一条事件。

它的核心字段是：

```text
id
msg
```

其中：

- `id` 对应某个 `Submission.id`
- `msg` 是具体事件内容

可以这样理解：

```text
Agent 说：对于请求 abc123，现在发生了这个事件。
```

## EventMsg：Agent 发出的具体事件

`EventMsg` 描述 Agent 内部发生了什么。

第一批翻译只关注正常 turn 中最常见的事件：

```text
TurnStarted
UserMessage
AgentMessage
AgentReasoning
ExecApprovalRequest
ExecCommandBegin
ExecCommandOutputDelta
ExecCommandEnd
TurnComplete
Error
```

这些事件可以组成一个基础流程：

```text
TurnStarted
UserMessage
AgentReasoning
AgentMessage
ExecApprovalRequest
ExecCommandBegin
ExecCommandOutputDelta
ExecCommandEnd
TurnComplete
```

它们让 Client 能够知道 Agent 当前在做什么，并据此更新界面。

## UserInput：用户输入的结构

`UserInput` 表示用户提供给 Codex 的输入。

第一批翻译包含这些类型：

```text
Text
Image
LocalImage
Skill
Mention
```

其中最重要的是 `Text`。

文本输入还可以包含 `TextElement`。`TextElement` 用来标记文本中的特殊范围，例如某个占位符或结构化提及。

## ByteRange 和 TextElement

`ByteRange` 表示 UTF-8 文本中的字节范围：

```text
start
end
```

`TextElement` 包含一个 `ByteRange`，并且可以带一个可选 placeholder。

这点很重要，因为 Codex 可能会把多个文本片段拼接成一条完整消息。拼接之后，原来每个文本片段里的 byte range 需要重新偏移，才能继续指向正确的位置。

例如：

```text
片段 1: "hi "
片段 2: "世界"
```

如果第二个片段里有一个范围 `[0, 6]`，拼接后它需要变成 `[3, 9]`，因为前面的 `"hi "` 占了 3 个 UTF-8 字节。

## TurnItem：一次 turn 中发生的结构化事项

`TurnItem` 是一次 turn 中发生的结构化记录。

第一批翻译只处理：

```text
UserMessageItem
AgentMessageItem
ReasoningItem
```

它们分别表示：

| TurnItem | 含义 |
| --- | --- |
| `UserMessageItem` | 用户发出的消息 |
| `AgentMessageItem` | Agent 输出的消息 |
| `ReasoningItem` | Agent 的 reasoning 摘要 |

这些结构可以转换成旧式事件。

例如：

- `UserMessageItem` 转成 `UserMessageEvent`
- `AgentMessageItem` 转成 `AgentMessageEvent`
- `ReasoningItem` 转成 `AgentReasoningEvent`

这说明 Codex 同时存在两种表达：

- 更结构化的 turn item
- 面向兼容和展示的 legacy event

## 一个基础通信示例

可以用下面这个流程理解 Client 和 Agent 的交互：

```text
Client -> Agent:
Submission(id="abc", op=UserInput(...))

Agent -> Client:
Event(id="abc", msg=TurnStarted(...))
Event(id="abc", msg=UserMessage(...))
Event(id="abc", msg=AgentReasoning(...))
Event(id="abc", msg=AgentMessage(...))
Event(id="abc", msg=TurnComplete(...))
```

如果中间需要执行命令，可能会出现：

```text
Agent -> Client:
Event(id="abc", msg=ExecApprovalRequest(...))

Client -> Agent:
Submission(id="def", op=ExecApproval(...))

Agent -> Client:
Event(id="abc", msg=ExecCommandBegin(...))
Event(id="abc", msg=ExecCommandOutputDelta(...))
Event(id="abc", msg=ExecCommandEnd(...))
```

注意：审批结果是新的 `Submission`，但它会影响之前正在进行的 turn。

## 当前 Python 翻译的边界

当前只翻译了最小核心子集，目标是描述一个普通 Codex turn。

已经覆盖：

- Client 到 Agent 的请求结构
- Agent 到 Client 的事件结构
- 用户输入结构
- 文本 byte range 处理
- turn item 到 legacy event 的转换

暂时没有展开：

- MCP
- dynamic tools
- hooks
- realtime conversation
- collaboration mode
- review mode
- context compaction
- guardian/security events

这些都可以以后逐步加入。

## 下一步：状态跟踪

完成数据结构之后，下一个自然问题是：

```text
给定一串 Agent 事件，Client 应该认为当前会话处于什么状态？
```

例如：

- `TurnStarted` 之后进入 running
- `ExecApprovalRequest` 之后进入 waiting_for_approval
- `ExecCommandBegin` 之后回到 running
- `TurnComplete` 之后进入 completed
- `Error` 之后进入 errored

这就是下一步要讨论的 State Tracking。它会把这些静态数据结构推进到动态会话逻辑。🔎
