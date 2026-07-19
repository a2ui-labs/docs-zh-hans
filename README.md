# A2UI：Agent-to-User Interface（智能体到用户界面）

A2UI 是一个开源项目，提供了一套适合表达“可更新的、由智能体生成的 UI”的格式与初始渲染器集合，让智能体能够生成或填充丰富的用户界面。

<img src="docs/public/assets/a2ui_gallery_examples.png" alt="A2UI 组件画廊" height="400">

*A2UI 渲染卡片的示例画廊，展示了它能够实现的多种界面组合。*

> **当前状态：** 该目录用于维护 A2UI 文档的简体中文版本，文档树结构与上游 `A2UI/docs/public` 保持一致。

## ⚠️ 状态：早期公开预览

> **说明：** A2UI 当前的正式发布版本是 **v0.9.1**，属于稳定的 v0.9 协议系列中的一个补丁版本。v1.0 规范目前是候选版本，v0.8 则已成为旧版。规范和实现已经可用，但仍在持续演进。我们开放该项目，是为了促进协作、收集反馈，并欢迎社区贡献（尤其是客户端渲染器方面）。后续会有变更。

## 概述

生成式 AI 非常擅长生成文本和代码，但当智能体需要向用户呈现丰富、可交互的界面时，尤其是在智能体远程运行或跨越信任边界时，这件事会变得困难。

**A2UI** 是一套开放标准和库，让智能体能够“说 UI”。智能体发送描述 UI *意图* 的声明式 JSON，客户端应用再用自己的原生组件库（Flutter、Angular、Lit 等）去渲染它。

这种方式让智能体生成的 UI 既能像数据一样安全，又能像代码一样富有表达力。

## 高层设计理念

A2UI 的设计目标，是解决智能体生成式或模板化 UI 响应在跨平台、可互操作场景中的关键问题。

项目的核心理念包括：

* **安全优先**：运行由 LLM 生成的任意代码可能带来安全风险。A2UI 是声明式数据格式，而不是可执行代码。你的客户端应用维护一份受信任、预先批准的 UI 组件“目录”（例如 Card、Button、TextField），智能体只能请求渲染目录中已有的组件。
* **LLM 友好且支持增量更新**：UI 被表示为带 ID 引用的扁平组件列表，便于 LLM 分步生成，因此天然适合渐进式渲染和响应迅速的用户体验。随着对话推进，智能体可以根据新的用户请求高效地逐步修改 UI。
* **与框架无关且可移植**：A2UI 将 UI 结构与 UI 实现分离。智能体发送组件树描述及其关联的数据模型，客户端应用负责将这些抽象描述映射到自身原生控件上，无论是 Web Components、Flutter Widgets、React Components，还是 SwiftUI Views。来自智能体的同一份 A2UI JSON 可以在不同框架构建的多个客户端中渲染。
* **灵活性**：A2UI 还提供开放注册表模式，允许开发者把服务端类型映射到自定义客户端实现，从原生移动组件到 React 组件都可以。通过注册“Smart Wrapper”，你可以把任何已有 UI 组件，包括用于承载遗留内容的安全 iframe 容器，接入 A2UI 的数据绑定与事件系统。更重要的是，这把安全控制权真正交还给开发者，让你可以在自定义组件逻辑中直接实施严格的沙箱策略与“信任阶梯”，而不是只依赖核心系统。

## 用例

一些典型场景包括：

* **动态数据收集**：智能体根据对话上下文（例如复杂预约场景）生成定制表单，包括日期选择器、滑块和输入框。
* **远程子智能体**：编排型智能体把任务委托给远程专业智能体（例如旅行预订智能体），后者返回一段可在主聊天窗口中渲染的 UI 载荷。
* **自适应工作流**：企业智能体可根据用户问题即时生成审批面板或数据可视化界面。

## 架构

A2UI 将“生成 UI”和“执行 UI”明确分离：

1. **生成**：智能体（使用 Gemini 或其他 LLM）生成或使用一份预先生成的 `A2UI Response`，即描述 UI 组件结构及其属性的 JSON 载荷。
2. **传输**：该消息被发送到客户端应用（通过 A2A、AG-UI 等）。
3. **解析**：客户端中的 **A2UI Renderer** 解析这段 JSON。
4. **渲染**：Renderer 将抽象组件（例如 `type: 'text-field'`）映射到客户端代码中的具体实现。

## 依赖关系

A2UI 被设计为一种轻量格式，但它适用于更大的生态系统：

* **传输层**：兼容 **A2A Protocol** 与 **AG-UI**。
* **LLM**：凡是能够输出 JSON 的模型，都可以生成 A2UI。
* **宿主框架**：需要使用受支持框架构建宿主应用（当前主要是 Web 或 Flutter）。

## 开始使用

选择一条与你的起点相匹配的路径：

| 路径                                                                                                                          | 你将获得                                                                                                                            | 时间   |
| ----------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| 🍜 **[快速开始：Restaurant Finder 演示](https://a2ui.org/quickstart/)**                                                      | 在本地完整运行 A2UI 全栈示例，使用 Gemini 驱动的 ADK 智能体和 Lit renderer。端到端学习 A2UI，并可按需自行定制。 | ~5 分钟 |
| ⚛️ **[在任意 Agent 框架与 Harness 中使用 A2UI](docs/public/guides/a2ui-with-any-agent-framework.md)**                     | 为你选择的框架或 harness 脚手架搭建一个 AG-UI 应用，然后在其上启用基于 AG-UI 的 A2UI 渲染。                                   | ~5 分钟 |
| 🎨 **[A2UI Composer](https://a2ui-composer.ag-ui.com/)** · **[Widget Builder](https://go.copilotkit.ai/A2UI-widget-builder)** | 通过可视化编辑器生成 A2UI JSON，并粘贴到任意智能体提示词中——无需安装。                                                       | ~1 分钟 |
| 🎬 **[A2UI Theater](https://a2ui-composer.ag-ui.com/theater)**                                                                | 在 Lit、React 和 Angular renderer 中逐步查看预置的 A2UI 流式场景——无需安装。                                         | ~1 分钟 |

### Restaurant Finder 演示——概要

前置条件：Node.js 18+（已启用 [Corepack](https://nodejs.org/api/corepack.html)）、[uv](https://docs.astral.sh/uv/)，以及一个 [Gemini API key](https://aistudio.google.com/apikey)。

```bash
git clone https://github.com/a2ui-project/a2ui.git
cd a2ui
export GEMINI_API_KEY="your_gemini_api_key"

# 启用 Corepack（macOS Homebrew 用户请参见下方提示）
corepack enable

yarn install
cd samples/client/lit
yarn demo:restaurant
```

> [!TIP]
> **macOS Homebrew 用户：** 如果你之前安装过独立的包管理器，请先解除冲突再安装 Corepack，这样 Corepack 才能按项目管理版本：
>
> ```bash
> brew unlink yarn pnpm
> brew install corepack
> corepack enable
> ```

以上命令会跨工作区安装依赖、构建 renderer、启动 Python 智能体，并在 `http://localhost:5173` 打开客户端。有关分步说明、其他演示以及故障排查，请参阅**[完整的快速开始指南](docs/public/quickstart.md)**。

### 在任意 Agent 框架与 Harness 中使用 A2UI——概要

```bash
npx create-ag-ui-app@latest
```

使用 AG-UI CLI 搭配你选择的框架或 harness（Google Chat、ADK、LangGraph、CrewAI、Mastra、Strands、Slack、Teams 等），然后按照**[AG-UI 指南](docs/public/guides/a2ui-with-any-agent-framework.md)**启用 A2UI 渲染。部分脚手架方案底层使用 [CopilotKit 的 A2UI 运行时](https://docs.copilotkit.ai/generative-ui/a2ui)搭配 Next.js，但整体搭建方式仍以 AG-UI 为核心。

### 其他 renderer

如果你是 Flutter 开发者，可以看看 [GenUI SDK](https://github.com/flutter/genui)，它底层使用的就是 A2UI。完整的客户端实现列表参见 [docs/public/reference/renderers.md](docs/public/reference/renderers.md)。

## 路线图

我们希望与社区一起推进以下工作：

* **规范稳定化**：迈向 v1.0 规范。
* **更多渲染器**：增加对 React、Jetpack Compose、iOS（SwiftUI）等的官方支持。
* **更多传输方式**：支持 REST 等更多方式。
* **更多智能体框架**：支持 Genkit、LangGraph 等。

## 贡献

A2UI 采用 **Apache 2.0** 许可证。我们相信 UI 的未来将是 agentic 的，也希望与你一起把它做出来。

关于如何开始贡献，请参阅 [CONTRIBUTING.md](CONTRIBUTING.md)。
