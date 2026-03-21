# 我可以如何使用 A2UI？

根据你的角色和使用场景，选择最适合的集成路径。

## 三条路径

### 路径 1：构建宿主应用（前端）

把 A2UI 渲染能力集成到你现有的应用中，或者直接构建一个由智能体驱动的新前端。

**选择一个渲染器：**

- **Web：** Lit、Angular、React
- **移动端 / 桌面端：** Flutter GenUI SDK

**快速安装：**

如果使用 Angular：

```bash
npm install @a2ui/angular @a2ui/web-lib
```

如果使用 React：

```bash
npm install @a2ui/react @a2ui/web-lib
```

把智能体消息源（SSE、WebSockets 或 A2A）接进来，再根据你的品牌风格定制样式。

**下一步：** [客户端接入指南](../guides/client-setup.md) | [主题与样式](../guides/theming.md)

---

### 路径 2：构建智能体（后端）

创建一个能够为任意兼容客户端生成 A2UI 响应的智能体。

**选择你的框架：**

- **Python：** Google ADK、LangChain、自定义方案
- **Node.js：** A2A SDK、Vercel AI SDK、自定义方案

在 LLM 提示词中加入 A2UI schema，生成 JSONL 消息，并通过 SSE、WebSockets 或 A2A 流式发送给客户端。

**下一步：** [智能体开发指南](../guides/agent-development.md)

---

### 路径 3：使用现成框架

通过内建支持 A2UI 的框架来使用它：

- **[AG UI / CopilotKit](https://ag-ui.com/)** - 带有 A2UI 渲染能力的全栈 React 框架
- **[Flutter GenUI SDK](https://docs.flutter.dev/ai/genui)** - 跨平台生成式 UI 方案（底层使用 A2UI）

**下一步：** [智能体 UI 生态](agent-ui-ecosystem.md) | [A2UI 在哪里被使用？](../ecosystem/a2ui-in-the-world.md)
