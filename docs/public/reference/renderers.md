# 渲染器（客户端库）

渲染器负责把 A2UI JSON 消息转换为适用于不同平台的原生 UI 组件。

[智能体](agents.md) 负责生成 A2UI 消息，
[传输层](../concepts/transports.md) 负责把消息传送到客户端。
客户端渲染器库必须能够缓冲并处理 A2UI 消息、实现 A2UI 生命周期，并渲染 surface（组件）。

你拥有很高的自由度，可以为现有渲染器接入自定义组件，也可以自行构建渲染器，以支持你自己的 UI 组件框架。

## 官方维护的渲染器

### Web

| 渲染器 | 平台 | v0.8 | v0.9.1 | v1.0 | 链接 |
|----------|----------|------|------|------|-------|
| **React** | Web | ✅ 稳定 | ✅ 稳定 | 🚧 计划中 | [代码](https://github.com/a2ui-project/a2ui/tree/main/renderers/react) |
| **Lit（Web Components）** | Web | ✅ 稳定 | ✅ 稳定 | 🚧 计划中 | [代码](https://github.com/a2ui-project/a2ui/tree/main/renderers/lit) |
| **Angular** | Web | ✅ 稳定 | ✅ 稳定 | 🚧 计划中 | [代码](https://github.com/a2ui-project/a2ui/tree/main/renderers/angular) |
| **Flutter（GenUI SDK）** | 移动端 / 桌面端 / Web | ✅ 稳定 | ✅ 稳定 | 🚧 计划中 | [文档](https://docs.flutter.dev/ai/genui) · [代码](https://github.com/flutter/genui) |

### 移动端

| 渲染器 | 平台 | v0.8 | v0.9.1 | v1.0 | 链接 |
|----------|----------|------|------|------|-------|
| **Flutter（GenUI SDK）** | 移动端 / 桌面端 / Web | ✅ 稳定 | ✅ 稳定 | 🚧 计划中 | [文档](https://docs.flutter.dev/ai/genui) · [代码](https://github.com/flutter/genui) |
| **SwiftUI** | iOS / macOS | — | — | 🚧 计划中 | — |
| **Jetpack Compose** | Android | — | — | 🚧 计划中 | — |

更多内容可查看 [路线图](../roadmap.md)。

## 生态渲染器

社区正在为更多平台构建 A2UI 渲染器：

- **[json-render](https://json-render.dev/docs/a2ui)** — Vercel 的 React 库，基于 Zod schema 渲染 A2UI 目录（[对比文章](https://dipjyotimetia.medium.com/vercels-json-render-vs-google-s-a2ui-the-head-to-head-6f213cf1a23b)）
- **[A2UI-Android](https://github.com/lmee/A2UI-Android)** — 社区版 Jetpack Compose 渲染器，提供 20+ 个组件（约 15 ⭐，v0.8）
- **[a2ui-react-native](https://github.com/sivamrudram-eng/a2ui-react-native)** — 面向 iOS/Android 的 React Native 渲染器（约 9 ⭐，v0.8）
- **[Lynx A2UI](https://lynxjs.org/next/react/genui/a2ui.html)** — 面向 A2UI 的 ReactLynx 渲染器（v0.9）

如需查看更多社区项目，以及了解如何提交自己的实现，请参阅 **[完整生态渲染器列表](../ecosystem/renderers.md)**。

## 渲染器如何工作

```
A2UI JSON → 渲染器 → 原生组件 → 你的应用
```

1. **接收** 传输层发送来的 A2UI 消息
2. **解析** JSON 并按 schema 进行校验
3. **使用平台原生组件渲染**
4. **按照应用主题进行样式处理**

## 使用渲染器

如果要将 A2UI 集成到你的应用中，可以按所选渲染器的接入指南开始：

- **[React](../guides/client-setup.md#react)**
- **[Lit（Web Components）](../guides/client-setup.md#web-components-lit)**
- **[Angular](../guides/client-setup.md#angular)**
- **[Flutter（GenUI SDK）](../guides/client-setup.md#flutter-genui-sdk)**

## 构建渲染器

想为你的平台构建一个渲染器？

- 查看 [路线图](../roadmap.md)，了解已规划的框架
- 参考已有渲染器的实现模式
- 阅读 [渲染器开发指南](../guides/renderer-development.md)，了解具体实现细节

### 关键要求

- 能解析 A2UI JSON 消息，尤其是 adjacency list 格式
- 能将 A2UI 组件映射到原生组件
- 能处理数据绑定与生命周期事件
- 能处理一系列增量式 A2UI 消息，以构建并更新 UI
- 支持服务端发起的更新
- 支持用户操作

### 下一步

- **[客户端接入指南](../guides/client-setup.md)**：查看集成说明
- **[快速开始](../quickstart.md)**：实际运行 Lit 渲染器
- **[组件参考](components.md)**：了解需要支持哪些组件
