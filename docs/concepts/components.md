# 组件与结构

A2UI 对组件层级采用 **邻接表模型**。与嵌套的 JSON 树不同，组件会以一个带 ID 引用的扁平列表来表示。

## 为什么使用扁平列表？

**传统的嵌套方式：**

- LLM 必须在一次生成中完成完全正确的嵌套
- 很难更新深层嵌套的组件
- 不适合增量流式输出

**A2UI 邻接表：**

- ✅ 结构扁平，LLM 更容易生成
- ✅ 可以增量发送组件
- ✅ 可按 ID 更新任意组件
- ✅ 结构与数据清晰分离

## 邻接表模型

=== "v0.8"

    ```json
    {
      "surfaceUpdate": {
        "components": [
          {
            "id": "root",
            "component": {
              "Column": {
                "children": { "explicitList": ["greeting", "buttons"] }
              }
            }
          },
          {
            "id": "greeting",
            "component": {
              "Text": { "text": { "literalString": "Hello" } }
            }
          },
          {
            "id": "buttons",
            "component": {
              "Row": {
                "children": { "explicitList": ["cancel-btn", "ok-btn"] }
              }
            }
          },
          {
            "id": "cancel-btn",
            "component": {
              "Button": {
                "child": "cancel-text",
                "action": { "name": "cancel" }
              }
            }
          },
          {
            "id": "cancel-text",
            "component": {
              "Text": { "text": { "literalString": "Cancel" } }
            }
          },
          {
            "id": "ok-btn",
            "component": {
              "Button": {
                "child": "ok-text",
                "action": { "name": "ok" }
              }
            }
          },
          {
            "id": "ok-text",
            "component": {
              "Text": { "text": { "literalString": "OK" } }
            }
          }
        ]
      }
    }
    ```

=== "v0.9"

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "root",
            "component": "Column",
            "children": ["greeting", "buttons"]
          },
          {
            "id": "greeting",
            "component": "Text",
            "text": "Hello"
          },
          {
            "id": "buttons",
            "component": "Row",
            "children": ["cancel-btn", "ok-btn"]
          },
          {
            "id": "cancel-btn",
            "component": "Button",
            "child": "cancel-text",
            "action": { "event": { "name": "cancel" } }
          },
          {
            "id": "cancel-text",
            "component": "Text",
            "text": "Cancel"
          },
          {
            "id": "ok-btn",
            "component": "Button",
            "child": "ok-text",
            "action": { "event": { "name": "ok" } }
          },
          {
            "id": "ok-text",
            "component": "Text",
            "text": "OK"
          }
        ]
      }
    }
    ```

    v0.9 使用更扁平的组件格式：`"component": "Text"` 取代嵌套的 `{"Text": {...}}`，而子组件则是普通数组，不再使用 `{"explicitList": [...]}`。

组件通过 ID 引用子组件，而不是通过嵌套关系。

## 组件基础

每个组件都有：

1. **ID**：唯一标识符（`"welcome"`）
2. **类型**：组件类型（`Text`、`Button`、`Card`）
3. **属性**：该类型特有的配置

=== "v0.8"

    ```json
    {
      "id": "welcome",
      "component": {
        "Text": {
          "text": { "literalString": "Hello" },
          "usageHint": "h1"
        }
      }
    }
    ```

=== "v0.9"

    ```json
    {
      "id": "welcome",
      "component": "Text",
      "text": "Hello",
      "variant": "h1"
    }
    ```

## 标准目录

A2UI 定义了一套按用途组织的标准组件目录：

- **布局**：Row、Column、List - 排列其他组件
- **展示**：Text、Image、Icon、Video、Divider - 展示信息
- **交互**：Button、TextField、CheckBox、DateTimeInput、Slider - 用户输入
- **容器**：Card、Tabs、Modal - 组合并组织内容

完整的组件画廊和示例请参见 [组件参考](../reference/components.md)。

## 静态子组件与动态子组件

**静态（`explicitList`）** - 固定的子 ID 列表：
```json
{
  "children": {
    "explicitList": ["back-btn", "title", "menu-btn"]
  }
}
```

**动态（`template`）** - 从数据数组生成子组件：
```json
{
  "children": {
    "template": {
      "dataBinding": "/items",
      "componentId": "item-template"
    }
  }
}
```

对于 `/items` 中的每一项，都渲染 `item-template`。更多细节请参见 [数据绑定](data-binding.md)。

## 通过值填充

组件有两种取值方式：

- **字面值** - 固定值：`{"text": {"literalString": "Welcome"}}`
- **数据绑定** - 来自数据模型：`{"text": {"path": "/user/name"}}`

LLM 可以直接生成带字面值的组件，也可以把它们绑定到数据路径上来呈现动态内容。

## 组合为 Surface

组件会组合成 **surface**（界面）：

=== "v0.8"

    1. LLM 通过 `surfaceUpdate` 生成组件定义
    2. LLM 通过 `dataModelUpdate` 填充数据
    3. LLM 通过 `beginRendering` 发出渲染信号
    4. 客户端将所有组件渲染为原生控件

=== "v0.9"

    1. LLM 通过 `createSurface` 创建 surface（指定 catalog）
    2. LLM 通过 `updateComponents` 生成组件定义
    3. LLM 通过 `updateDataModel` 填充数据
    4. 客户端将所有组件渲染为原生控件

surface 是一个完整、连贯的 UI（表单、仪表盘、聊天界面等）。

## 增量更新

- **新增** - 发送带新 ID 的组件定义
- **更新** - 发送具有现有 ID 且属性更新后的组件定义
- **移除** - 更新父组件的 `children` 列表，排除被移除的 ID

扁平结构让所有更新都变成简单的基于 ID 的操作。

## 自定义组件

除了标准目录之外，客户端还可以为领域特定需求定义自定义组件：

- **如何做**：在渲染器中注册自定义组件类型
- **是什么**：图表、地图、自定义可视化、专用小部件
- **安全性**：自定义组件仍然属于客户端受信任的目录

自定义组件会从客户端渲染器 _公布_ 给 LLM。随后 LLM 就可以在标准目录之外继续使用它们。

实现细节请参见 [自定义组件指南](../guides/custom-components.md)。

## 最佳实践

1. **使用描述性 ID**：用 `"user-profile-card"`，不要用 `"c1"`
2. **保持层级浅**：避免过深嵌套
3. **把结构与内容分离**：使用数据绑定，而不是字面值
4. **通过模板复用**：一个模板，通过动态子项生成多个实例
