# 什么是 A2UI？

**A2UI（Agent to UI）是一种面向智能体驱动界面的声明式 UI 协议。** AI 智能体可以生成丰富、可交互的 UI，并在不同平台（Web、移动端、桌面端）上以原生方式渲染，而无需执行任意代码。

## 问题是什么

**纯文本的智能体交互效率很低：**

```
User: "Book a table for 2 tomorrow at 7pm"
Agent: "Okay, for what day?"
User: "Tomorrow"
Agent: "What time?"
...
```

**更好的方式：** 由智能体生成带有日期选择器、时间选择器和提交按钮的表单，让用户直接与 UI 交互，而不是来回发送文本。

## 挑战在哪里

在多智能体系统中，智能体通常运行在远端（不同服务器、不同组织）。它们无法直接操作你的 UI，只能通过发送消息来表达。

**传统做法：** 在 iframe 中发送 HTML/JavaScript

- 负担重，视觉上割裂
- 安全性复杂
- 难以匹配应用现有样式

**真正需要的是：** 传递一种“像数据一样安全、像代码一样有表达力”的 UI。

## 解决方案

A2UI 通过 JSON 消息来描述 UI，并具备以下特性：

- 可由 LLM 作为结构化输出生成
- 可以通过任意传输层发送（A2A、AG UI、SSE、WebSockets）
- 由客户端使用自己的原生组件进行渲染

**结果是：** 客户端掌控安全和样式，而由智能体生成的 UI 仍然能呈现原生体验。

### 示例

=== "v0.8 (Stable)"

    ```jsonl
    {
      "surfaceUpdate": {
        "surfaceId": "booking",
        "components": [
          {
            "id": "title",
            "component": {
              "Text": {
                "text": { "literalString": "Book Your Table" },
                "usageHint": "h1"
              }
            }
          },
          {
            "id": "datetime",
            "component": {
              "DateTimeInput": {
                "value": { "path": "/booking/date" },
                "enableDate": true
              }
            }
          },
          {
            "id": "submit-text",
            "component": {
              "Text": {
                "text": { "literalString": "Confirm" }
              }
            }
          },
          {
            "id": "submit-btn",
            "component": {
              "Button": {
                "child": "submit-text",
                "action": { "name": "confirm_booking" }
              }
            }
          }
        ]
      }
    }
    {
      "dataModelUpdate": {
        "surfaceId": "booking",
        "contents": [
          {
            "key": "booking",
            "valueMap": [
              { "key": "date", "valueString": "2025-12-16T19:00:00Z" }
            ]
          }
        ]
      }
    }
    {
      "beginRendering": {
        "surfaceId": "booking",
        "root": "title"
      }
    }
    ```

=== "v0.9 (Draft)"

    ```jsonl
    {
      "version": "v0.9",
      "createSurface": {
        "surfaceId": "booking",
        "catalogId": "https://a2ui.org/specification/v0_9/basic_catalog.json"
      }
    }
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "booking",
        "components": [
          {
            "id": "title",
            "component": "Text",
            "text": "Book Your Table",
            "variant": "h1"
          },
          {
            "id": "datetime",
            "component": "DateTimeInput",
            "value": { "path": "/booking/date" },
            "enableDate": true
          },
          {
            "id": "submit-text",
            "component": "Text",
            "text": "Confirm"
          },
          {
            "id": "submit-btn",
            "component": "Button",
            "child": "submit-text",
            "variant": "primary",
            "action": {
              "event": { "name": "confirm_booking" }
            }
          }
        ]
      }
    }
    {
      "version": "v0.9",
      "updateDataModel": {
        "surfaceId": "booking",
        "path": "/booking",
        "value": {
          "date": "2025-12-16T19:00:00Z"
        }
      }
    }
    ```

    v0.9 的关键差异包括：`createSurface` 取代 `beginRendering`；组件采用更扁平的结构，例如使用 `"component": "Text"` 而不是嵌套对象；所有消息都包含 `version` 字段。

客户端会把这些消息渲染为原生组件（Angular、Flutter、React 等）。

## 核心价值

**1. 安全性：** 这是声明式数据，而不是代码。智能体只能从客户端受信任的 catalog 中请求组件，没有执行代码的风险。

**2. 原生体验：** 不依赖 iframe。客户端使用自己的 UI 框架渲染，天然继承应用的样式、可访问性和性能特征。

**3. 可移植性：** 一份智能体响应可在多端运行。同一份 JSON 可以渲染到 Web（Lit / Angular / React）、移动端（Flutter / SwiftUI / Jetpack Compose）和桌面端。

## 设计原则

**1. 对 LLM 友好：** 使用带 ID 引用的扁平组件列表，便于增量生成、纠错和流式输出。

**2. 与框架无关：** 智能体发送抽象组件树，客户端将其映射到原生控件（Web / 移动端 / 桌面端）。

**3. 关注点分离：** 拆分为三层——UI 结构、应用状态、客户端渲染。这样可以支持数据绑定、响应式更新以及更清晰的架构。

## A2UI 不是什么

- 它不是一个框架（而是一种协议）
- 它不是 HTML 的替代品（它面向智能体生成 UI，而不是静态网站）
- 它不是完整的样式系统（样式控制权在客户端，服务端样式支持有限）
- 它不限于 Web（同样适用于移动端和桌面端）

## 关键概念

- **Surface**：组件承载画布（如对话框、侧边栏、主视图）
- **Component**：UI 元素（Button、TextField、Card 等）
- **Data Model**：应用状态，组件会绑定到它
- **Catalog**：可用组件类型集合
- **Message**：JSON 对象（如 `surfaceUpdate`、`dataModelUpdate`、`beginRendering` 等）

若想比较相似项目，请参阅 [Agent UI Ecosystem](agent-ui-ecosystem.md)。
