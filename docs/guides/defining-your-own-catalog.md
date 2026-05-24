# 定义你自己的 Catalog

虽然 [Basic Catalog](../../specification/v0_9/catalogs/basic/catalog.json) 很适合入门和引导应用启动，但多数生产应用都会定义自己的 catalog，以反映自身的设计系统。

通过定义自己的 catalog，你可以把智能体限制在应用中真实存在的组件和视觉语言范围内，而不是让它使用通用输入框或按钮。

## 为什么要定义自己的 Catalog？

每个 A2UI surface 都由一个 **Catalog** 驱动。Catalog 本质上是一个 JSON Schema 文件，用来告诉智能体可以使用哪些组件、函数和主题。

定义自己的 catalog 有以下好处：

- **与设计系统对齐**：限制智能体只使用你的应用中真实存在的组件和视觉语言。
- **安全性与类型安全**：你在客户端应用中注册完整 catalog，确保只有可信组件会被渲染。
- **不需要 Mapper**：建议构建直接反映客户端设计系统的 catalog，而不是尝试通过适配器把通用 catalog（如 Basic Catalog）映射过去。

Basic Catalog 只是一个示例，并且刻意保持精简，以便不同 renderer 都能轻松实现。

## 工作方式

1.  **定义 Catalog**：创建 catalog 定义（JSON Schema），列出应用支持的组件、函数和样式。
2.  **注册 Catalog**：在客户端应用中注册 catalog 及其对应的组件实现（renderer）。
3.  **声明支持能力**：客户端告知智能体它支持哪些 catalog（通过 `supportedCatalogIds`）。
4.  **智能体选择 Catalog**：智能体为给定 UI surface 选择一个 catalog（通过创建消息中的 `catalogId`，例如 `createSurface`）。
5.  **智能体生成 UI**：智能体使用该 catalog 中按名称定义的组件来生成组件消息。

## 实现指南

建议创建能直接映射到现有组件库的 catalog。

=== "Web（Lit / Angular / React）"

    要在 Web 上实现自己的 catalog：

    - 创建包含组件定义的 JSON Schema。
    - 在所选 Web renderer 中创建自己的 `Component` 对象和 `Catalog` 对象。
        - 将 schema 或引用 ID 提供给智能体。

    _各框架的详细指南即将推出。_

=== "Flutter"

    要在 Flutter 中实现自己的 catalog：

    - 定义描述 widget 属性的 JSON Schema。
    - 使用自定义 renderer 将 schema 映射到 Flutter widget。

    *详细 Flutter 集成指南即将推出。*

## 安全注意事项

定义和注册 catalog 时：

1.  **组件白名单**：只在 catalog 定义中注册你信任的组件。除非有严格控制，否则不要暴露会提供危险能力的组件（例如执行任意脚本）。
2.  **校验属性**：始终校验来自智能体消息的组件属性，确保它们符合预期类型约束。
3.  **清理文本**：除非已经建立安全边界，否则避免渲染未经清理的智能体提供内容。

## 下一步

- **[主题与样式](theming.md)**：自定义组件的外观和体验。
- **[组件参考](../reference/components.md)**：探索可能可复用的标准类型。
- **[Agent 开发](agent-development.md)**：构建会与你的 Catalog 交互的智能体。
