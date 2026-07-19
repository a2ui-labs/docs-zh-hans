# A2UI 目录

## 概述

本指南定义了 A2UI 目录架构，并给出实现路线图。它解释了目录 schema 的结构，说明了如何使用预置的“基础目录”以及如何定义自己的应用专属目录，并细化了目录协商、版本管理和运行时验证的技术协议。

## 目录定义

目录是一个 [JSON Schema 文件](../../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6)，用于描述智能体可以用来通过服务端驱动 UI 定义 A2UI surface 的组件、函数和主题。来自智能体的所有 A2UI JSON 都会根据所选目录进行验证。

[目录 JSON Schema](../../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6) 如下：

```json
{
  "Catalog": {
    "type": "object",
    "description": "A collection of component and function definitions.",
    "properties": {
      "catalogId": {
        "type": "string",
        "description": "Unique identifier for this catalog."
      },
      "components": {
        "type": "object",
        "description": "Definitions for UI components supported by this catalog.",
        "additionalProperties": {
          "$ref": "https://json-schema.org/draft/2020-12/schema"
        }
      },
      "functions": {
        "type": "array",
        "description": "Definitions for functions supported by this catalog.",
        "items": {
          "$ref": "#/$defs/FunctionDefinition"
        }
      },
      "theme": {
        "title": "A2UI Theme",
        "description": "A schema that defines a catalog of A2UI theme properties.",
        "type": "object",
        "additionalProperties": {
          "$ref": "https://json-schema.org/draft/2020-12/schema"
        }
      }
    },
    "required": ["catalogId"],
    "additionalProperties": false
  }
}
```

## 目录策略

每个 A2UI surface 都由一个目录驱动。目录本质上就是一个 JSON Schema 文件，用来告诉智能体可以使用哪些组件、函数和主题。

无论你是在构建一个简单原型还是复杂的生产应用，要求都一样：你必须提供一个目录定义，供智能体用来表达 UI。

### 基础目录

为了帮助开发者快速开始，A2UI 团队维护了 [基础目录](../../../specification/v0_9/catalogs/basic/catalog.json)。

这是一个预定义的目录文件，包含一组标准通用组件（按钮、输入框、卡片）和函数。它并不是某种特殊的目录“类型”；它只是我们已经编写好、并且配有开源渲染器的目录版本。

基础目录可以帮助你快速引导应用，或在不从零编写 schema 的情况下验证 A2UI 概念。它有意保持简洁，以便不同渲染器都能轻松实现。

由于 A2UI 的设计目标是让 LLM 在设计时或运行时都能生成 UI，我们并不认为可移植性要求多个客户端共享同一个标准化目录；LLM 可以为每个前端单独解释各自的目录。

[查看 A2UI v0.9 基础目录](../../../specification/v0_9/catalogs/basic/catalog.json)

### 定义你自己的目录

虽然基础目录适合起步，但大多数生产应用都会定义自己的目录，以体现自己的设计系统。

通过定义自己的目录，你可以限制智能体只能使用你应用中实际存在的组件和视觉语言，而不是通用输入框或按钮。这个目录可以完全从头构建，也可以从基础目录导入定义以节省时间，例如在定义你自己的独特 Card 组件时，复用基础文本定义。

为了简单起见，我们建议直接构建与客户端设计系统一致的目录，而不是试图通过适配器把基础目录映射过去。由于 A2UI 面向 GenUI，我们预期 LLM 可以为不同客户端解释不同的目录。

[查看 Rizzcharts 目录示例](../../../samples/community/agent/adk/rizzcharts/catalog_schemas/0.9/rizzcharts_catalog_definition.json)

### 建议

| 使用场景 | 建议 | 工作量 |
| :---------------------------------- | :----------------------------------------------------------------------------- | :----------------------------- |
| 将 A2UI 加到成熟前端 | 定义一个与现有设计系统一致的目录。 | 中等 |
| 将 A2UI 加到新应用/绿地应用 | 先使用基础目录，再随着应用演进迁移到自己的目录。 | 低（假设渲染器已存在） |

## 构建目录

目录是一个 JSON Schema 文件，需符合 [目录 schema](../../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6)；它定义了智能体在构建 surface 时可以使用的组件、主题和函数。

### 示例：最小目录

下面是一个只定义单个组件的简单目录。

```json
{
  "$id": "https://github.com/.../hello_world/v1/catalog.json",
  "components": {
    "HelloWorldBanner": {
      "type": "object",
      "description": "A simple banner greeting.",
      "properties": {
        "message": {
          "type": "string",
          "description": "The banner text."
        },
        "backgroundColor": {
          "type": "string",
          "default": "#f0f0f0"
        }
      },
      "required": ["message"]
    }
  }
}
```

当智能体使用该目录时，会生成一个严格符合该结构的 payload：

```json
[
  {
    "version": "v0.9",
    "createSurface": {
      "surfaceId": "hello-world-surface",
      "catalogId": "https://github.com/.../hello_world/v1/catalog.json"
    }
  },
  {
    "version": "v0.9",
    "updateComponents": {
      "surfaceId": "hello-world-surface",
      "components": [
        {
          "id": "root",
          "component": "HelloWorldBanner",
          "message": "Hello, world! Welcome to your first catalog.",
          "backgroundColor": "#4CAF50"
        }
      ]
    }
  }
]
```

### 目录链接

A2UI 目录必须是独立的（不引用外部文件），以简化 LLM 推理和依赖管理。

虽然最终目录必须是独立文件，但在本地开发阶段，你仍然可以借助指向外部文档的 JSON Schema `$ref`，以模块化方式编写目录。

为了自动完成这些外部文件引用的打包与注册，这一目录注册过程被称为 **“链接”（Linking）**，并整合到一个统一的、跨平台的 Node.js 脚本（**`register-catalogs.js`**）中。

这个链接脚本被原生封装在 **Xcode Build Phases**（用于 iOS/macOS 客户端构建）和 **Gradle 任务**（用于 Android 客户端构建）中，以便在应用构建阶段无缝地编译、聚合并链接静态与动态 schema。

### 组合与导入

你不需要从零定义所有内容。你可以定义一个目录，复用基础目录或其他目录中已有的组件，并重用现有渲染逻辑。

#### 示例：扩展基础目录

这个目录导入了基础目录的所有元素，并新增了一个 `SuggestionChips` 组件。

```json
{
  "$id": "https://github.com/.../hello_world_with_all_basic/v1/catalog.json",
  "components": {
    "allOf": [
      {"$ref": "basic_catalog_definition.json#/components"},
      {
        "SuggestionChips": {
          "type": "object",
          "description": "A list of suggested prompts",
          "properties": {
            "suggestions": {
              "type": "array",
              "description": "The suggested prompts."
            }
          },
          "required": ["suggestions"]
        }
      }
    ]
  }
}
```

**请务必在编译期间使用你所在平台的 Xcode Build Phase 或 Gradle 任务（运行 `register-catalogs.js`）来链接并解析外部引用。**

#### 示例：挑选部分组件

这个目录只从基础目录导入 `Text`，用于构建一个简单的 Popup surface。

```json
{
  "$id": "https://github.com/.../hello_world_with_some_basic/v1/catalog.json",
  "components": {
    "allOf": [
      {"$ref": "catalogs/basic/catalog.json#/components/Text"},
      {
        "Popup": {
          "type": "object",
          "description": "A modal overlay that displays an icon and text.",
          "properties": {
            "text": {"$ref": "common_types.json#/$defs/ComponentId"}
          },
          "required": ["text"]
        }
      }
    ]
  }
}
```

**请务必在编译期间使用你所在平台的 Xcode Build Phase 或 Gradle 任务（运行 `register-catalogs.js`）来链接并解析外部引用。**

### 实现渲染器

客户端渲染器通过把 schema 定义映射为实际代码来实现目录。

hello world 目录的示例 TypeScript 渲染器：

```typescript
import {Catalog, DEFAULT_CATALOG} from '@a2ui/angular';
import {inputBinding} from '@angular/core';

export const RIZZ_CHARTS_CATALOG = {
  ...DEFAULT_CATALOG, // Include the basic catalog
  HelloWorldBanner: {
    type: () => import('./hello_world_banner').then(r => r.HelloWorldBanner),
    bindings: ({properties}) => [
      inputBinding(
        'message',
        () => ('message' in properties && properties['message']) || undefined,
      ),
    ],
  },
} as Catalog;
```

以及 `hello_world_banner` 的实现：

```typescript
import {DynamicComponent} from '@a2ui/angular';
import {Component, Input} from '@angular/core';

@Component({
  selector: 'hello-world-banner',
  imports: [],
  template: `
    <div>
      <h2>Hello World Banner</h2>
      <p>{{ message }}</p>
    </div>
  `,
})
export class HelloWorldBanner extends DynamicComponent {
  @Input() message?: string;
}
```

你可以在 [Orchestrator 示例](../../../samples/community/client/angular/projects/orchestrator/src/a2ui-catalog/catalog.ts) 中查看一个可运行的客户端渲染器示例。

## A2UI 目录协商

由于客户端和智能体可以支持多个目录，它们必须通过目录协商握手来约定使用哪个目录。

### 第 1 步：智能体广播其支持的目录（可选）

智能体可以选择性地广播它能够使用哪些目录（例如在 A2A Agent Card 中）。这仅作为参考信息：它有助于客户端了解智能体是否支持其特定功能，但客户端不必依赖它。

以下示例展示了一个 A2A AgentCard，声明该智能体支持 basic 和 rizzcharts 目录：

```json
{
  "name": "Ecommerce Dashboard Agent",
  "description": "This agent visualizes ecommerce data...",
  "capabilities": {
    "extensions": [
      {
        "uri": "https://a2ui.org/a2a-extension/a2ui/v0.8",
        "description": "Provides agent driven UI using the A2UI JSON format.",
        "params": {
          "supportedCatalogIds": [
            "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json",
            "https://github.com/.../rizzcharts_catalog_definition.json"
          ]
        }
      }
    ]
  }
}
```

### 第 2 步：客户端广播其支持的目录（必需）

客户端会在每条消息的 metadata 中，按偏好顺序向智能体发送一份 supportedCatalogIds 列表。这能准确告诉智能体，客户端当前已经准备好渲染哪些内容。

以下示例展示了在 metadata 中包含 supportedCatalogIds 的 A2A 消息：

```json
{
  "parts": [
    {
      "text": "What is the current status of my flight?"
    }
  ],
  "metadata": {
    "a2uiClientCapabilities": {
      "supportedCatalogIds": [
        "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json",
        "https://github.com/.../rizzcharts_catalog_definition.json"
      ]
    }
  }
}
```

### 第 3 步：智能体选择

当智能体创建一个新的 surface 时，它会从客户端的 `supportedCatalogIds` 列表中选择最匹配的目录。这个选择在该 surface 的整个生命周期内保持锁定。如果找不到兼容的目录，智能体将不会发送 UI。

以下示例展示了智能体发送的 A2UI 消息，其中定义了某个 surface 使用的 catalog_id：

```json
{
  "createSurface": {
    "surfaceId": "salesDashboard",
    "catalogId": "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json"
  }
}
```

## 目录命名与版本管理

A2UI 组件目录需要进行版本管理，因为目录定义通常是在编译期内置的，因此智能体生成的内容与客户端能够渲染的内容之间的任何不匹配都可能影响 UI。

### CatalogId 命名约定

`catalogId` 是一个用于客户端与智能体之间协商的唯一文本标识符。

- **格式**：虽然 `catalogId` 在技术上只是一个字符串，但 A2UI 的约定是使用 **URI**（例如 `https://example.com/catalogs/mysurface/v1/catalog.json`）。
- **目的**：我们使用 URI，是为了让该 ID 具有全局唯一性，并方便开发者在浏览器中直接查看。
- **不涉及运行时获取**：这个 URI 并不意味着智能体或客户端会在运行时下载该目录。**目录定义必须事先（即在编译/部署时）就已被智能体和客户端知晓**。这个 URI 仅作为一个稳定的标识符使用。

### 版本管理准则

为了在不破坏旧版客户端或智能体的前提下支持持续演进，A2UI 会根据变更是否**可以被安全忽略**，对目录更新进行分类。

虽然标准 JSON 解析器会忽略未知字段，但在服务端驱动 UI 中丢弃某个组件，可能会导致其整个视图树都被丢弃。为了在安全性和灵活性之间取得平衡，更新被划分为**破坏性（Breaking）**和**非破坏性（Non-Breaking）**两类，并依靠**优雅降级（Graceful Degradation）**来消化版本差异。

- **破坏性变更（需要提升主版本号）**  
  任何以旧版客户端无法安全忽略的方式改变结构的变更，都需要提升 `catalogId` URI 中的**主（Major）**版本号（例如从 `v1` 升级到 `v2`）。
    - **新增容器组件**：例如新增一个 `Grid` 或 `Accordion` 组件。如果旧版客户端忽略了某个容器，它会丢弃该容器的全部子节点，从而破坏 UI 树。
    - **移除容器组件**：例如移除一个 `Grid` 或 `Accordion` 组件。如果旧版智能体仍在使用该容器，客户端会忽略它，并丢弃其全部子节点，从而破坏 UI 树。
    - **更改字段类型**：例如把某个属性从 `string` 改为 `object`。这会导致旧版客户端上的 JSON Schema 验证失败。
    - **新增必需属性**：且没有默认值，因为旧版智能体不知道需要发送该字段。

- **非破坏性变更（在同一主版本号下允许）**  
  可以被安全忽略、或能够优雅降级而不破坏布局或数据模型的变更，可以保持在当前版本。
    - **新增叶子组件（非容器）**：例如新增 `Badge` 或 `Tooltip`。如果被忽略，布局仍保持完整。
    - **新增可选属性**：例如在 Card 上新增 `subtitle`。
    - **移除属性**：如果智能体不再发送某个属性，客户端可以安全地忽略它。
    - **新增函数或样式**：这些通常可以被忽略，而不会改变组件的语义。
    - **元数据变更**：更新 `description` 字段或修正文档中的拼写错误，不需要升级版本号，也不会影响运行时行为。

### 优雅降级

**非破坏性变更依赖于优雅降级。** 如果智能体在旧版客户端上使用了一个新组件/属性，客户端**必须**优雅地处理它（例如忽略它，或渲染文本回退，或显示“不支持”占位符），而不是崩溃。客户端也可以将验证错误报告给智能体，从而让智能体自我纠正并自动降级 UI。

#### 优雅降级示例

以下是在实践中如何处理目录版本不匹配的情况：

- **旧版 iOS 客户端使用的目录比智能体所用的更旧**
    - 智能体发送了一个旧版 iOS 客户端不认识的新组件 `Badge`。客户端会为其渲染一个通用文本框占位符或安全的文字描述，同时保持界面其余部分正常可用。
    - 智能体在 `Button` 上发送了一个旧版客户端不认识的新属性 `badge`。客户端会安全地忽略它，并渲染标准按钮。
    - 智能体不再发送在后续目录版本中被移除的 `Facepile` 组件。这不会给客户端带来任何问题。

- **Web 客户端先于智能体推出了新的目录版本**
    - Web 客户端支持新的 `Badge` 组件，但智能体还不知道这个组件。
    - Web 客户端移除了 `Button` 上的 `badge` 属性，因此如果智能体发送该属性，客户端会忽略它。
    - Web 客户端为 `Button` 新增了智能体尚不知道的新样式。这同样不会造成任何问题，因为智能体不会使用它们。

### 使用 CatalogId 进行版本管理

我们建议在 catalogId 中包含版本号。这样就可以借助 A2UI 目录协商，在迁移期间同时支持多个版本，从而确保零停机。

**推荐模式：**

| 变更类型 | URI 示例 | 说明 |
| :----------- | :----------------------------- | :------------------------------------------------------------ |
| **当前版本** | .../rizzcharts/v1/catalog.json | 1.x 版本。支持 1.x 分支中的所有增量式更新。 |
| **破坏性变更** | .../rizzcharts/v2/catalog.json | 引入破坏性结构变更的新 schema。 |

### 处理迁移

要在不破坏现有活动智能体的前提下升级目录，可以使用 A2UI 目录协商：

1. **客户端更新**：客户端把自己的 supportedCatalogIds 列表更新为同时包含新旧两个版本（例如 [".../v2/...", ".../v1/..."]）。
2. **智能体更新**：智能体使用 v2 schema 重新构建。当它们发现客户端支持 v2 时，会优先选择 v2。
3. **旧版支持**：尚未重新构建的旧版智能体会继续在客户端列表中匹配 v1，从而保持正常运行。

## A2UI Schema 验证与回退

为了确保稳定的用户体验，A2UI 采用了一种两阶段验证策略。这种“纵深防御（defense in depth）”方法能够尽早捕获错误，同时确保客户端在面对意外负载时依然保持健壮。

### 两阶段验证

1. **智能体侧（发送前）**：在传输任何 UI 负载之前，智能体运行时会先根据目录定义对生成的 JSON 进行验证。
    - 目的：在源头捕获虚构（hallucinated）的属性或格式错误的结构。
    - 结果：如果验证失败，智能体可以尝试修复或重新生成 A2UI JSON，也可以进行优雅降级，例如在对话式应用中回退为纯文本。
2. **客户端侧**：收到负载后，客户端库会根据其本地的目录定义对 JSON 进行验证。
    - 目的：安全性与稳定性。这确保了在用户设备上执行的代码严格符合预期的约定，从而防范版本不匹配或被篡改的智能体输出。
    - 结果：此处发生的失败会通过“error”客户端消息报告给智能体。

### 优雅降级

即使某个负载通过了 schema 验证，渲染器在运行时仍可能遇到问题（例如缺失的资源、尚未加载的组件实现，或特定平台的限制）。

客户端在遇到这些错误时不应崩溃，而应采用优雅降级：

- **未知组件**：如果某个组件在 schema 中是已知的，但渲染器尚未实现它，则渲染一个“安全”的回退内容（例如显示该组件调试名称的通用卡片），或直接跳过渲染该节点。
- **文本回退**：如果整个 surface 都无法渲染，则显示原始文本描述（如果有的话），或显示一条通用错误信息：*“该界面无法显示。”*

### 客户端到服务端的错误报告

当客户端检测到验证错误或运行时故障时，可以将其报告给智能体。这样，智能体系统就可以为开发者记录该故障，或据此调整其后续行为。

客户端会使用标准的 A2UI 客户端到服务端事件 Schema，发送一个 `VALIDATION_FAILED` 事件。

以下示例展示了客户端报告缺失必需字段的情况：

```json
{
  "version": "v0.9",
  "error": {
    "code": "VALIDATION_FAILED",
    "surfaceId": "flight-status-card-123",
    "path": "/components/FlightCard/flightNumber",
    "message": "Missing required property 'flightNumber' in component 'FlightCard'."
  }
}
```

## 内联目录

支持客户端在运行时发送内联目录，但不建议在生产环境中使用。更多详情请参见[这里](../../../specification/v0_9/docs/a2ui_protocol.md#client-capabilities--metadata)。
