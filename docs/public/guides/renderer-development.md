# A2UI 渲染器实现指南

本文档概述了实现 A2UI 协议新渲染器时所需具备的功能，适用于正在构建新渲染器的开发者（例如 React、Flutter、iOS 等）。

> NOTE: 版本感知指南
>
> 本指南提供了 v0.8、v0.9.1（当前生产版本）和 v1.0（候选版本）的实现清单。请使用下方的标签页选择你要针对的版本。

## Web 渲染器：使用 `@a2ui/web_core`（`web_core`）

如果你正在构建 Web 渲染器（React、Vue、Svelte 等），就不需要从零实现消息处理、状态管理或 schema 校验。**[`@a2ui/web_core`](https://github.com/a2ui-project/a2ui/tree/main/renderers/web_core)** 包（`web_core`）提供了所有维护中的 Lit、Angular 和 React 渲染器共享的、与框架无关的逻辑。

### `web_core` 提供什么

| 模块 | 功能 |
|--------|-------------|
| **`MessageProcessor`** | 处理 A2UI JSONL 流、分发消息、管理 surface 生命周期 |
| **`SurfaceModel` / `SurfaceGroupModel`** | 管理 surface、组件和数据模型的状态 |
| **`DataModel` / `DataContext`** | 解析数据绑定、基于路径查找、模板列表渲染 |
| **`ComponentModel`** | 组件树状态、邻接表 → 树的解析 |
| **Types & Schemas** | 所有 A2UI 组件、primitive、颜色、样式以及 JSON schema 校验的 TypeScript 类型 |
| **Expression parser** | 客户端函数求值（v0.9+） |

### 维护中的渲染器如何使用它

三个 Web 渲染器都遵循同样的模式 - `web_core` 负责协议，渲染器负责 UI：

```typescript
// Types - 所有渲染器共享
import type * as Types from '@a2ui/web_core/types/types';
import type * as Primitives from '@a2ui/web_core/types/primitives';

// v0.8：消息处理与状态
import {A2uiMessageProcessor} from '@a2ui/web_core/data/model-processor';

// v0.9.1 / v1.0：消息处理、surface、catalog
import {MessageProcessor} from '@a2ui/web_core/v0_9';
import {SurfaceModel} from '@a2ui/web_core/v0_9';

// 样式与布局辅助工具
import * as Styles from '@a2ui/web_core/styles/index';
```

你的渲染器只需要：

1. **将 A2UI 组件类型映射到你的框架组件**（例如 `Text` → `<p>`，`Button` → `<button>`）
2. **订阅 `web_core` 的状态变化**并重新渲染
3. **转发用户 Action**，通过 `MessageProcessor` 回传

这个模式的可运行示例请参见 [React renderer](https://github.com/a2ui-project/a2ui/tree/main/renderers/react)、[Lit renderer](https://github.com/a2ui-project/a2ui/tree/main/renderers/lit) 和 [Angular renderer](https://github.com/a2ui-project/a2ui/tree/main/renderers/angular)。

### 版本支持

`web_core` 按版本导出对应的 API：

- `@a2ui/web_core/v0_8` - 稳定版 v0.8
- `@a2ui/web_core/v0_9` - 支持 v0.9/v0.9.1，包含 `createSurface`、自定义 catalog、客户端函数
- `@a2ui/web_core/v1_0` - 候选版 v1.0 支持，包括基于 RPC 的 Action 响应

> TIP: 从 `web_core` 开始
>
> 不使用 `web_core` 来构建 Web 渲染器，就意味着你要重写大约 3,000 行消息处理、状态管理和 schema 校验逻辑。除非你有明确理由要走不同路径，否则建议直接使用它。

---

## I. 核心协议实现清单

本节详细说明 A2UI 协议的基础机制。合规的渲染器必须实现这些系统，才能成功解析服务端流、管理状态并处理用户交互。

=== "v0.8"

    - **JSONL 流解析**：逐行读取流式响应，将每一行解码为独立的 JSON 对象。
    - **消息分发器**：识别消息类型（`beginRendering`、`surfaceUpdate`、`dataModelUpdate`、`deleteSurface`）并路由到正确的处理器。
    - **Surface 管理**：
        - 每个 surface 通过 `surfaceId` 键入。
        - 处理 `surfaceUpdate`：在该 surface 的缓冲区中添加/更新组件。
        - 处理 `deleteSurface`：删除该 surface 及其所有关联的数据和组件。
    - **组件缓冲**：
        - 为每个 surface 维护一个组件缓冲区（例如 `Map<String, Component>`）。
        - 通过解析 `id` 引用（`children.explicitList`、`child`、`contentChild` 等）来重建 UI 树。
    - **数据模型存储**：
        - 为每个 surface 维护数据模型状态。
        - 处理 `dataModelUpdate`：使用邻接表形式（`[{ "key": "name", "valueString": "Bob" }]`）更新指定路径上的值。
    - **渐进式渲染**：
        - 在收到 `beginRendering` 之前先缓存更新。
        - 收到 `beginRendering` 后，从指定的 `root` ID 开始渲染，并应用主题样式。
    - **数据绑定解析**：
        - 使用 `literalString` / `literalNumber` / `path` 来解析 `BoundValue` 对象。
    - **动态列表**：
        - 对于 `children.template`，遍历 `template.dataBinding` 处的数据列表，并使用 `template.componentId` 渲染组件。
    - **客户端到服务端**：
        - 将包含已解析路径上下文的 `userAction` 发送给服务端。
        - 在传输元数据中包含 `a2uiClientCapabilities`。

=== "v0.9.1 (Current)"

    - **JSONL 流解析**：逐行读取流式响应，将每一行解码为独立的 JSON 对象。
    - **消息分发器**：识别消息类型（`createSurface`、`updateComponents`、`updateDataModel`、`deleteSurface`）并路由到正确的处理器。
    - **MIME 类型校验**：基于标准化的 `application/a2ui+json` MIME 类型来拦截负载。
    - **Surface 管理**：
        - 每个 surface 通过 `surfaceId` 键入。
        - 处理 `createSurface`：创建 surface，绑定 `catalogId`，并注册 `theme` 和 `sendDataModel`。
        - 处理 `updateComponents`：使用 `"component": "Type"` 判别字段，以扁平格式添加/更新组件。
        - 处理 `deleteSurface`：删除该 surface 及其所有关联的数据和组件。
    - **组件缓冲**：
        - 为每个 surface 维护一个组件缓冲区（例如 `Map<String, Component>`）。
        - 通过解析容器组件 `children` 数组或 `child` 字段中的 ID 引用来重建 UI 树。
    - **数据模型存储**：
        - 为每个 surface 维护数据模型状态。
        - 处理 `updateDataModel`：使用标准 JSON 对象，以 upsert 语义更新指定路径上的数据。
    - **渐进式渲染**：
        - 一旦在 `updateComponents` 中解析出有效的根组件（ID 为 `root`），即立即渲染，无需等待特殊的渲染信号。
    - **数据绑定解析**：
        - 解析简化后的绑定值（可以是字面量，也可以是 `{"path": "..."}`）。
    - **动态列表**：
        - 对于子模板，遍历 `path` 处的数据数组，并渲染由 `componentId` 指定的模板。
    - **客户端函数**：
        - 对已注册的、由 catalog 定义的函数求值（例如 `formatString` 插值）。
    - **客户端到服务端**：
        - 发送包含已解析路径上下文的 `action`（取代 `userAction`）。
        - 如果请求了 `sendDataModel`，则自动在元数据中包含完整的客户端数据模型。
        - 如果 schema 校验失败，向服务端发送结构化的 `ValidationFailed` 错误消息。

=== "v1.0 (Candidate)"

    在 v0.9.1 所有要求的基础上，扩展以下内容：
    - **Surface 属性**：
        - 处理带有 `surfaceProperties`（由 `theme` 重命名而来）的 `createSurface`。surface schema 中不再支持自定义主品牌色。
    - **Action 响应（RPC）**：
        - 处理来自服务端的 `actionResponse` 消息，其中包含 `actionId` 以及返回的 `value` 或 `error`。
    - **客户端到服务端**：
        - 在 `action` 负载中生成并包含 `actionId`。
        - 当客户端期望获得响应时，可在 Action 上支持 `wantResponse: true`。
        - 如果使用 A2A，发送给服务端的每一条 A2A `Message` 都必须在其 `metadata` 字段中包含一个 `a2uiClientCapabilities` 对象。
    - **能力**：
        - 在能力交换过程中，用 `surfaceProperties` 取代 `theme` 对外暴露。

---

## II. 标准组件目录清单

为了保证跨平台的一致用户体验，A2UI 定义了一套标准组件。你的客户端应该把这些抽象定义映射到对应的原生 UI widget。

=== "v0.8"

    - **Text**：渲染文本。支持 `usageHint`（h1-h5、body、caption）。
    - **Image**：从 URL 渲染图片。支持 `fit` 和 `usageHint`（avatar、hero 等）。
    - **Icon**：渲染系统图标。
    - **Video**：渲染视频播放器。
    - **AudioPlayer**：渲染带描述的音频播放器。
    - **Divider**：渲染水平/垂直分隔线。
    - **Row** / **Column**：水平/垂直排列子项。支持 `distribution` 和 `alignment`。支持子项 `weight`。
    - **List**：渲染可滚动列表。
    - **Card**：带圆角和阴影的盒状布局。
    - **Tabs**：使用 `tabItems` 实现的标签页导航。
    - **Modal**：由 `entryPointChild` 触发、展示 `contentChild` 的弹出层。
    - **Button**：触发 `userAction` 的可点击按钮。支持 `primary` 变体。
    - **CheckBox**：布尔值复选框。
    - **TextField**：支持 `label`、`textFieldType`（`shortText`、`longText` 等）和 `validationRegexp` 的输入框。
    - **MultipleChoice**：支持 `options`、`maxAllowedSelections` 以及单选/多选值。
    - **Slider**：支持使用 `minValue`、`maxValue` 定义的范围。

=== "v0.9.1 (Current)"

    - **Text**：渲染文本。支持 `variant`（取代 `usageHint`）。
    - **Image**：从 URL 渲染图片。支持 `fit` 和 `variant`。
    - **Icon**：渲染系统图标。
    - **Video**：渲染视频播放器。
    - **AudioPlayer**：渲染带描述的音频播放器。
    - **Divider**：渲染水平/垂直分隔线。
    - **Row** / **Column**：水平/垂直排列子项。支持 `justify` 和 `align`。支持子项 `weight`。
    - **List**：渲染可滚动列表。
    - **Card**：带圆角和阴影的盒状布局。
    - **Tabs**：使用 `tabs` 实现的标签页导航。
    - **Modal**：由 `trigger` 触发、展示 `content` 的弹出层。
    - **Button**：触发 `action` 的可点击按钮。支持 `variant`（primary、borderless）。
    - **CheckBox**：布尔值复选框。
    - **TextField**：支持 `label`、`value`（取代 `text`）、`variant`（`shortText`、`longText` 等）和 `checks`（校验函数）的输入框。
    - **ChoicePicker**：（取代 MultipleChoice）支持 `options` 和 `variant`（`mutuallyExclusive`、`multipleSelection`）。
    - **Slider**：支持使用 `min`、`max`（取代 `minValue`、`maxValue`）定义的范围。

=== "v1.0 (Candidate)"

    在 v0.9.1 所有组件的基础上，扩展以下内容：
    - **Video**：支持 `posterUrl` 属性以展示预览图。
    - **TextField**：支持 `placeholder` 属性。
