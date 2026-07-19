# A2UI 与其他方案相比如何？

Agentic UI 这一领域正在快速演进。下面说明 A2UI 与其他主流方案之间的关系。

## 一览表

| | **A2UI** | **MCP Apps** | **AG-UI** |
|---|---|---|---|
| **方式** | 声明式组件蓝图 | 通过 `ui://` URI 提供预构建 HTML | 连接后端与前端的高带宽协议 |
| **渲染** | 原生组件（Angular、Flutter、Lit 等） | 沙箱 `iframe` | 由开发者定义（任意框架） |
| **样式** | 宿主应用控制，继承设计系统 | 完全隔离，由远端服务控制外观 | 由开发者控制，是宿主应用的一部分 |
| **安全性** | 声明式数据，不执行代码 | 依赖沙箱 iframe 隔离 | 在你自己的应用中运行受信任代码 |
| **多智能体** | ✅ 可跨信任边界 | ✅ 支持多个 MCP 服务器 | ⚠️ 主要面向单智能体 |
| **跨平台** | ✅ Web、移动端、桌面端、原生 | ⚠️ 偏 Web（iframe） | ✅ 协议本身与框架无关 |
| **LLM 生成** | ✅ 为流式输出而设计 | ❌ 由服务端预先构建 | ✅ 可通过 A2UI 集成 |
| **规范** | 开放协议（Apache 2.0） | [MCP extension](https://modelcontextprotocol.io/docs/extensions/apps)（SEP-1865） | 开源（由 CopilotKit 维护） |

## A2UI vs MCP Apps

[MCP Apps](https://blog.modelcontextprotocol.io/posts/2025-11-21-mcp-apps/) 把 UI 视为一种**资源**。服务器通过 `ui://` URI 提供预构建 HTML，并在沙箱 iframe 中渲染。远端集成方掌控全部内容与外观，配置通常通过 tool calling 完成。A2UI 采取的是**声明式 UI** 路线。智能体发送组件蓝图，但宿主应用掌控样式、主题，以及组件如何被配置和渲染。若服务端应该拥有完整 UI 体验，选择 MCP Apps；若你希望得到动态、跨平台、并且能自然融入应用的 UI，选择 A2UI。

## A2UI vs AG-UI / CopilotKit

[AG-UI](https://ag-ui.com/) 是一种**传输协议**，用于把智能体后端与前端连接起来，并实现实时状态同步。A2UI 则是一种**UI 格式**，也就是描述“渲染什么”的载荷。两者是互补关系：AG-UI 负责“管道”，A2UI 负责“内容”。AG-UI 由 [CopilotKit](https://copilotkit.ai) 团队开发，而他们也参与了 [A2UI Composer](../composer.md) 的建设。AG-UI 对 A2UI 提供开箱即用的兼容性。

## A2UI vs ChatKit（OpenAI）

[ChatKit](https://platform.openai.com/docs/guides/chatkit) 在 OpenAI 生态中提供了一体化体验。A2UI 与 ChatKit 在设计理念上有一些相似之处，例如都定义了一组基础组件，并采用可配置、声明式的抽象层。A2UI 则是**平台无关**的，更适合那些要在 Web、移动端与桌面端打造自有 agentic surface 的开发者，或者需要让智能体跨信任边界渲染 UI 的多智能体系统。

## 如何组合使用

这些方案是互补关系，而不是彼此竞争：

- **A2UI + AG-UI** —— 由 AG-UI 负责传输，由 A2UI 负责生成式 UI 格式
- **A2UI + A2A** —— 在多智能体系统中，通过 [A2A protocol](../concepts/transports.md) 发送 A2UI 消息
- **A2UI + MCP** —— 即将到来的桥接方案可让 MCP 服务器在提供 HTML 资源的同时，也提供 A2UI 蓝图

## 延伸阅读

- [什么是 A2UI？](what-is-a2ui.md) —— 协议概览
- [传输层](../concepts/transports.md) —— A2UI 消息如何在智能体与客户端之间流转
- [A2UI 在哪里被使用？](../ecosystem/a2ui-in-the-world.md) —— 案例研究与采用方
