---
title: "Client State Machine Design：从 Codex 事件流推导界面状态"
description: "讨论 Codex Client 侧状态机的职责、状态划分、事件驱动转移，以及对应的 Python 实现方式。"
pubDatetime: 2026-06-16T05:25:21Z
tags: [Codex, State Machine, Python]
---

## 背景

在前面的学习中，我们已经先翻译了 Codex Client 和 Agent 之间通信所需的核心数据结构。

大致模型是：

```text
Client -> Agent: Submission
Agent -> Client: Event
```

这一步解决的是“Client 和 Agent 之间传递什么数据”。

接下来的问题是：Client 收到 Agent 发来的事件后，应该如何理解当前会话状态？

这就是 Client 侧状态机要解决的问题。

## Client 状态机和 Agent 状态机的区别

Codex 中可以区分两类状态：

| 类型 | 关注点 | 作用 |
| --- | --- | --- |
| Agent internal state | Codex 实际正在做什么 | 决定下一步执行逻辑 |
| Client visible state | 用户应该看到什么、能做什么 | 驱动界面展示和交互 |

Agent 内部状态更复杂，可能包含：

- 当前 turn 生命周期
- 会话历史
- 模型请求状态
- pending tool calls
- pending approvals
- 命令执行状态
- patch 应用状态
- 错误和重试处理
- 上下文压缩状态

Client 侧状态机要简单得多。

它不是为了替代 Agent 的内部状态，而是为了让界面知道：

- 是否应该显示 spinner
- 是否应该显示审批提示
- 是否可以继续输入
- 是否应该展示最终回答
- 是否应该展示错误

可以概括为：

```text
Agent state = Codex 实际正在做什么
Client state = 用户应该看到什么、可以做什么
```

## 状态集合

第一版 Client 状态机可以保持很小，只包含这些状态：

```text
PENDING
RUNNING
WAITING_FOR_APPROVAL
COMPLETED
INTERRUPTED
ERRORED
```

含义如下：

| 状态 | 含义 |
| --- | --- |
| `PENDING` | 还没有 active turn，或者还没有收到 Agent 事件 |
| `RUNNING` | Agent 正在处理当前 turn |
| `WAITING_FOR_APPROVAL` | Agent 请求用户审批某个动作，例如 shell 命令 |
| `COMPLETED` | 当前 turn 正常完成 |
| `INTERRUPTED` | 当前 turn 被中断 |
| `ERRORED` | 当前 turn 失败 |

这不是完整 Codex 状态模型，而是 Client 可见状态的最小版本。

## 事件如何驱动状态变化

Client 状态主要由 Agent 发出的 `EventMsg` 推导出来。

核心转移规则是：

| 事件 | 状态变化 |
| --- | --- |
| `TurnStarted` | `RUNNING` |
| `AgentReasoning` | 状态不变，追加 reasoning 文本 |
| `AgentMessage` | 状态不变，更新最后一条 Agent 消息 |
| `ExecApprovalRequest` | `WAITING_FOR_APPROVAL` |
| `ExecCommandBegin` | `RUNNING` |
| `ExecCommandOutputDelta` | 状态不变 |
| `ExecCommandEnd` | 状态不变 |
| `TurnComplete` | `COMPLETED` |
| `Error` | `ERRORED` |

可以用一个基础流程理解：

```text
PENDING
  -> TurnStarted
RUNNING
  -> ExecApprovalRequest
WAITING_FOR_APPROVAL
  -> ExecCommandBegin
RUNNING
  -> TurnComplete
COMPLETED
```

## 用户动作和 Agent 事件的关系

Client 状态机主要根据 Agent 事件更新，但用户动作也会参与流程。

例如审批流程：

```text
Agent emits ExecApprovalRequest
Client state -> WAITING_FOR_APPROVAL

User approves
Client sends ExecApproval

Agent emits ExecCommandBegin
Client state -> RUNNING
```

注意：用户点击批准这一步本身不代表命令已经开始执行。

Client 更稳妥的做法是等 Agent 发出 `ExecCommandBegin` 后，再把状态切回 `RUNNING`。

这样 Client 展示的是 Agent 确认后的状态，而不是单纯基于本地 UI 操作猜测。

## Python 实现思路

在 Python 翻译中，可以把状态机写成一个很小的 reducer。

核心公式是：

```text
current_state + incoming_event -> next_state
```

也就是：

```python
def apply_event(state: SessionState, event: Event) -> SessionState:
    ...
```

为了让逻辑更容易测试，状态对象可以设计成不可变 dataclass：

```python
@dataclass(frozen=True)
class SessionState:
    status: SessionStatus = SessionStatus.PENDING
    current_turn_id: str | None = None
    last_agent_message: str | None = None
    reasoning: tuple[str, ...] = ()
    pending_approval_call_id: str | None = None
    error_message: str | None = None
```

状态枚举可以是：

```python
class SessionStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    WAITING_FOR_APPROVAL = "waiting_for_approval"
    COMPLETED = "completed"
    INTERRUPTED = "interrupted"
    ERRORED = "errored"
```

`apply_event()` 不直接修改原对象，而是返回一个新状态。

这让状态变化更接近下面这个模型：

```text
old state
  + event
  = new state
```

## 事件回放

除了单个事件转移，还需要一个事件回放函数：

```python
def replay_events(events: Iterable[Event]) -> SessionState:
    state = SessionState()
    for event in events:
        state = apply_event(state, event)
    return state
```

它的作用是：给定一串 Agent 事件，重新推导当前 Client 可见状态。

这对理解 Client 行为很重要，因为 Client 看到的界面状态，本质上可以从事件流中推导出来。

## 应该测试哪些流程

第一版状态机只需要覆盖三个流程。

### 1. 正常 turn

事件序列：

```text
TurnStarted
AgentReasoning
AgentMessage
TurnComplete
```

期望结果：

```text
status = COMPLETED
last_agent_message = 最后一条 Agent 消息
reasoning = reasoning 文本列表
```

### 2. 审批 turn

事件序列：

```text
TurnStarted
ExecApprovalRequest
ExecCommandBegin
ExecCommandEnd
TurnComplete
```

期望中间状态：

```text
ExecApprovalRequest 后 -> WAITING_FOR_APPROVAL
ExecCommandBegin 后 -> RUNNING
TurnComplete 后 -> COMPLETED
```

### 3. 错误 turn

事件序列：

```text
TurnStarted
Error
```

期望结果：

```text
status = ERRORED
error_message = 错误信息
```

## 当前实现边界

这一版状态机只处理 Client 可见状态，不处理 Agent 的完整内部执行状态。

它不会实现：

- 模型请求调度
- 工具选择
- 真正的 shell 执行
- patch 应用
- 复杂权限策略
- 上下文压缩逻辑

这些属于 Agent 内部逻辑或后续阶段。

当前阶段只回答一个问题：

```text
Client 收到这些事件后，应该认为当前会话处于什么状态？
```

## 小结

Client 状态机是连接协议数据结构和人机交互的中间层。

协议数据结构告诉我们：

```text
Agent 发来了什么事件
```

Client 状态机进一步回答：

```text
这个事件对界面状态意味着什么
```

第一版实现保持足够小：状态枚举、不可变 `SessionState`、`apply_event()` 和 `replay_events()`。

这让我们从静态数据结构，进入了动态事件流的理解。🙂
