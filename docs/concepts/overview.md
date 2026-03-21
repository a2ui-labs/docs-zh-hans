# 核心概念

本节介绍 A2UI 的基础架构。理解这些概念，将帮助你构建高质量的智能体驱动界面。

## 全局视角

A2UI 围绕三个核心思想构建：

1. **流式消息**：UI 更新以 JSON 消息序列的方式，从智能体流向客户端
2. **声明式组件**：UI 以数据描述，而不是以代码编程实现
3. **数据绑定**：UI 结构与应用状态彼此分离，从而支持响应式更新

## 关键主题

### [数据流](data-flow.md)
解释消息如何从智能体流向最终渲染的 UI。内容包括餐厅预约流程的完整生命周期示例、传输方式（SSE、WebSockets、A2A）、渐进式渲染，以及错误处理。

### [组件结构](components.md)
介绍 A2UI 用来表示组件层级的 **adjacency list model**。你将了解为什么扁平列表比嵌套树更适合这类场景，如何使用静态与动态子组件，以及增量更新的最佳实践。

### [数据绑定](data-binding.md)
讲解组件如何通过 JSON Pointer 路径连接到应用状态。内容覆盖响应式组件、动态列表、输入绑定，以及“结构与状态分离”这一使 A2UI 强大的关键设计。

## 消息类型

=== "v0.8 (Stable)"

    - **`surfaceUpdate`**：定义或更新 UI 组件
    - **`dataModelUpdate`**：更新应用状态
    - **`beginRendering`**：通知客户端开始渲染
    - **`deleteSurface`**：删除一个 UI surface

=== "v0.9 (Draft)"

    - **`createSurface`**：创建新的 surface，并指定其 catalog
    - **`updateComponents`**：在 surface 中新增或更新 UI 组件
    - **`updateDataModel`**：更新应用状态
    - **`deleteSurface`**：删除一个 UI surface

    v0.9 将 surface 创建与渲染过程明确分离，`createSurface` 同时取代了 `beginRendering` 以及 `surfaceUpdate` 中隐含的 surface 创建逻辑。所有消息都包含 `version` 字段。

完整技术细节请参阅 [消息参考](../reference/messages.md)。
