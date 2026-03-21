# 生态渲染器

由社区和第三方维护的 A2UI 渲染器实现。

!!! note
    这些渲染器由各自作者维护，而不是由 A2UI 团队维护。
    请自行确认每个项目的兼容性、版本支持范围和维护状态。

## 社区渲染器

| 渲染器 | 平台 | v0.8 | v0.9 | 活跃度 | 链接 |
|----------|----------|------|------|----------|-------|
| **@a2ui-sdk/react** | React（Web） | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/easyops-cn/a2ui-sdk?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/easyops-cn/a2ui-sdk?style=flat-square&label=updated) | [GitHub](https://github.com/easyops-cn/a2ui-sdk) · [npm](https://www.npmjs.com/package/@a2ui-sdk/react) · [文档](https://a2ui-sdk.js.org/) |
| **A2UI-Android** | Android（Compose） | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/lmee/A2UI-Android?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/lmee/A2UI-Android?style=flat-square&label=updated) | [GitHub](https://github.com/lmee/A2UI-Android) |
| **a2ui-react-native** | React Native | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/sivamrudram-eng/a2ui-react-native?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/sivamrudram-eng/a2ui-react-native?style=flat-square&label=updated) | [GitHub](https://github.com/sivamrudram-eng/a2ui-react-native) |
| **@zhama/a2ui** | React（Web） | ✅ | ❌ | — | [npm](https://www.npmjs.com/package/@zhama/a2ui) |
| **A2UI-react** | React（Web） | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/jem-computer/A2UI-react?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/jem-computer/A2UI-react?style=flat-square&label=updated) | [GitHub](https://github.com/jem-computer/A2UI-react) |

### 值得关注的项目

这些项目仍处于早期阶段或实验阶段：

- **[@xpert-ai/a2ui-react](https://www.npmjs.com/package/@xpert-ai/a2ui-react)** — 基于 ShadCN UI 组件的 React 渲染器（v0.0.1，发布于 2026 年 1 月）
- **[a2ui-3d-renderer](https://github.com/josh-english-2k18/a2ui-3d-renderer)** — 面向 A2UI 的实验性 Three.js/WebGL 3D 渲染器（约 2 星）
- **[ai-kit-a2ui](https://github.com/AINative-Studio/ai-kit-a2ui)** — 面向 AIKit 框架的 React + ShadCN 渲染器（约 2 星）

### 重点项目

**@a2ui-sdk/react** 是当前最成熟的社区 React 渲染器，已经发布了 11 个版本，使用 Radix UI primitives、Tailwind CSS，并拥有独立文档站点。它最早在 [A2UI 讨论区](https://github.com/google/A2UI/discussions/489) 中发布。

**A2UI-Android** 填补了一个重要空白，它是目前唯一的社区 Jetpack Compose 渲染器，覆盖 Android 5.0+，提供 20+ 个组件，并支持数据绑定与可访问性。

**a2ui-react-native** 是目前唯一的 React Native 渲染器，可通过一套代码在 iOS 和 Android 上使用 A2UI。

### Python / PyPI

截至 2026 年 3 月，PyPI 上还没有可信的 A2UI 渲染器包。A2UI 渲染器本质上是客户端（UI）库，因此其生态自然主要集中在 JavaScript / TypeScript 与原生移动框架上。

## 提交你的渲染器

如果你构建了 A2UI 渲染器，我们很愿意把它列在这里。

### 提交方式

1. **Fork** [google/A2UI](https://github.com/google/A2UI) 仓库
2. **编辑** 本文件（`docs/ecosystem/renderers.md`）—— 在社区渲染器表格中新增一行，填入渲染器名称、平台、npm 包（如有）、支持版本和源码链接
3. **向 `google/A2UI` 提交 PR**，并简要描述你的渲染器
4. **在 [GitHub Discussions](https://github.com/google/A2UI/discussions) 发帖** —— 让社区知道你做了什么。一个简短的演示视频会非常有帮助。

需要灵感？可以浏览仓库中的 **[community samples](https://github.com/google/A2UI/tree/main/samples)**。这些示例覆盖 Angular、Lit 和基于 ADK 的智能体，是很好的起点。

### 什么样的社区渲染器更容易被采纳？

如果你的项目具备以下特点，就更有可能被接受并被他人使用：

- 有**公开源码**（最好开源，MIT 或 Apache 2.0 更佳）
- 明确说明**支持哪个 A2UI 规范版本**（v0.8、v0.9 或两者）
- 覆盖**核心组件**：文本、按钮、输入组件与基础布局
- 带有包含安装说明和最小示例的 **README**
- **持续维护中** —— 如果你已不再维护，请明确标记为 archived

社区渲染器并不需要已经达到生产可用水平才可以列出。实验性和早期项目也非常欢迎进入“值得关注的项目”部分。
