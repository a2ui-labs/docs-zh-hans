---
hide:
  - toc
---

<!-- markdownlint-disable MD041 -->
<!-- markdownlint-disable MD033 -->
<div style="text-align: center; margin: 2rem 0 3rem 0;" markdown>

<!-- Logo for Light Mode (shows dark logo on light background) -->
<img src="assets/A2UI_dark.svg" alt="A2UI Logo" width="120" class="light-mode-only" style="margin-bottom: 1rem;">
<!-- Logo for Dark Mode (shows light logo on dark background) -->
<img src="assets/A2UI_light.svg" alt="A2UI Logo" width="120" class="dark-mode-only" style="margin-bottom: 1rem;">

# 面向智能体驱动界面的协议

<p style="font-size: 1.2rem; max-width: 800px; margin: 0 auto 1rem auto; opacity: 0.9; line-height: 1.6;">
A2UI 让 AI 智能体能够生成丰富、可交互的用户界面，并在 Web、移动端和桌面端以原生方式渲染，而无需执行任意代码。
</p>

</div>

## 规范版本

| 版本 | 状态 | 说明 |
|---------|--------|-------------|
| **[v0.8](specification/v0.8-a2ui.md)** | **稳定版** | 当前可用于生产的版本。包含 surface、component、data binding 与 adjacency list 模型。 |
| **[v0.9](specification/v0.9-a2ui.md)** | **草案** | 新增 `createSurface`、客户端函数、自定义 catalog 以及扩展规范。[查看演进指南 →](specification/v0.9-evolution-guide.md) |

A2UI 采用 Apache 2.0 许可证，
由 Google 发起，CopilotKit 与开源社区共同参与贡献，
并正在 [GitHub](https://github.com/google/A2UI) 上持续活跃开发。

A2UI 要解决的问题是：**AI 智能体如何在跨越信任边界的情况下，安全地发送丰富的 UI？**

不同于只返回文本，或直接执行高风险代码，A2UI 让智能体发送**声明式组件描述**，再由客户端使用自己的原生控件进行渲染。这就像让智能体学会了一门通用的 UI 语言。

在这个仓库中，你会找到
[A2UI 规范](specification/v0.8-a2ui.md)（v0.8 稳定版，v0.9 草案），
客户端侧的 [渲染器](reference/renderers.md) 实现（Angular、Flutter、Lit、Markdown 等），
以及用于在智能体与客户端之间传递 A2UI 消息的 [传输层](concepts/transports.md)（如 A2A）。

<div class="grid cards" markdown>

- :material-shield-check: **从设计上保证安全**

    ---

    它是声明式数据格式，而不是可执行代码。智能体只能使用你在目录中预先批准的组件，从源头避免 UI 注入攻击。

- :material-rocket-launch: **对 LLM 友好**

    ---

    扁平、可流式传输的 JSON 结构，专为易生成而设计。LLM 不需要一次性生成完美 JSON，也可以增量构建 UI。

- :material-devices: **框架无关**

    ---

    一份智能体响应可在多端复用。你可以在 Angular、Flutter、React 或原生移动端中，用自己的样式组件渲染同一份 UI。

- :material-chart-timeline: **渐进式渲染**

    ---

    UI 更新生成后即可立即流式传输。用户无需等待完整响应，就能实时看到界面逐步构建出来。

</div>

## 5 分钟快速上手

<div class="grid cards" markdown>

- :material-clock-fast:{ .lg .middle } **[快速开始指南](quickstart.md)**

    ---

    运行餐厅查找示例，亲自体验由 Gemini 驱动的智能体如何使用 A2UI。

    [:octicons-arrow-right-24: 立即开始](quickstart.md)

- :material-book-open-variant:{ .lg .middle } **[核心概念](concepts/overview.md)**

    ---

    了解 surface、组件、数据绑定与邻接表模型。

    [:octicons-arrow-right-24: 学习概念](concepts/overview.md)

- :material-code-braces:{ .lg .middle } **[开发指南](guides/client-setup.md)**

    ---

    将 A2UI 渲染器集成到你的应用中，或者构建能够生成 UI 的智能体。

    [:octicons-arrow-right-24: 开始构建](guides/client-setup.md)

- :material-file-document:{ .lg .middle } **协议规范**

    ---

    深入阅读完整技术规范：[v0.8（稳定版）](specification/v0.8-a2ui.md) · [v0.9（草案）](specification/v0.9-a2ui.md)

    [:octicons-arrow-right-24: 阅读 v0.8 规范](specification/v0.8-a2ui.md)

</div>

## 工作原理

1. **用户发送消息** 给 AI 智能体
2. **智能体生成 A2UI 消息**，描述 UI（结构 + 数据）
3. **消息被流式传输** 到客户端应用
4. **客户端使用原生组件渲染**（Angular、Flutter、React 等）
5. **用户与 UI 交互**，再把操作回传给智能体
6. **智能体响应**，并返回更新后的 A2UI 消息

![End-to-End Data Flow](assets/end-to-end-data-flow.png)

## A2UI 实际效果

### 景观设计师示例

<div style="margin: 2rem 0;">
  <div style="border-radius: .8rem; overflow: hidden; box-shadow: var(--md-shadow-z2);">
    <video width="100%" height="auto" controls playsinline style="display: block; aspect-ratio: 16/9; object-fit: cover;">
      <source src="assets/landscape-architect-demo.mp4" type="video/mp4">
      你的浏览器不支持 video 标签。
    </video>
  </div>
  <p style="text-align: center; margin-top: 1rem; opacity: 0.8;">
    观看智能体为景观设计应用动态生成完整界面。用户上传照片后，智能体使用 Gemini 理解图像内容，并生成针对景观需求的定制表单。
  </p>
</div>

### 自定义组件：交互式图表与地图

<div style="margin: 2rem 0;">
  <div style="border-radius: .8rem; overflow: hidden; box-shadow: var(--md-shadow-z2);">
    <video width="100%" height="auto" controls playsinline style="display: block; aspect-ratio: 16/9; object-fit: cover;">
      <source src="assets/a2ui-custom-component.mp4" type="video/mp4">
      你的浏览器不支持 video 标签。
    </video>
  </div>
  <p style="text-align: center; margin-top: 1rem; opacity: 0.8;">
    观看智能体如何在回答数值汇总问题时选择图表组件，在回答地理位置问题时选择 Google Map 组件。这两个组件都由客户端作为自定义组件提供。
  </p>
</div>

### A2UI Composer

CopilotKit 也提供了一个公开可试用的 [A2UI Widget Builder](https://go.copilotkit.ai/A2UI-widget-builder)。

[![A2UI Composer](assets/A2UI-widget-builder.png)](https://go.copilotkit.ai/A2UI-widget-builder)
