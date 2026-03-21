# A2UI 的实际应用

A2UI 正在被 Google 及其合作组织的团队采用，用来构建下一代由代理驱动的应用。下面是一些 A2UI 在真实世界中的落地案例。

## 生产部署

### Google Opal：面向所有人的 AI Mini-App

[Opal](http://opal.google) 让成千上万的人能够用自然语言构建、编辑和分享 AI mini-app，而且不需要写代码。

**Opal 如何使用 A2UI：**

Google 的 Opal 团队从一开始就是 A2UI 的 **核心贡献者**。他们使用 A2UI 来驱动动态、生成式 UI 系统，从而让 Opal 的 AI mini-app 成为可能。

- **快速原型验证**：快速构建并测试新的 UI 模式
- **用户生成应用**：任何人都可以创建带自定义 UI 的应用
- **动态界面**：界面会自动适配不同的使用场景

> "A2UI 是我们工作的基础。它让 AI 可以用新颖的方式驱动用户体验，而不必受限于固定的前端。它的声明式特性和对安全性的关注，让我们能够快速且安全地进行实验。"
>
> **— Dimitri Glazkov**, Opal 团队首席工程师

**了解更多：** [opal.google](http://opal.google)

---

### Gemini Enterprise：面向业务的自定义代理

Gemini Enterprise 让企业能够构建强大的自定义 AI 代理，并针对各自的工作流与数据进行定制。

**Gemini Enterprise 如何使用 A2UI：**

A2UI 正被集成进来，使企业代理能够在业务应用中渲染 **丰富、可交互的 UI**，不再只是简单的文本回复，而是能够引导员工完成复杂工作流。

- **数据录入表单**：由 AI 生成的结构化数据采集表单
- **审批仪表盘**：用于审核和批准流程的动态 UI
- **工作流自动化**：面向复杂任务的分步式界面
- **企业定制 UI**：为行业特定需求量身定做的界面

> "我们的客户需要代理做的不只是回答问题，他们需要代理引导员工完成复杂工作流。A2UI 将让基于 Gemini Enterprise 开发的开发者，让代理生成完成任何任务所需的动态定制 UI，从数据录入表单到审批仪表盘都可以，大幅加速工作流自动化。"
>
> **— Fred Jabbour**, Gemini Enterprise 产品经理

**了解更多：** [Gemini Enterprise](https://cloud.google.com/gemini)

---

### Flutter GenUI SDK：面向移动端的生成式 UI

[Flutter GenUI SDK](https://docs.flutter.dev/ai/genui) 为 Flutter 应用带来了动态的、AI 生成的 UI，覆盖移动端、桌面端和 Web。

**GenUI 如何使用 A2UI：**

GenUI SDK 使用 **A2UI 作为底层协议** 来在服务端代理与 Flutter 应用之间通信。你使用 GenUI 时，实际上是在底层使用 A2UI。

- **跨平台支持**：同一个代理可在 iOS、Android、Web、桌面端运行
- **原生性能**：Flutter 组件在各平台上以原生方式渲染
- **品牌一致性**：UI 会匹配你的应用设计系统
- **服务端驱动 UI**：由代理控制展示内容，无需应用更新

> "我们的开发者选择 Flutter，是因为它让他们能够快速创建富有表现力、品牌感强且高度定制的设计系统，并且在每个平台上都能保持出色体验。A2UI 非常适合 Flutter 的 GenUI SDK，因为它能确保每个用户在每个平台上都获得高质量、原生感很强的体验。"
>
> **— Vijay Menon**, Dart & Flutter 工程总监

**试试：**

- [GenUI 文档](https://docs.flutter.dev/ai/genui)
- [入门视频](https://www.youtube.com/watch?v=nWr6eZKM6no)
- [Verdure 示例](https://github.com/flutter/genui/tree/main/examples/verdure)（客户端-服务端 A2UI 示例）

---

### Google ADK：Agent Development Kit

[Agent Development Kit](https://google.github.io/adk-docs/)（ADK）是 Google 面向构建与部署 AI 代理的开源框架。其内置开发者 UI [ADK Web](https://github.com/google/adk-web) 支持原生 A2UI 渲染。

**ADK 如何使用 A2UI：**

ADK 集成了 A2UI v0.8 标准目录，可以自动把符合规范的代理内容直接渲染为聊天界面中的原生 UI 组件。ADK 还负责 A2UI ↔ A2A 消息转换，因此用 ADK 构建的代理可以向任意支持 A2UI 的客户端发送富 UI。

- **内置渲染**：ADK Web 在开发者 UI 中原生渲染 A2UI 组件
- **A2A 集成**：A2UI 消息会在 A2A DataPart 元数据与 ADK 事件之间转换
- **Agent SDK**：[A2UI Python agent SDK](https://github.com/google/A2UI/tree/main/agent_sdks/python) 为从代理生成 A2UI 提供了 ADK 扩展

**试试：**
- [ADK 文档](https://google.github.io/adk-docs/)
- [ADK Web](https://github.com/google/adk-web)（支持 A2UI 的开发者 UI）
- [Agent Development Guide](../guides/agent-development.md)（用 ADK 构建 A2UI 代理）

---

## 合作伙伴集成

### AG UI / CopilotKit：全栈 Agentic 框架

[AG UI](https://ag-ui.com/) 和 [CopilotKit](https://www.copilotkit.ai/) 提供了一套完整的 agentic 应用构建框架，并且 **从第一天起就兼容 A2UI**。

**它们如何协同：**

AG UI 擅长在自定义前端和其专属代理之间建立高带宽连接。加入 A2UI 支持后，开发者可以两者兼得：

- **状态同步**：AG UI 负责应用状态和聊天历史
- **A2UI 渲染**：渲染来自第三方代理的动态 UI
- **多代理支持**：协调来自多个代理的 UI
- **React 集成**：与 React 应用无缝集成

> "AG UI 擅长在自建前端和其专属代理之间建立高带宽连接。加入 A2UI 支持后，我们让开发者同时拥有两者的优势。他们现在可以构建内容丰富、状态同步的应用，也可以通过 A2UI 渲染来自第三方代理的动态 UI。这非常适合多代理世界。"
>
> **— Atai Barkai**, CopilotKit 与 AG UI 创始人

**了解更多：**

- [AG UI](https://ag-ui.com/)
- [CopilotKit](https://www.copilotkit.ai/)

---

### Google 的 AI 驱动产品

随着 Google 在全公司范围内采用 AI，A2UI 提供了一种 **让 AI 代理交换用户界面而不仅仅是文本的标准化方式**。

**内部代理采用：**

- **多代理工作流**：多个代理共同贡献同一个 surface
- **远程代理支持**：运行在不同服务上的代理也可以提供 UI
- **标准化**：跨团队通用协议可以减少集成开销
- **对外暴露**：内部代理可以轻松对外开放（例如 Gemini Enterprise）

> "就像 A2A 让任何代理都可以跨平台与另一个代理对话一样，A2UI 通过 orchestrator 标准化了用户界面层，并支持远程代理场景。这对内部团队非常强大，让他们能够快速开发出富 UI 才是常态而非例外的代理。随着 Google 进一步推进生成式 UI，A2UI 为任何客户端上的服务端驱动 UI 提供了理想平台。"
>
> **— James Wren**, AI Powered Google 高级首席工程师

---

## 社区项目

A2UI 社区正在打造一些很有意思的项目：

### 开源示例

- **Restaurant Finder** ([samples/agent/adk/restaurant_finder](https://github.com/google/A2UI/tree/main/samples/agent/adk/restaurant_finder))
    - 带动态表单的桌位预订
    - Gemini 驱动的代理
    - 提供完整源代码

- **Contact Lookup** ([samples/agent/adk/contact_lookup](https://github.com/google/A2UI/tree/main/samples/agent/adk/contact_lookup))
    - 带结果列表的搜索界面
    - A2A 代理示例
    - 演示数据绑定

- **Component Gallery** ([samples/client/angular - gallery mode](https://github.com/google/A2UI/tree/main/samples/client/angular))
    - 所有组件的交互式展示
    - 带代码的实时示例
    - 非常适合学习

### 第三方集成

- **[json-render](https://json-render.dev/docs/a2ui)** —— Vercel 的 React 库，可通过 Zod schema 渲染 A2UI 组件目录。参见 [json-render vs. A2UI comparison](https://dipjyotimetia.medium.com/vercels-json-render-vs-google-s-a2ui-the-head-to-head-6f213cf1a23b)。
- **[OpenClaw Canvas](https://docs.openclaw.ai/platforms/mac/canvas)** —— OpenClaw 使用 A2UI 通过其 canvas 系统在连接设备上渲染代理生成的 UI。参见 [architecture overview](https://ppaolo.substack.com/p/openclaw-system-architecture-overview)。

### 社区贡献

你已经用 A2UI 做出东西了吗？[分享给社区吧！](community.md)

---

## 下一步

- [快速入门指南](../quickstart.md) - 试用演示
- [代理开发](../guides/agent-development.md) - 构建代理
- [客户端设置](../guides/client-setup.md) - 集成渲染器
- [社区](community.md) - 加入社区

---

**正在生产环境中使用 A2UI？** 欢迎在 [GitHub Discussions](https://github.com/google/A2UI/discussions) 分享你的故事。
