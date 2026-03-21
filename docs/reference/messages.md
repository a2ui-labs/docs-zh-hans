# 消息类型

本参考文档详细说明所有 A2UI 消息类型。

## 消息格式

所有 A2UI 消息都是以 JSON Lines（JSONL）形式发送的 JSON 对象。每一行只包含一条消息。

=== "v0.8 消息类型"

    - `beginRendering` — 通知客户端开始渲染一个 surface
    - `surfaceUpdate` — 新增或更新组件
    - `dataModelUpdate` — 更新应用状态
    - `deleteSurface` — 删除一个 surface

=== "v0.9 消息类型"

    - `createSurface` — 创建一个 surface 并指定其 catalog
    - `updateComponents` — 新增或更新组件
    - `updateDataModel` — 更新应用状态
    - `deleteSurface` — 删除一个 surface

    所有 v0.9 消息都包含 `"version": "v0.9"` 字段。

---

## beginRendering (v0.8) / createSurface (v0.9)

通知客户端初始化并渲染一个 surface。

=== "v0.8 — `beginRendering`"

    ### Schema

    ```typescript
    {
      beginRendering: {
        surfaceId: string;      // Required: Unique surface identifier
        root: string;           // Required: The ID of the root component to render
        catalogId?: string;     // Optional: URL of component catalog
        styles?: object;        // Optional: Styling information
      }
    }
    ```

    ### 属性

    | 属性 | 类型 | 必需 | 说明 |
    | ----------- | ------ | -------- | --------------------------------------------------------------------------------------- |
    | `surfaceId` | string | ✅ | 该 surface 的唯一标识符。 |
    | `root`      | string | ✅ | 该 surface 的 UI 树根组件 `id`。 |
    | `catalogId` | string | ❌ | 组件 catalog 的标识符。若省略，则默认使用 v0.8 标准 catalog。 |
    | `styles`    | object | ❌ | UI 的样式信息，由 catalog 定义。 |

    ### 示例

    ```json
    {
      "beginRendering": {
        "surfaceId": "main",
        "root": "root-component"
      }
    }
    ```

    **使用自定义 catalog：**

    ```json
    {
      "beginRendering": {
        "surfaceId": "custom-ui",
        "root": "root-custom",
        "catalogId": "https://my-company.com/a2ui/v0.8/my_custom_catalog.json"
      }
    }
    ```

    必须在组件定义之后发送。客户端会在收到 `beginRendering` 之前缓存 `surfaceUpdate` 和 `dataModelUpdate` 消息。

=== "v0.9 — `createSurface`"

    ### Schema

    ```typescript
    {
      version: "v0.9";
      createSurface: {
        surfaceId: string;      // Required: Unique surface identifier
        catalogId: string;      // Required: URL of component catalog
        theme?: object;         // Optional: Theme configuration
        sendDataModel?: boolean; // Optional: Request client to send data model updates
      }
    }
    ```

    ### 属性

    | 属性 | 类型 | 必需 | 说明 |
    | --------------- | ------- | -------- | --------------------------------------------------------------- |
    | `surfaceId`     | string  | ✅ | 该 surface 的唯一标识符。 |
    | `catalogId`     | string  | ✅ | 组件 catalog 的标识符。 |
    | `theme`         | object  | ❌ | 主题配置（例如 `primaryColor`）。 |
    | `sendDataModel` | boolean | ❌ | 如果为 true，客户端会把数据模型变化发回服务器。 |

    ### 示例

    ```json
    {
      "version": "v0.9",
      "createSurface": {
        "surfaceId": "main",
        "catalogId": "https://a2ui.org/specification/v0_9/basic_catalog.json"
      }
    }
    ```

    在 v0.9 中，`createSurface` 取代了 `beginRendering`。根节点通过约定确定：`updateComponents` 中必须有一个组件的 `"id"` 为 `"root"`。`catalogId` 为必需项。

---

## surfaceUpdate (v0.8) / updateComponents (v0.9)

在 surface 内新增或更新组件。

=== "v0.8 — `surfaceUpdate`"

    ### Schema

    ```typescript
    {
      surfaceUpdate: {
        surfaceId: string;        // Required: Target surface
        components: Array<{       // Required: List of components
          id: string;             // Required: Component ID
          component: {            // Required: Wrapper for component data
            [ComponentType]: {    // Required: Exactly one component type
              ...properties       // Component-specific properties
            }
          }
        }>
      }
    }
    ```

    ### 属性

    | 属性 | 类型 | 必需 | 说明 |
    | ------------ | ------ | -------- | ------------------------------ |
    | `surfaceId`  | string | ✅ | 要更新的 surface ID。 |
    | `components` | array  | ✅ | 组件定义数组。 |

    ### Component 对象

    `components` 数组中的每个对象都必须包含：

    - `id`（string，必需）：该 surface 内唯一标识符
    - `component`（object，必需）：一个包装对象，且只能包含一个键，也就是组件类型（例如 `Text`、`Button`）

    ### 示例

    **单个组件：**

    ```json
    {
      "surfaceUpdate": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": {
              "Text": {
                "text": {"literalString": "Hello, World!"},
                "usageHint": "h1"
              }
            }
          }
        ]
      }
    }
    ```

    **多个组件（adjacency list）：**

    ```json
    {
      "surfaceUpdate": {
        "surfaceId": "main",
        "components": [
          {
            "id": "root",
            "component": {
              "Column": {
                "children": {"explicitList": ["header", "body"]}
              }
            }
          },
          {
            "id": "header",
            "component": {
              "Text": {
                "text": {"literalString": "Welcome"}
              }
            }
          },
          {
            "id": "body",
            "component": {
              "Card": {
                "child": "content"
              }
            }
          },
          {
            "id": "content",
            "component": {
              "Text": {
                "text": {"path": "/message"}
              }
            }
          }
        ]
      }
    }
    ```

    **更新已有组件：**

    ```json
    {
      "surfaceUpdate": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": {
              "Text": {
                "text": {"literalString": "Hello, Alice!"},
                "usageHint": "h1"
              }
            }
          }
        ]
      }
    }
    ```

    `id: "greeting"` 的组件会被更新，而不是重复创建。

    ### 使用说明

    - 必须在 `beginRendering` 消息中指定一个组件作为 `root`，以作为树根。
    - 组件采用 adjacency list 形式（带 ID 引用的扁平结构）。
    - 发送已存在 ID 的组件会更新该组件。
    - 子组件通过 ID 引用。
    - 组件可以增量添加（流式）。

=== "v0.9 — `updateComponents`"

    ### Schema

    ```typescript
    {
      version: "v0.9";
      updateComponents: {
        surfaceId: string;        // Required: Target surface
        components: Array<{       // Required: List of components
          id: string;             // Required: Component ID
          component: string;      // Required: Component type name
          ...properties           // Component-specific properties (flat)
        }>
      }
    }
    ```

    ### 属性

    | 属性 | 类型 | 必需 | 说明 |
    | ------------ | ------ | -------- | ------------------------------ |
    | `surfaceId`  | string | ✅ | 要更新的 surface ID。 |
    | `components` | array  | ✅ | 组件定义数组。 |

    ### Component 对象

    在 v0.9 中，组件结构更扁平：

    - `id`（string，必需）：该 surface 内唯一标识符
    - `component`（string，必需）：组件类型名称（例如 `"Text"`、`"Button"`）
    - 其余属性都直接位于组件对象顶层。

    ### 示例

    **单个组件：**

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": "Text",
            "text": "Hello, World!",
            "variant": "h1"
          }
        ]
      }
    }
    ```

    **多个组件：**

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "root",
            "component": "Column",
            "children": ["header", "body"]
          },
          {
            "id": "header",
            "component": "Text",
            "text": "Welcome"
          },
          {
            "id": "body",
            "component": "Card",
            "child": "content"
          },
          {
            "id": "content",
            "component": "Text",
            "text": {"path": "/message"}
          }
        ]
      }
    }
    ```

    **更新已有组件：**

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": "Text",
            "text": "Hello, Alice!",
            "variant": "h1"
          }
        ]
      }
    }
    ```

    ### 使用说明

    - 其中一个组件必须约定为 `"id": "root"`，作为树根（这是约定，而不是单独的消息字段）。
    - 组件类型是字符串（`"component": "Text"`），而不是包装对象。
    - 属性直接平铺在组件对象上，不再嵌套在类型键下。
    - 数据绑定使用 `{"path": "/pointer"}`（JSON Pointer）——键名与 v0.8 相同，但采用标准 JSON Pointer 路径。
    - 组件可以增量添加（流式）。

### 错误

| 错误 | 原因 | 解决方案 |
| ---------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| 未找到 surface      | `surfaceId` 不存在             | 确保同一个 surface 始终使用唯一且一致的 `surfaceId`。surface 会在第一次更新时隐式创建。 |
| 无效的组件类型 | 未知组件类型                 | 检查该组件类型是否存在于协商后的 catalog 中。                                                                 |
| 无效的属性       | 该类型不存在此属性   | 对照 catalog schema 进行验证。                                                                                         |
| 循环引用     | 组件把自己引用为子项 | 修复组件层级结构。                                                                                               |

---

## dataModelUpdate (v0.8) / updateDataModel (v0.9)

更新组件所绑定的数据模型。

=== "v0.8 — `dataModelUpdate`"

    ### Schema

    ```typescript
    {
      dataModelUpdate: {
        surfaceId: string;      // Required: Target surface
        path?: string;          // Optional: Path to a location in the model
        contents: Array<{       // Required: Data entries
          key: string;
          valueString?: string;
          valueNumber?: number;
          valueBoolean?: boolean;
          valueMap?: Array<{...}>;
        }>
      }
    }
    ```

    ### 属性

    | 属性 | 类型 | 必需 | 说明 |
    | ----------- | ------ | -------- | ---------------------------------------------------------------------------------------------------- |
    | `surfaceId` | string | ✅ | 要更新的 surface ID。 |
    | `path`      | string | ❌ | 数据模型中的位置路径（例如 `user`）。如果省略，则更新应用于根节点。 |
    | `contents`  | array  | ✅ | 作为 adjacency list 的数据条目数组。每个条目都有一个 `key` 和一个带类型的 `value*` 属性。 |

    ### `contents` 邻接列表

    `contents` 数组是键值对列表。数组中的每个对象都必须包含一个 `key`，以及且仅有一个 `value*` 属性（`valueString`、`valueNumber`、`valueBoolean` 或 `valueMap`）。这种结构对 LLM 友好，并避免从通用 `value` 字段推断类型时产生问题。

    ### 示例

    **初始化整个模型：**

    ```json
    {
      "dataModelUpdate": {
        "surfaceId": "main",
        "contents": [
          {
            "key": "user",
            "valueMap": [
              { "key": "name", "valueString": "Alice" },
              { "key": "email", "valueString": "alice@example.com" }
            ]
          },
          { "key": "items", "valueMap": [] }
        ]
      }
    }
    ```

    **更新嵌套属性：**

    ```json
    {
      "dataModelUpdate": {
        "surfaceId": "main",
        "path": "user",
        "contents": [
          { "key": "email", "valueString": "alice@newdomain.com" }
        ]
      }
    }
    ```

    这会修改 `/user/email`，而不会影响 `/user/name`。

    ### 使用说明

    - 数据模型是按 surface 分开的。
    - 当绑定数据发生变化时，组件会自动重新渲染。
    - 与其替换整个模型，不如优先对具体路径做粒度更细的更新。
    - 使用带类型的值字段（`valueString`、`valueNumber`、`valueBoolean`、`valueMap`）——对 LLM 友好，无需类型推断。
    - 任何数据转换（例如格式化日期）都必须在服务器端完成后再发送消息。

=== "v0.9 — `updateDataModel`"

    ### Schema

    ```typescript
    {
      version: "v0.9";
      updateDataModel: {
        surfaceId: string;      // Required: Target surface
        path?: string;          // Optional: JSON Pointer path (defaults to "/")
        value?: any;            // Optional: Value to set (omit to delete)
      }
    }
    ```

    ### 属性

    | 属性 | 类型 | 必需 | 说明 |
    | ----------- | ------ | -------- | ----------------------------------------------------- |
    | `surfaceId` | string | ✅ | 要更新的 surface ID。 |
    | `path`      | string | ❌ | JSON Pointer 路径（例如 `/user/email`）。默认值为 `/`（根）。 |
    | `value`     | any    | ❌ | 要设置的值。如果省略，则会删除 `path` 对应的键。 |

    ### 示例

    **初始化整个模型：**

    ```json
    {
      "version": "v0.9",
      "updateDataModel": {
        "surfaceId": "main",
        "path": "/",
        "value": {
          "user": {
            "name": "Alice",
            "email": "alice@example.com"
          },
          "items": []
        }
      }
    }
    ```

    **更新嵌套属性：**

    ```json
    {
      "version": "v0.9",
      "updateDataModel": {
        "surfaceId": "main",
        "path": "/user/email",
        "value": "alice@newdomain.com"
      }
    }
    ```

    ### 使用说明

    - v0.9 使用标准 JSON Pointer 路径和普通 JSON 值，不再使用带类型包装。
    - 如果省略 `path`，默认值为 `"/"`（根）。
    - `value` 可以是任意 JSON 类型（string、number、boolean、object、array、null）。省略则表示删除。
    - 比 v0.8 的 `contents` 邻接列表更简单，更接近标准 JSON Patch 语义。
    - 引用了 `{"path": "/user/email"}` 的组件会在该路径变化时自动更新。

---

## deleteSurface

删除一个 surface 及其所有组件和数据。

=== "v0.8 — `deleteSurface`"

    ### Schema

    ```typescript
    {
      deleteSurface: {
        surfaceId: string;        // Required: Surface to delete
      }
    }
    ```

    ### 示例

    ```json
    {
      "deleteSurface": {
        "surfaceId": "modal"
      }
    }
    ```

=== "v0.9 — `deleteSurface`"

    ### Schema

    ```typescript
    {
      version: "v0.9";
      deleteSurface: {
        surfaceId: string;        // Required: Surface to delete
      }
    }
    ```

    ### 示例

    ```json
    {
      "version": "v0.9",
      "deleteSurface": {
        "surfaceId": "modal"
      }
    }
    ```

### 属性

| 属性 | 类型 | 必需 | 说明 |
| ----------- | ------ | -------- | --------------------------- |
| `surfaceId` | string | ✅ | 要删除的 surface ID |

### 使用说明

- 删除与该 surface 相关联的所有组件
- 清除该 surface 的数据模型
- 客户端应将该 surface 从 UI 中移除
- 即使 surface 不存在也可以安全删除（无操作）
- 适用于关闭弹窗、对话框或页面跳转时
- 两个版本的结构相同（v0.9 只是增加了 `version` 字段）

---

## 消息顺序

### 要求

1. `beginRendering` 必须在该 surface 的初始 `surfaceUpdate` 消息之后发送。
2. `surfaceUpdate` 可以在 `dataModelUpdate` 之前或之后发送。
3. 不同 surface 的消息彼此独立。
4. 多条消息可以增量更新同一个 surface。

### 推荐顺序

=== "v0.8"

    ```jsonl
    { "surfaceUpdate":    { "surfaceId": "main", "components": [...] } }
    { "dataModelUpdate":  { "surfaceId": "main", "contents": {...} } }
    { "beginRendering":   { "surfaceId": "main", "root": "root-id" } }
    ```

=== "v0.9"

    ```jsonl
    { "version": "v0.9", "createSurface":    { "surfaceId": "main", "catalogId": "..." } }
    { "version": "v0.9", "updateComponents": { "surfaceId": "main", "components": [...] } }
    { "version": "v0.9", "updateDataModel":  { "surfaceId": "main", "path": "/", "value": {...} } }
    ```

### 渐进式构建

=== "v0.8"

    ```jsonl
    { "surfaceUpdate":   { "surfaceId": "main", "components": [...] } }  // Header
    { "surfaceUpdate":   { "surfaceId": "main", "components": [...] } }  // Body
    { "beginRendering":  { "surfaceId": "main", "root": "root-id" } }   // Render
    { "surfaceUpdate":   { "surfaceId": "main", "components": [...] } }  // Footer
    { "dataModelUpdate": { "surfaceId": "main", "contents": {...} } }    // Data
    ```

=== "v0.9"

    ```jsonl
    { "version": "v0.9", "createSurface":    { "surfaceId": "main", "catalogId": "..." } }
    { "version": "v0.9", "updateComponents": { "surfaceId": "main", "components": [...] } }  // Header
    { "version": "v0.9", "updateComponents": { "surfaceId": "main", "components": [...] } }  // Body + Footer
    { "version": "v0.9", "updateDataModel":  { "surfaceId": "main", "path": "/", "value": {...} } }
    ```

## 验证

=== "v0.8"

    验证对象：

    - **[server_to_client.json](https://a2ui.org/specification/v0_8/server_to_client.json)**：消息包封装 schema
    - **[standard_catalog_definition.json](https://a2ui.org/specification/v0_8/standard_catalog_definition.json)**：组件 schema

=== "v0.9"

    验证对象：

    - **[server_to_client.json](https://a2ui.org/specification/v0_9/server_to_client.json)**：消息包封装 schema
    - **[basic_catalog.json](https://a2ui.org/specification/v0_9/basic_catalog.json)**：组件 schema

## 延伸阅读

- **[组件画廊](components.md)**：所有可用组件类型
- **[数据绑定指南](../concepts/data-binding.md)**：了解数据绑定如何工作
- **[Agent 开发指南](../guides/agent-development.md)**：生成有效消息
