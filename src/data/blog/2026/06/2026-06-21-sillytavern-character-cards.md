---
title: "SillyTavern 角色卡格式简明指南"
description: "介绍 SillyTavern 角色卡的用途、常见文件形态、V1 与 V2 格式差异，以及官方文档与生态规范的关系。"
pubDatetime: 2026-06-21T06:30:59Z
tags: [SillyTavern, Character Card, AI Roleplay]
---

## SillyTavern 角色卡是什么？ 🃏

在 SillyTavern 里，**角色卡**是一份可导入、可导出的角色资料。它的作用是把一个角色的核心设定交给前端和模型使用，让模型在对话时知道应该扮演谁、如何说话、处在什么场景里。

一张角色卡通常会包含：角色名、角色描述、性格、场景、第一条消息、示例对话、作者备注、标签，以及一些扩展字段。部分角色卡还会包含角色专属的 lorebook，也就是与角色绑定的世界信息或关键词触发资料。

## 常见文件形态

SillyTavern 角色卡常见有两种形态：

| 形态 | 说明 |
| --- | --- |
| PNG / WebP 图片 | 看起来是一张图片，但角色数据嵌入在图片元数据里。这是社区里最常见的分享方式。 |
| JSON 文件 | 直接以 JSON 对象保存角色数据，适合编辑、生成和程序处理。 |

换句话说，图片版角色卡并不只是头像；真正重要的是图片元数据中保存的 JSON 内容。

## V1：较早的扁平格式

较早的 V1 格式比较简单，字段直接放在 JSON 顶层：

```json
{
  "name": "角色名",
  "description": "角色描述 / 外观 / 背景",
  "personality": "性格摘要",
  "scenario": "当前场景",
  "first_mes": "角色的第一条消息",
  "mes_example": "示例对话"
}
```

这些字段构成了角色卡的基础。即使在 V2 中，它们仍然是核心内容，只是位置被移动到了 `data` 对象里。

## V2：现代角色卡结构 ✨

现在更常见的是 Character Card V2。它会在顶层声明规格信息，并把角色数据放进 `data`：

```json
{
  "spec": "chara_card_v2",
  "spec_version": "2.0",
  "data": {
    "name": "角色名",
    "description": "角色描述 / 外观 / 背景",
    "personality": "性格摘要",
    "scenario": "当前场景",
    "first_mes": "角色的第一条消息",
    "mes_example": "示例对话",

    "creator_notes": "作者备注",
    "system_prompt": "可选的系统提示词覆盖",
    "post_history_instructions": "对话历史之后追加的指令",
    "alternate_greetings": ["另一个开场白"],

    "tags": ["tag1", "tag2"],
    "creator": "作者名",
    "character_version": "1.0",
    "extensions": {}
  }
}
```

V2 相比 V1 的主要变化是：

- 顶层增加 `spec` 和 `spec_version`，用于明确这是 `chara_card_v2`。
- 原来的角色字段被放进 `data`，减少被旧编辑器误读或破坏的风险。
- 增加作者备注、系统提示词、对话历史后指令、备用开场白、标签、作者、版本号和扩展字段。
- `extensions` 用来保存应用自定义数据，默认应为空对象 `{}`。

## 角色专属 lorebook

V2 还允许角色卡携带 `character_book`，也就是角色专属 lorebook。它可以保存关键词触发条目，在对话中根据关键词把相关资料注入提示词。

一个简化结构如下：

```json
{
  "data": {
    "character_book": {
      "name": "示例 lorebook",
      "entries": [
        {
          "keys": ["关键词"],
          "content": "触发后注入的设定内容",
          "enabled": true,
          "insertion_order": 0,
          "extensions": {}
        }
      ],
      "extensions": {}
    }
  }
}
```

这类资料适合保存世界观、术语、人物关系或只有在特定关键词出现时才需要加入上下文的信息。

## SillyTavern 文档与 V2 规范的关系 🔗

这里有一个容易混淆的点：**SillyTavern 官方文档会说明角色编辑器里的字段和使用方式，但低层的 `chara_card_v2` 格式规范并不是托管在 SillyTavern 文档站本身。**

可以参考两个入口：

- SillyTavern 官方角色设计文档：<https://docs.sillytavern.app/usage/core-concepts/characterdesign/>
- Character Card V2 生态规范：<https://github.com/malfoyslastname/character-card-spec-v2/blob/main/spec_v2.md>

SillyTavern 文档适合理解每个字段在界面和提示词构建中的作用；V2 规范适合理解 JSON 的精确结构、字段要求和跨前端兼容性。

## 小结

SillyTavern 角色卡可以理解为“角色设定 + 对话启动信息 + 可选元数据”的可移植包。图片格式方便分享，JSON 格式方便编辑和自动化处理。对于新建或解析角色卡，优先理解 Character Card V2 会更实用，因为它覆盖了现代角色卡常见的字段、扩展数据和角色专属 lorebook。
