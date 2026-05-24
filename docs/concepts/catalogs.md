# A2UI 目录

## 概述

本指南定义了 A2UI 目录架构，并给出实现路线图。它解释了目录 schema 的结构，说明了如何使用预置的“基础目录”以及如何定义自己的应用专属目录，并细化了目录协商、版本管理和运行时验证的技术协议。

## 目录定义

目录是一个 [JSON Schema 文件](../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6)，用于描述智能体可以用来通过服务端驱动 UI 定义 A2UI surface 的组件、函数和主题。来自智能体的所有 A2UI JSON 都会根据所选目录进行验证。

[目录 JSON Schema](../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6) 如下：

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
    "required": [
      "catalogId"
    ],
    "additionalProperties": false
  }
}
```

## 目录策略

每个 A2UI surface 都由一个目录驱动。目录本质上就是一个 JSON Schema 文件，用来告诉智能体可以使用哪些组件、函数和主题。

无论你是在构建一个简单原型还是复杂的生产应用，要求都一样：你必须提供一个目录定义，供智能体用来表达 UI。

### 基础目录

为了帮助开发者快速开始，A2UI 团队维护了 [基础目录](../../specification/v0_9/catalogs/basic/catalog.json)。

这是一个预定义的目录文件，包含一组标准通用组件（按钮、输入框、卡片）和函数。它并不是某种特殊的目录“类型”；它只是我们已经编写好、并且配有开源渲染器的目录版本。

基础目录可以帮助你快速引导应用，或在不从零编写 schema 的情况下验证 A2UI 概念。它有意保持简洁，以便不同渲染器都能轻松实现。

由于 A2UI 的设计目标是让 LLM 在设计时或运行时都能生成 UI，我们并不认为可移植性要求多个客户端共享同一个标准化目录；LLM 可以为每个前端单独解释各自的目录。

[查看 A2UI v0.9 基础目录](../../specification/v0_9/catalogs/basic/catalog.json)

### 定义你自己的目录

虽然基础目录适合起步，但大多数生产应用都会定义自己的目录，以体现自己的设计系统。

通过定义自己的目录，你可以限制智能体只能使用你应用中实际存在的组件和视觉语言，而不是通用输入框或按钮。这个目录可以完全从头构建，也可以从基础目录导入定义以节省时间，例如在定义你自己的独特 Card 组件时，复用基础文本定义。

为了简单起见，我们建议直接构建与客户端设计系统一致的目录，而不是试图通过适配器把基础目录映射过去。由于 A2UI 面向 GenUI，我们预期 LLM 可以为不同客户端解释不同的目录。

[查看 Rizzcharts 目录示例](../../samples/agent/adk/rizzcharts/catalog_schemas/0.9/rizzcharts_catalog_definition.json)

### 建议

| 使用场景 | 建议 | 工作量 |
| :---------------------------------- | :----------------------------------------------------------------------------- | :----------------------------- |
| 将 A2UI 加到成熟前端    | 定义一个与现有设计系统一致的目录。                     | 中等                         |
| 将 A2UI 加到新应用/绿地应用 | 先使用基础目录，再随着应用演进迁移到自己的目录。 | 低（假设渲染器已存在） |

## 构建目录

目录是一个 JSON Schema 文件，需符合 [目录 schema](../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6)；它定义了智能体在构建 surface 时可以使用的组件、主题和函数。

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
      "required": [
        "message"
      ]
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

### 独立目录

A2UI 目录必须是独立的（不引用外部文件），以简化 LLM 推理和依赖管理。

虽然最终目录必须是独立文件，但在本地开发阶段，你仍然可以用模块化方式编写目录，借助指向外部文档的 JSON Schema `$ref`。在发布之前，请运行 `tools/build_catalog/assemble_catalog.py`，将所有外部文件引用打包成一个单独、独立的 JSON Schema 文件：

```bash
uv run tools/build_catalog/assemble_catalog.py [INPUTS ...] --output-name <OUTPUT_NAME> [--catalog-id <ID>] [--version <VERSION>] [--extend-basic-catalog] [--out-dir <DIR>] [--verbose]
```

其中：
- `inputs`：一个或多个 A2UI 组件目录 JSON 的路径或 URL。
- `--output-name`：必填，合并后目录的名称，例如 `my_merged_catalog`。如果省略 `.json` 扩展名，会自动补上。
- `--catalog-id`：输出文件的自定义 `catalogId`。默认值为 `urn:a2ui:catalog:<base_name>`。
- `--version`：用于官方目录回退的 A2UI 规范版本，可选 `0.9` 或 `0.10`，默认 `0.9`。
- `--extend-basic-catalog`：如果提供，则无论输入目录是否显式引用，都会自动把整个 `basic_catalog.json` 包含到根输出中。
- `--out-dir`，`-o`：生成后的目录保存目录，默认 `dist`。
- `--verbose`，`-v`：启用详细调试日志，便于排查问题。

### 组合与导入

你不需要从零定义所有内容。你可以定义一个目录，复用基础目录或其他目录中已有的组件，并重用现有渲染逻辑。

#### 示例：扩展基础目录

这个目录导入了基础目录的所有元素，并新增了一个 `SuggestionChips` 组件。

```json
{
  "$id": "https://github.com/.../hello_world_with_all_basic/v1/catalog.json",
  "components": {
    "allOf": [
      { "$ref": "basic_catalog_definition.json#/components" },
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
          "required": [ "suggestions" ]
        }
      }
    ]
  }
}
```

**发布前务必运行 `tools/build_catalog/assemble_catalog.py`，以解析外部 `$ref`。**

#### 示例：挑选部分组件

这个目录只从基础目录导入 `Text`，用于构建一个简单的 Popup surface。

```json
{
  "$id": "https://github.com/.../hello_world_with_some_basic/v1/catalog.json",
  "components": {
    "allOf": [
      { "$ref": "basic_catalog.json#/components/Text" },
      {
        "Popup": {
          "type": "object",
          "description": "A modal overlay that displays an icon and text.",
          "properties": {
            "text": { "$ref": "common_types.json#/$defs/ComponentId" }
          },
          "required": [ "text" ]
        }
      }
    ]
  }
}
```

**发布前务必运行 `tools/build_catalog/assemble_catalog.py`，以解析外部 `$ref`。**

### 实现渲染器

客户端渲染器通过把 schema 定义映射为实际代码来实现目录。

hello world 目录的示例 TypeScript 渲染器：

```typescript
import { Catalog, DEFAULT_CATALOG } from '@a2ui/angular';
import { inputBinding } from '@angular/core';

export const RIZZ_CHARTS_CATALOG = {
  ...DEFAULT_CATALOG, // 包含基础 catalog
  HelloWorldBanner: {
    type: () => import('./hello_world_banner').then((r) => r.HelloWorldBanner),
    bindings: ({ properties }) => [
      inputBinding('message', () => ('message' in properties && properties['message']) || undefined)
    ],
  },
} as Catalog;
```

以及 `hello_world_banner` 的实现：

```typescript
import { DynamicComponent } from '@a2ui/angular';
import { Component, Input } from '@angular/core';
