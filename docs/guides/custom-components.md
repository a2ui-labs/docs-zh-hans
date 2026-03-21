# 自定义组件目录

通过定义 **自定义 catalog**，把你自己的组件和标准 A2UI 组件一起扩展到 A2UI 中。

## 为什么需要自定义 catalog？

A2UI 标准 catalog 提供了常见 UI 元素（按钮、文本框等），但你的应用可能还需要一些专用组件：

- **领域特定小部件**：股票行情、医疗图表、CAD 查看器
- **第三方集成**：Google Maps、支付表单、聊天小部件
- **品牌特定组件**：自定义日期选择器、商品卡片、仪表盘

**自定义 catalog** 是一组组件集合，可以包含：
- 标准 A2UI 组件（Text、Button、TextField 等）
- 你的自定义组件（GoogleMap、StockTicker 等）
- 第三方组件

你注册到客户端应用里的应该是整个 catalog，而不是单个组件。这样代理和客户端就能在保持安全性与类型安全的同时，约定一套共享且可扩展的组件集合。

## 自定义 catalog 的工作方式

1.  **客户端定义 catalog**：创建一个 catalog 定义，同时列出标准组件和自定义组件。
2.  **客户端注册 catalog**：把 catalog（以及它的组件实现）注册到客户端应用中。
3.  **客户端声明支持**：客户端告诉代理它支持哪些 catalog。
4.  **代理选择 catalog**：代理为某个 UI surface 选择一个 catalog。
5.  **代理生成 UI**：代理使用该 catalog 中按名称列出的组件来生成组件消息（v0.8 使用 `surfaceUpdate`，v0.9 使用 `updateComponents`）。

## 定义自定义 catalog

TODO：补充针对各个平台定义自定义 catalog 的详细指南。

**Web（Lit / Angular）：**

- 如何定义同时包含标准组件与自定义组件的 catalog
- 如何在 A2UI 客户端中注册 catalog
- 如何实现自定义组件类

**Flutter：**

- 如何使用 GenUI 定义自定义 catalog
- 如何注册自定义组件渲染器

**查看可运行示例：**

- [Lit samples](https://github.com/google/a2ui/tree/main/samples/client/lit)
- [Angular samples](https://github.com/google/a2ui/tree/main/samples/client/angular)
- [Flutter GenUI docs](https://docs.flutter.dev/ai/genui)

## 代理侧：使用来自自定义 catalog 的组件

一旦某个 catalog 在客户端注册完成，代理就可以在 `surfaceUpdate` 消息中使用其中的组件。

代理会通过 `beginRendering` 消息中的 `catalogId` 指定要使用哪个 catalog。

TODO：补充以下示例：

- 代理如何选择 catalog
- 代理如何引用 catalog 中的自定义组件
- catalog 版本管理如何工作

## 数据绑定与动作

自定义组件支持与标准组件相同的数据绑定和动作机制：

- **数据绑定**：自定义组件可以使用 JSON Pointer 语法把属性绑定到数据模型路径
- **动作**：自定义组件可以发出动作，由代理接收并处理

## 安全注意事项

创建自定义 catalog 和组件时：

1. **允许列表化组件**：catalog 中只注册你信任的组件
2. **验证属性**：始终验证来自代理消息的组件属性
3. **净化用户输入**：如果组件接受用户输入，在处理前先清洗
4. **限制 API 访问**：不要把敏感 API 或凭据暴露给自定义组件

TODO：补充详细的安全最佳实践和代码示例。

## 下一步

- **[主题与样式](theming.md)**：自定义组件的外观与风格
- **[Component Reference](../reference/components.md)**：查看全部标准组件
- **[Agent Development](agent-development.md)**：构建使用自定义组件的代理
