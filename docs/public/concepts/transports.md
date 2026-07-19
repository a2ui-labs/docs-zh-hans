# 传输层（消息传递）

传输层负责把 A2UI 消息从智能体送到客户端。A2UI 与具体传输方式无关，只要能够发送 JSON，就可以使用。

实际组件渲染由 [renderer](../reference/renderers.md) 完成，
[agents](../reference/agents.md) 负责生成 A2UI 消息，
而把消息从智能体传到客户端，就是 transport 的职责。

## 工作方式

```
Agent → Transport → Client Renderer
```

A2UI 定义了一串 JSON 消息序列。传输层负责把这串消息从智能体送到客户端。常见实现方式是使用 JSON Lines（JSONL）这样的流式格式，其中每一行都是一条独立的 A2UI 消息。

## 可用的传输方式

| 传输方式 | 状态 | 使用场景 |
|-----------|--------|----------|
| **A2A Protocol** | ✅ 稳定 | 多智能体系统、企业级网格 |
| **AG-UI** | ✅ 稳定 | 全栈 React、Vue、Angular 应用（CopilotKit） |
| **REST API** | 📋 规划中 | 简单 HTTP 接口 |
| **WebSockets** | 💡 提议中 | 实时双向通信 |
| **SSE (Server-Sent Events)** | 💡 提议中 | Web 流式传输 |

## A2A Protocol

[Agent2Agent (A2A) protocol](https://a2a-protocol.org) 提供安全、标准化的智能体通信能力。A2UI 通过 A2A 扩展，可以较容易地接入这个协议。

**优势：**

- 内建安全与认证能力
- 支持多种消息格式、认证机制与传输协议
- 职责边界清晰

如果你已经在使用 A2A，那么这部分接入几乎是自动完成的。

TODO：补充详细指南。

**另见：** [A2A 扩展规范](../specification/v0.8-a2a-extension.md)

## AG-UI

[AG-UI](https://ag-ui.com/) 会将 A2UI 消息转换为 AG-UI 事件，并自动处理传输与状态同步。它常用于全栈 React、Vue 和 Angular 应用。CopilotKit 是 AG-UI 的创建者，也是它的主要使用方。

**另见：** [在任意 Agent 框架中使用 A2UI（通过 AG-UI）](../guides/a2ui-with-any-agent-framework.md)：搭配你选择的 Agent 框架设置 CopilotKit，并启用 A2UI 渲染。

## 自定义传输方式

任何能够发送 JSON 的传输方式都可以使用：

**HTTP/REST：**

```javascript
// TODO: Add an example
```

**WebSockets：**

```javascript
// TODO: Add an example
```

**Server-Sent Events：**

```javascript
// TODO: Add an example
```

## 下一步

- **[A2A Protocol 文档](https://a2a-protocol.org)**：了解 A2A
- **[A2A 扩展规范](../specification/v0.8-a2a-extension.md)**：查看 A2UI 与 A2A 的结合细节
