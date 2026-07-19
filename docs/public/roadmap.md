# 路线图

这份路线图概述了 A2UI 项目的当前状态与未来计划。项目仍在积极开发中，优先级可能会根据社区反馈与新出现的使用场景而调整。

## 当前状态

### 协议

| 版本 | 状态 | 说明 |
|------|------|------|
| **v0.8** | 🆗 历史版本 | 首个公开版本，提供最简支持 |
| **v0.9** | 🆗 历史版本 | 功能完整，提供历史版本支持 |
| **v0.9.1** | 🆗 当前版本 | 稳定版本 |
| **v1.0** | 📋 候选版本 | 发布候选版本 |

### 渲染器

| 客户端库 | 状态 | 平台 | 说明 |
|----------|------|------|------|
| **Web Core Lib** | ✅ 稳定版 | Web | 所有 Web 渲染器共享的核心库 |
| **Web Components (Lit)** | ✅ 稳定版 | Web | 与框架无关，到处可用 |
| **Angular** | ✅ 稳定版 | Web | 完整 Angular 集成 |
| **Flutter (GenUI SDK)** | ✅ 稳定版 | 多平台 | 可用于移动端、Web 和桌面端 |
| **React** | ✅ 稳定版 | Web | 官方 React 渲染器 |
| [**Lynx**](https://lynxjs.org/next/react/genui/a2ui.html) | ✅ 稳定版 | 移动端、Web、桌面端 | 面向 A2UI v0.9 的 ReactLynx 渲染器 |
| **SwiftUI** | 📋 规划中 | iOS/macOS | 预计 2026 Q2 |
| **Jetpack Compose** | 📋 规划中 | Android | 预计 2026 Q2 |
| **Vue** | 💡 提议中 | Web | 社区有需求 |
| [**Svelte/Kit**](https://svelte.dev/docs/kit/introduction) | 💡 提议中 | Web | [社区有需求](https://news.ycombinator.com/item?id=46287728) |
| **ShadCN (React)** | 💡 提议中 | Web | 社区有需求 |

### 传输层

| 传输方式 | 状态 | 说明 |
|----------|------|------|
| **A2A Protocol** | ✅ 已完成 | 原生 A2A 传输 |
| **AG-UI** | ✅ 已完成 | Day-zero 兼容 |
| **REST API** | ✅ 已完成 | 基于 HTTP 的请求/响应 |
| **WebSockets** | ✅ 已完成 | 双向 WS 通信 |
| **MCP (Model Context Protocol)** | ✅ 已完成 | 上下文共享 |

### 智能体框架

| 集成项 | 状态 | 说明 |
|--------|------|------|
| **任何支持 A2A 的 Agent** | ✅ 已完成 | 基于 A2A 协议即可获得 Day-zero 兼容 |
| **任何支持 AG-UI 的 Agent** | ✅ 已完成 | 基于 AG-UI 协议即可获得 Day-zero 兼容 |
| **AG2** | ✅ 已完成 | [A2UIAgent](https://docs.ag2.ai/latest/docs/user-guide/reference-agents/a2uiagent) |
| **ADK** | 🚧 开发中 | 仍在设计开发者体验，参考 [samples](../../samples/agent/adk) |
| **Genkit** | 💡 提议中 | 社区有兴趣 |
| **LangGraph** | 💡 提议中 | 社区有兴趣 |
| **CrewAI** | 💡 提议中 | 社区有兴趣 |
| **Claude Agent SDK** | 💡 提议中 | 社区有兴趣 |
| **OpenAI Agent SDK** | 💡 提议中 | 社区有兴趣 |
| **Microsoft Agent Framework** | 💡 提议中 | 社区有兴趣 |
| **AWS Strands Agent SDK** | 💡 提议中 | 社区有兴趣 |

## 最近里程碑

### 2025 年 Q2

Google 多个团队开展了大量研究项目，并将其集成到内部产品和智能体中。

### 2025 年 Q4 v0.8

- 发布 v0.8.0 规范
- 发布 A2A 扩展（感谢 Google A2A 团队，在 [a2asummit.ai](https://a2asummit.ai) 上首次预告）
- 发布 Flutter 渲染器（感谢 Flutter 团队）
- 发布 Angular 渲染器（感谢 Angular 团队）
- 发布 Web Components（Lit）渲染器（感谢 Opal 团队及朋友们）
- 发布 AG-UI / CopilotKit 集成（感谢 CopilotKit 团队）
- GitHub 上公开发布（Apache 2.0）

### 2026 年 Q2 v0.9

- 发布 0.9 规范
- 发布支持 0.9 规范的 Web core 与渲染器
- 推出官方 A2UI React 渲染器
- 改进渲染器的主题支持
- 推出 A2UI Agents SDK（Python）
- 提升开发体验

## 即将到来的里程碑

### 2026 年 Q3 v0.9 与 v1.0

**Jetpack Compose Renderer（Android）：**

- 原生 Compose UI 组件
- 支持 Material Design 3
- 深度集成 Android 平台

**SwiftUI Renderer（iOS/macOS）：**

- 原生 SwiftUI 组件
- 支持 iOS 设计语言
- 支持 macOS

**性能优化**

- 渲染器性能基准测试
- 大型组件树的懒加载
- 列表虚拟滚动
- 组件记忆化策略

### 2026 年 Q4 v1.0

**完成协议 v1.0 的定稿，包括：**

- 稳定性保证
- 从 v0.9 迁移的路径
- 完整测试套件
- 渲染器认证计划

## 长期愿景

### 完整应用 UI

可配置、可定制的 UI —— 由 A2UI 提供支持（智能体可选）

- 面向构建时的 A2UI 开发者工具
- A2UI 缓存模式，用作静态 UI 的配置层
- 完整应用组合的设计与示例

### 多智能体协作

增强多个智能体共同作用于同一个 UI 的能力：

- 推荐的智能体组合模式
- 冲突解决策略
- 共享 surface 管理

### 可访问性特性

一等公民级别的可访问性支持：

- ARIA 属性生成
- 屏幕阅读器优化
- 键盘导航标准
- 对比度与配色指导

### 高级 UI 模式

支持更复杂的 UI 交互：

- 拖放
- 手势与动画
- 3D 渲染
- AR / VR 界面（探索中）

### 生态增长

- 更多框架集成
- 第三方组件库
- 智能体市场集成
- 企业级功能与支持

## 如何影响路线图

我们欢迎社区参与优先级讨论：

1. **提交反馈表单**：使用[这份匿名表单](https://forms.gle/Tj8E3dMsJ1NcvQnFA)分享你的需求与反馈
2. **给 Issues 投票**：为你关心的 GitHub issues 点 👍
3. **提出功能建议**：在 GitHub 上发起 discussion（先搜索是否已有相关讨论）
4. **提交 PR**：自己动手构建你需要的功能（先搜索是否已有相关 PR）
5. **参与讨论**：分享你的使用场景和需求（先搜索是否已有相关讨论）

## 发布周期

- **大版本**（1.0、2.0）：每年一次，或在需要重大破坏性变更时发布
- **小版本**（1.1、1.2）：按季度发布新功能
- **补丁版本**（1.1.1、1.1.2）：按需发布 bug 修复

## 版本策略

A2UI 遵循 [Semantic Versioning](https://semver.org/)：

- **MAJOR**：不兼容的协议变更
- **MINOR**：向后兼容的新功能
- **PATCH**：向后兼容的 bug 修复

## 保持关注

- Watch [GitHub 仓库](https://github.com/a2ui-project/a2ui) 获取更新
- Star 仓库以表达支持
- 关注 releases 以便及时获知新版本

---

**最后更新：** 2026 年 6 月

如果你对路线图有问题，请在 [GitHub](https://github.com/a2ui-project/a2ui/discussions) 发起讨论。
