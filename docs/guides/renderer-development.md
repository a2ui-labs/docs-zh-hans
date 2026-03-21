# A2UI 渲染器实现指南

本文档概述了实现 A2UI 协议新渲染器时所需具备的功能，适用于正在构建新渲染器的开发者（例如 React、Flutter、iOS 等）。

!!! info "版本说明"
    本指南主要描述 v0.8 的消息流。v0.9 重命名了若干消息（`surfaceUpdate` → `updateComponents`、`dataModelUpdate` → `updateDataModel`、`beginRendering` → `createSurface`），并采用更扁平的组件格式。详情请参见 [v0.9 specification](../specification/v0.9-a2ui.md)。

## Web 渲染器：使用 `@a2ui/web-lib`（`web_core`）

如果你正在构建 Web 渲染器（React、Vue、Svelte 等），就不需要从零实现消息处理、状态管理或 schema 校验。**[`@a2ui/web-lib`](https://github.com/google/A2UI/tree/main/renderers/web_core)** 包（`web_core`）提供了所有维护中的 Lit、Angular 和 React 渲染器共享的、与框架无关的逻辑。

### `web_core` 提供什么

| 模块 | 功能 |
|--------|-------------|
| **`MessageProcessor`** | 处理 A2UI JSONL 流、分发消息、管理 surface 生命周期 |
| **`SurfaceModel` / `SurfaceGroupModel`** | 管理 surface、组件和数据模型的状态 |
| **`DataModel` / `DataContext`** | 解析数据绑定、基于路径查找、模板列表渲染 |
| **`ComponentModel`** | 组件树状态、邻接表 → 树的解析 |
| **Types & Schemas** | 所有 A2UI 组件、primitive、颜色、样式以及 JSON schema 校验的 TypeScript 类型 |
| **Expression parser** | 客户端函数求值（v0.9） |

### 维护中的渲染器如何使用它

三个 Web 渲染器都遵循同样的模式 - `web_core` 负责协议，渲染器负责 UI：

```typescript
// Types - 所有渲染器共享
import type * as Types from '@a2ui/web_core/types/types';
import type * as Primitives from '@a2ui/web_core/types/primitives';

// v0.8：消息处理与状态
import { A2uiMessageProcessor } from '@a2ui/web_core/data/model-processor';

// v0.9：消息处理、surface、catalog
import { MessageProcessor } from '@a2ui/web_core/v0_9';
import { SurfaceModel } from '@a2ui/web_core/v0_9';

// 样式与布局辅助工具
import * as Styles from '@a2ui/web_core/styles/index';
```

Your renderer only needs to:

1. **将 A2UI 组件类型映射到你的框架组件**（例如 `Text` → `<p>`，`Button` → `<button>`）
2. **订阅 `web_core` 的状态变化** 并重新渲染
3. **把用户动作** 再通过 `MessageProcessor` 转发回去

这个模式的可运行示例请参见 [React renderer](https://github.com/google/A2UI/tree/main/renderers/react)、[Lit renderer](https://github.com/google/A2UI/tree/main/renderers/lit) 和 [Angular renderer](https://github.com/google/A2UI/tree/main/renderers/angular)。

### 版本支持

`web_core` 同时导出 v0.8 和 v0.9 API：

- `@a2ui/web_core/v0_8` 或 `@a2ui/web_core`（默认）- 稳定的 v0.8
- `@a2ui/web_core/v0_9` - 带有 `createSurface`、自定义 catalog、客户端函数的 v0.9
- `@a2ui/web_core/v0_9/basic_catalog` - v0.9 基础 catalog 的表达式解析器和内置函数

!!! tip "从 `web_core` 开始"
    不使用 `web_core` 来构建 Web 渲染器，就意味着你要重写大约 3,000 行消息处理、状态管理和 schema 校验逻辑。除非你有明确理由要走不同路径，否则建议直接使用它。

---

## I. 核心协议实现清单

本节详细说明 A2UI 协议的基础机制。合规的渲染器必须实现这些系统，才能成功解析服务端流、管理状态并处理用户交互。

### 消息处理与状态管理

- **JSONL 流解析**：实现一个可以逐行读取流式响应，并把每一行解码为独立 JSON 对象的解析器。
- **消息分发器**：创建一个分发器，用于识别消息类型（`beginRendering`、`surfaceUpdate`、`dataModelUpdate`、`deleteSurface`）并路由到正确的处理器。
- **Surface 管理**：
  - 实现一个数据结构来管理多个 UI surface，每个 surface 通过自己的 `surfaceId` 键入。
  - 处理 `surfaceUpdate`：向指定 surface 的组件缓冲区中添加或更新组件。
  - 处理 `deleteSurface`：删除指定 surface 及其关联的所有数据和组件。
- **组件缓冲区（邻接表）**：
  - 对于每个 surface，维护一个组件缓冲区（例如 `Map<String, Component>`），按 `id` 存储所有组件定义。
  - 在渲染时能够通过解析容器组件中的 `id` 引用（`children.explicitList`、`child`、`contentChild` 等）重建 UI 树。
- **数据模型存储**：
  - 对于每个 surface，维护一个独立的数据模型存储（例如 JSON 对象或 `Map<String, any>`）。
  - 处理 `dataModelUpdate`：更新指定 `path` 上的数据模型。`contents` 会是邻接表格式（例如 `[{ "key": "name", "valueString": "Bob" }]`）。

### 渲染逻辑

- **渐进式渲染控制**：
  - 缓存所有进入的 `surfaceUpdate` 和 `dataModelUpdate` 消息，不要立刻渲染。
  - 处理 `beginRendering`：这个消息是明确触发某个 surface 初始渲染并设置根组件 ID 的信号。
    - 从指定的 `root` 组件 ID 开始渲染。
    - 如果提供了 `catalogId`，要确保使用相应的组件 catalog（如果省略，则默认使用标准 catalog）。
    - 应用该消息中提供的任何全局 `styles`（例如 `font`、`primaryColor`）。
- **数据绑定解析**：
  - 为组件属性中出现的 `BoundValue` 对象实现解析器。
  - 如果只有 `literal*` 值（`literalString`、`literalNumber` 等），就直接使用它。
  - 如果只有 `path`，就根据 surface 的数据模型来解析它。
  - 如果同时存在 `path` 和 `literal*`，先把该 `path` 上的数据模型更新为字面值，再把组件属性绑定到这个 `path`。
- **动态列表渲染**：
  - 对于带有 `children.template` 的容器，遍历位于 `template.dataBinding` 的数据列表（它会解析为数据模型中的一个列表）。
  - 对数据列表中的每一项，渲染 `template.componentId` 指定的组件，并让该项数据在模板内部可用于相对数据绑定。

### 客户端到服务端通信

- **事件处理**：
  - 当用户与某个定义了 `action` 的组件交互时，构建一个 `userAction` 负载。
  - 把 `action.context` 中的所有数据绑定都相对于数据模型进行解析。
  - 将完整的 `userAction` 对象发送到服务端的事件处理端点。
- **客户端能力上报**：
  - 在发送给服务器的 **每一条** A2A 消息中，都要作为元数据包含一个 `a2uiClientCapabilities` 对象。
  - 这个对象应通过 `supportedCatalogIds` 声明客户端支持的组件 catalog（例如包含标准 0.8 catalog 的 URI）。
  - 如果服务器支持，也可以选择提供 `inlineCatalogs`，用于临时的自定义组件定义。
- **错误上报**：实现一种机制，把 `error` 消息发送给服务器，以报告任何客户端侧错误（例如数据绑定失败、未知组件类型）。

## II. 标准组件目录清单

为了保证跨平台的一致用户体验，A2UI 定义了一套标准组件。你的客户端应该把这些抽象定义映射到对应的原生 UI widget。

### 基础内容

- **Text**：渲染文本内容。必须支持 `text` 上的数据绑定，以及用于样式的 `usageHint`（h1-h5、body、caption）。
- **Image**：从 URL 渲染图片。必须支持 `fit`（cover、contain 等）和 `usageHint`（avatar、hero 等）属性。
- **Icon**：从 catalog 指定的标准集合中渲染预定义图标。
- **Video**：为给定 URL 渲染视频播放器。
- **AudioPlayer**：为给定 URL 渲染音频播放器，可选带描述。
- **Divider**：渲染视觉分隔线，支持 `horizontal` 和 `vertical` 两个方向。

### 布局与容器

- **Row**：水平排列子项。必须支持 `distribution`（justify-content）和 `alignment`（align-items）。子项可以有 `weight` 属性来控制 flex-grow 行为。
- **Column**：垂直排列子项。必须支持 `distribution` 和 `alignment`。子项可以有 `weight` 属性来控制 flex-grow 行为。
- **List**：渲染一个可滚动的项目列表。必须支持 `direction`（`horizontal`/`vertical`）和 `alignment`。
- **Card**：一种把子内容视觉上分组的容器，通常带边框、圆角和/或阴影。只有一个 `child`。
- **Tabs**：显示一组标签页的容器。包含 `tabItems`，其中每一项都有 `title` 和 `child`。
- **Modal**：显示在主内容之上的对话框。由 `entryPointChild`（例如按钮）触发，在激活时展示 `contentChild`。

### 交互与输入组件

- **Button**：一个可点击元素，会触发 `userAction`。必须能够包含一个 `child` 组件（通常是 Text 或 Icon），并且其样式可以根据 `primary` 布尔值变化。
- **CheckBox**：可切换的复选框，反映一个布尔值。
- **TextField**：文本输入框。必须支持 `label`、`text`（值）、`textFieldType`（`shortText`、`longText`、`number`、`obscured`、`date`）以及 `validationRegexp`。
- **DateTimeInput**：用于选择日期和/或时间的专用输入控件。必须支持 `enableDate` 和 `enableTime`。
- **MultipleChoice**：用于从列表（`options`）中选择一个或多个选项的组件。必须支持 `maxAllowedSelections`，并把 `selections` 绑定到列表或单一值。
- **Slider**：用于从给定范围（`minValue`、`maxValue`）中选择数值（`value`）的滑块。
