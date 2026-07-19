# 客户端设置指南

使用适合你平台的渲染器，把 A2UI 集成到你的应用中。

## 渲染器

| 渲染器 | 平台 | v0.8 | v0.9 | 状态 |
| ------------------------ | ------------------ | ---- | ---- | ----------------- |
| **[React](https://github.com/a2ui-project/a2ui/tree/main/renderers/react)** | Web | ✅ | ✅ | ✅ 稳定 |
| **[Lit (Web Components)](https://github.com/a2ui-project/a2ui/tree/main/renderers/lit)** | Web | ✅ | ✅ | ✅ 稳定 |
| **[Angular](https://github.com/a2ui-project/a2ui/tree/main/renderers/angular)** | Web | ✅ | ✅ | ✅ 稳定 |
| **[Flutter (GenUI SDK)](https://docs.flutter.dev/ai/genui)** | Mobile/Desktop/Web | ✅ | ✅ | ✅ 稳定 |
| **SwiftUI** | iOS/macOS | — | — | 🚧 计划于 2026 Q2 |
| **Jetpack Compose** | Android | — | — | 🚧 计划于 2026 Q2 |

## 组件目录

组件目录可以是任意组件集合。A2UI 提供了一个 “Basic Catalog”，但我们预期你会添加自己的组件、共享库，或者完全用自己的组件替换掉基础组件。

**真正重要的是你的设计系统。** 你可以注册任意一组组件和函数，A2UI 都能配合它们工作。catalog 只是代理和渲染器之间的契约。

如何定义一个与你的设计系统相匹配的 catalog，请参见 [定义你自己的 Catalog](defining-your-own-catalog.md)。

## 共享 Web 库

所有 Web 渲染器（Lit、Angular、React）都共享一个共同基础：**`@a2ui/web_core`**。这个库提供消息处理器、状态管理和数据绑定逻辑，是每个 Web 渲染器都需要的。各框架专属的渲染器都建立在它之上，只额外添加各自框架的渲染层。

这意味着跨 Web 平台的核心协议处理是一致的，差异只在于组件渲染。

共享的 `web_core` 库提供：

- **Message Processor**：管理 A2UI 状态并处理传入消息。

## Web Components（Lit）

```bash
npm install @a2ui/lit @a2ui/web_core
```

安装完成后，即可在应用中使用该渲染器。Lit 渲染器使用：

- **Message Processor**：封装 A2UI message processor。
- **`<a2ui-surface>` 组件**：在你的应用中渲染 surface
- **Lit Signals**：提供响应式状态管理，支持自动 UI 更新

**查看可运行示例：** [Lit shell sample](https://github.com/a2ui-project/a2ui/tree/main/samples/client/lit/shell) —— 详细运行步骤请查看其 README。

## Angular

```bash
npm install @a2ui/angular @a2ui/web_core
```

安装完成后，即可在应用中使用该渲染器。Angular 渲染器提供：

- **`A2uiRendererService`**：管理 A2UI 消息处理器和响应式模型的服务。
- **`a2ui-v09-component-host` 组件**：一个动态组件宿主，用于渲染来自 surface 的 A2UI 组件。
- **`A2UI_RENDERER_CONFIG` token**：用于使用 catalogs 和动作处理器配置渲染器。

### 设置示例（v0.9）

A2UI 对协议相关实现使用版本化导入。对于 v0.9，请按如下方式配置应用的 providers：

```typescript
import { ApplicationConfig } from '@angular/core';
import {
  A2UI_RENDERER_CONFIG,
  A2uiRendererService,
  BasicCatalog
} from '@a2ui/angular/v0_9';

export const appConfig: ApplicationConfig = {
  providers: [
    {
      provide: A2UI_RENDERER_CONFIG,
      useValue: {
        catalogs: [new BasicCatalog()],
        actionHandler: (action) => {
          console.log('Action dispatched:', action);
        }
      }
    },
    A2uiRendererService
  ]
};
```

**查看可运行示例：** [Angular v0.9 Explorer](https://github.com/a2ui-project/a2ui/tree/main/renderers/angular/a2ui_explorer)

### Streaming

Angular 客户端默认使用流式 API。如果要禁用流式传输，请在启动应用前将 `ENABLE_STREAMING` 环境变量设为 `false`：

```bash
export ENABLE_STREAMING=false
yarn start restaurant
```

> [!NOTE]
> **包管理器说明：** 上面的 `yarn start` 命令是专门用于在 A2UI monorepo 仓库内运行示例应用的。如果是在本仓库之外做日常使用或独立项目，你可以选择自己喜欢的包管理器（例如 npm、pnpm）。

## React

```bash
npm install @a2ui/react @a2ui/web_core
```

React 渲染器提供：

- **`<A2UISurface>` 组件**：在你的 React 应用中渲染 A2UI surface
- **`useA2UI()` hook**：可从任意组件访问消息处理器
- **`MessageProcessor` 类**：处理传入的 A2UI 消息（与其他 Web 渲染器共享）

**查看可运行示例：** [React shell](https://github.com/a2ui-project/a2ui/tree/main/samples/client/react/shell)

## Flutter（GenUI SDK）

```bash
flutter pub add flutter_genui
```

Flutter 使用 GenUI SDK，提供原生的 A2UI 渲染能力。

**文档：** [GenUI SDK](https://docs.flutter.dev/ai/genui) | [GitHub](https://github.com/flutter/genui) | [GenUI Flutter Package 中的 README](https://github.com/flutter/genui/blob/main/packages/genui/README.md#getting-started-with-genui)

## 连接代理

你的客户端应用需要：

1. **接收 A2UI 消息**，来源于代理（通过传输层）
2. **处理消息**，使用 Message Processor
3. **把用户动作发送回代理**

常见的传输方式：

- **Server-Sent Events（SSE）**：从服务器到客户端的单向流式传输
- **WebSockets**：双向实时通信
- **A2A Protocol**：支持 A2UI 的标准化 agent-to-agent 通信

参见 [samples/client/lit/shell/client.ts](https://github.com/a2ui-project/a2ui/tree/main/samples/client/lit/shell/client.ts) 中使用 A2A protocol 客户端的示例。

**参见：** [Transports guide](../concepts/transports.md)

## 处理用户动作

当用户与 A2UI 组件交互时（点击按钮、提交表单等），客户端会：

1. 捕获组件发出的动作事件
2. 解析该动作所需的任何数据上下文
3. 将动作发送给代理
4. 处理代理返回的响应消息

参见 [samples/client/lit/shell/app.ts](https://github.com/a2ui-project/a2ui/tree/main/samples/client/lit/shell/app.ts) 中 `#maybeRenderData` 内的 `@a2uiaction` 事件处理器，了解按钮点击和表单提交的处理示例。

## 错误处理

需要处理的常见错误：

- **无效的 Surface ID**：在收到 `beginRendering`（v0.8）或 `createSurface`（v0.9）之前就引用了 surface
- **无效的 Component ID**：组件 ID 在同一个 surface 内必须唯一
- **无效的数据路径**：检查数据模型结构和 JSON Pointer 语法
- **Schema 校验失败**：确认消息格式与 A2UI 规范一致

参见 [samples/client/lit/shell/app.ts](https://github.com/a2ui-project/a2ui/tree/main/samples/client/lit/shell/app.ts) 中 `#sendMessage` 内的 `try...catch` 代码块，了解通信错误的处理示例。

## 下一步

- **[Quickstart](../quickstart.md)**：试用演示应用
- **[主题与样式](theming.md)**：自定义外观与风格
- **[定义你自己的 Catalog](defining-your-own-catalog.md)**：扩展组件目录
- **[Agent Development](agent-development.md)**：构建生成 A2UI 的代理
- **[Reference Documentation](../reference/messages.md)**：深入了解协议
