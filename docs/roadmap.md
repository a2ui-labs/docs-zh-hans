# 路线图

这份路线图概述了 A2UI 项目的当前状态与未来计划。项目仍在积极开发中，优先级可能会根据社区反馈与新出现的使用场景而调整。

## 当前状态

### 协议

| 版本 | 状态 | 说明 |
|---------|--------|-------|
| **v0.8** | ✅ 稳定版 | 首个公开版本 |
| **v0.9** | 🚧 草案 | 以提示词优先为核心的规范改进 |

主要特性：

- ✅ 流式 JSONL 消息格式
- ✅ 四种核心消息类型（`surfaceUpdate`、`dataModelUpdate`、`beginRendering`、`deleteSurface`）
- ✅ Adjacency list 组件模型
- ✅ 基于 JSON Pointer 的数据绑定
- ✅ 结构与状态分离

### 渲染器

| 客户端库 | 状态 | 平台 | 说明 |
|-----------------|--------|----------|-------|
| **Web Components (Lit)** | ✅ 稳定版 | Web | 与框架无关，到处可用 |
| **Angular** | ✅ 稳定版 | Web | 完整 Angular 集成 |
| **Flutter (GenUI SDK)** | ✅ 稳定版 | 多平台 | 可用于移动端、Web 和桌面端 |
| **React** | 🚧 开发中 | Web | 预计 2026 Q1 |
| **SwiftUI** | 📋 规划中 | iOS/macOS | 预计 2026 Q2 |
| **Jetpack Compose** | 📋 规划中 | Android | 预计 2026 Q2 |
| **Vue** | 💡 提议中 | Web | 社区有需求 |
| [**Svelte/Kit**](https://svelte.dev/docs/kit/introduction) | 💡 提议中 | Web | [社区有需求](https://news.ycombinator.com/item?id=46287728) |
| **ShadCN (React)** | 💡 提议中 | Web | 社区有需求 |

### 传输层

| 传输方式 | 状态 | 说明 |
|-------------|--------|-------|
| **A2A Protocol** | ✅ 已完成 | 原生 A2A 传输 |
| **AG UI** | ✅ 已完成 | Day-zero 兼容 |
| **REST API** | 📋 规划中 | 双向通信 |
| **WebSockets** | 💡 提议中 | 双向通信 |
| **SSE (Server-Sent Events)** | 💡 提议中 | Web 流式传输 |
| **MCP (Model Context Protocol)** | 💡 提议中 | 社区有兴趣 |

### 智能体 UI 工具包

| Agent UI 工具包 | 状态 | 说明 |
|-------------|--------|-------|
| **CopilotKit** | ✅ 已完成 | 基于 AG UI 实现 Day-zero 兼容 |
| **Open AI ChatKit** | 💡 提议中 | 社区有兴趣 |
| **Vecel AI SDK UI** | 💡 提议中 | 社区有兴趣 |

### 智能体框架

| 集成项 | 状态 | 说明 |
|-------------|--------|-------|
| **任何支持 A2A 的 Agent** | ✅ 已完成 | 基于 A2A 协议即可获得 Day-zero 兼容 |
| **ADK** | 📋 规划中 | 仍在设计更好的开发体验，参考 [samples](https://github.com/google/A2UI/tree/main/samples/agent/adk) |
| **Genkit** | 💡 提议中 | 社区有兴趣 |
| **LangGraph** | 💡 提议中 | 社区有兴趣 |
| **CrewAI** | 💡 提议中 | 社区有兴趣 |
| **AG2** | 💡 提议中 | 社区有兴趣 |
| **Claude Agent SDK** | 💡 提议中 | 社区有兴趣 |
| **OpenAI Agent SDK** | 💡 提议中 | 社区有兴趣 |
| **Microsoft Agent Framework** | 💡 提议中 | 社区有兴趣 |
| **AWS Strands Agent SDK** | 💡 提议中 | 社区有兴趣 |

## 最近里程碑

### 2025 年 Q2

Google 多个团队开展了大量研究项目，并将其集成到内部产品和智能体中。

### 2025 年 Q4

- 发布 v0.8.0 规范
- 发布 A2A 扩展（感谢 Google A2A 团队，在 [a2asummit.ai](https://a2asummit.ai) 上首次预告）
- 发布 Flutter 渲染器（感谢 Flutter 团队）
- 发布 Angular 渲染器（感谢 Angular 团队）
- 发布 Web Components（Lit）渲染器（感谢 Opal 团队及朋友们）
- 发布 AG UI / CopilotKit 集成（感谢 CopilotKit 团队）
- GitHub 上公开发布（Apache 2.0）

## 即将到来的里程碑

### 2026 年 Q1

#### A2UI v0.9

- 发布 0.9 规范候选版本
- 改进渲染器的主题支持（完整）
- 改进智能体侧的服务端主题支持（最小可用）
- 提升开发体验

#### React 渲染器

提供一个原生 React 渲染器，带有基于 hooks 的 API 和完整 TypeScript 支持。

- React 对常见组件的支持
- React 对自定义组件的支持
- 用于处理消息的 `useA2UI` hook
- React 的主题支持

### 2026 年 Q2

#### 原生移动端渲染器

面向 iOS 与 Android 平台的原生渲染器。

**SwiftUI Renderer（iOS / macOS）：**

- 原生 SwiftUI 组件
- 支持 iOS 设计语言
- 支持 macOS

**Jetpack Compose Renderer（Android）：**

- 原生 Compose UI 组件
- 支持 Material Design 3
- 深度集成 Android 平台

#### 性能优化

- 渲染器性能基准测试
- 大型组件树的懒加载
- 列表虚拟滚动
- 组件记忆化策略

### 2026 年 Q4

#### 协议 v1.0

完成 v1.0 协议定稿，包括：

- 稳定性保证
- 从 v0.9 迁移的路径
- 完整测试套件
- 渲染器认证计划

## 长期愿景

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
- Agent marketplace 集成
- 企业级功能与支持

## 社区需求

以下是社区提出的一些需求（排名不分先后）：

- **更多渲染器集成**：把你的客户端库映射到 A2UI
- **更多智能体框架**：把你的智能体框架映射到 A2UI
- **更多传输层**：把你的传输层映射到 A2UI
- **社区组件库**：与社区共享自定义组件
- **社区示例**：与社区共享自定义示例
- **社区评测**：生成式 UI 的评测场景与标注数据集
- **开发体验优化**：如果你能构建更好的 A2UI 使用体验，欢迎分享给社区

## 如何影响路线图

我们欢迎社区参与优先级讨论：

1. **给 Issues 投票**：为你关心的 GitHub issues 点 👍
2. **提出功能建议**：在 GitHub 上发起 discussion（先搜索是否已有相关讨论）
3. **提交 PR**：自己动手构建你需要的功能（先搜索是否已有相关 PR）
4. **参与讨论**：分享你的使用场景和需求（先搜索是否已有相关讨论）

## 发布周期

- **大版本**（1.0、2.0）：每年一次，或在需要重大破坏性变更时发布
- **小版本**（1.1、1.2）：按季度发布新功能
- **补丁版本**（1.1.1、1.1.2）：按需发布 bug 修复

## 版本策略

A2UI 遵循 [Semantic Versioning](https://semver.org/)：

- **MAJOR**：不兼容的协议变更
- **MINOR**：向后兼容的新功能
- **PATCH**：向后兼容的 bug 修复

## 参与进来

想参与路线图建设？

- 在 [GitHub Discussions](https://github.com/google/A2UI/discussions) 中**提出功能建议**
- **构建原型**并分享给社区
- 在 GitHub Issues 中**参与讨论**

## 保持关注

- Watch [GitHub 仓库](https://github.com/google/A2UI) 获取更新
- Star 仓库以表达支持
- 关注 releases 以便及时获知新版本

---

**最后更新：** 2026 年 3 月

如果你对路线图有问题，请在 [GitHub](https://github.com/google/A2UI/discussions) 发起讨论。
