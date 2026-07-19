# 编写自定义组件

了解如何在 A2UI 中定义、实现和注册自定义组件。本指南以 `rizzcharts` 示例为例，重点说明如何围绕你的 Angular 代码来编写组件。

## 概览

编写一个新组件主要包含四个步骤：

1.  **定义 Catalog Schema**：在 JSON Schema 中指定组件属性和类型。
2.  **定义组件（客户端）**：使用你的框架（例如 Angular）实现 UI。
3.  **注册到 Renderer（客户端）**：将组件加入客户端 catalog。
4.  **从 Agent 调用**：通过 `send_a2ui_json_to_client` 指示 agent 使用该组件。

---

## 1. 定义 Catalog Schema

Catalog schema 定义了你的 catalog API。它列出可用组件及其属性，agent 会用这些信息构造 UI payload。

**这个 schema 是客户端与服务端（agent）之间的契约。** 两端必须对该 schema 达成一致，渲染才能工作。客户端声明自己支持哪些 catalog，服务端选择兼容的 catalog。关于这个握手如何工作，见 [A2UI Catalog Negotiation](../concepts/catalogs.md#a2ui-catalog-negotiation)。

在 [`rizzcharts`](../../../samples/community/agent/adk/rizzcharts/python/README.md) 示例中，catalog schema 定义在 [`rizzcharts_catalog_definition.json`](../../../samples/community/agent/adk/rizzcharts/catalog_schemas/0.9/rizzcharts_catalog_definition.json)。

下面是 `Chart` 组件的 schema：

```json
"Chart": {
  "type": "object",
  "description": "An interactive chart that uses a hierarchical list of objects for its data.",
  "properties": {
    "type": {
      "type": "string",
      "description": "The type of chart to render.",
      "enum": [
        "doughnut",
        "pie"
      ]
    },
    "title": {
      "type": "object",
      "description": "The title of the chart. Can be a literal string or a data model path.",
      "properties": {
        "literalString": {
          "type": "string"
        },
        "path": {
          "type": "string"
        }
      }
    },
    "chartData": {
      "type": "object",
      "description": "The data for the chart, provided as a list of items. Can be a literal array or a data model path.",
      "properties": {
        "literalArray": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "label": {
                "type": "string"
              },
              "value": {
                "type": "number"
              },
              "drillDown": {
                "type": "array",
                "description": "An optional list of items for the next level of data.",
                "items": {
                  "type": "object",
                  "properties": {
                    "label": {
                      "type": "string"
                    },
                    "value": {
                      "type": "number"
                    }
                  },
                  "required": [
                    "label",
                    "value"
                  ]
                }
              }
            },
            "required": [
              "label",
              "value"
            ]
          }
        },
        "path": {
          "type": "string"
        }
      }
    }
  },
  "required": [
    "type",
    "chartData"
  ]
}
```

---

## 2. 实现组件（客户端）

使用客户端框架实现组件。对于 Angular，组件应继承 `@a2ui/angular` 提供的 `DynamicComponent`。

在 [`orchestrator`](../../../samples/community/client/angular/projects/orchestrator/README.md) 示例中，`Chart` 组件定义在 [`chart.ts`](../../../samples/community/client/angular/projects/orchestrator/src/a2ui-catalog/chart.ts)。

{% raw %}

```typescript
import {DynamicComponent} from '@a2ui/angular';
import * as Primitives from '@a2ui/web_core/types/primitives';
import * as Types from '@a2ui/web_core/types/types';
import {Component, computed, input, Signal, signal} from '@angular/core';

@Component({
  selector: 'a2ui-chart',
  template: `
    <div>
      <h2>{{ resolvedTitle() }}</h2>
      <canvas baseChart [data]="currentData()" [type]="chartType()"></canvas>
    </div>
  `,
})
export class Chart extends DynamicComponent<Types.CustomNode> {
  readonly type = input.required<string>();
  protected readonly chartType = computed(() => this.type() as ChartType);

  readonly title = input<Primitives.StringValue | null>();
  protected readonly resolvedTitle = computed(() => super.resolvePrimitive(this.title() ?? null));

  readonly chartData = input.required<Primitives.StringValue | null>();
  // ... data resolution logic using super.resolvePrimitive for data paths
}
```

{% endraw %}

实现组件时请注意这些要点：

- **继承 `DynamicComponent`**：这样可以访问 `resolvePrimitive` 来解析数据绑定。
- **使用 Angular Inputs**：将 schema 中的属性映射到 Angular input。

---

## 3. 注册到 Renderer（客户端）

组件实现完成后，将其注册到客户端 catalog。这个步骤会把组件名（agent 使用的名称）映射到实现类。

在 [`orchestrator`](../../../samples/community/client/angular/projects/orchestrator/README.md) 示例中，这在 [`catalog.ts`](../../../samples/community/client/angular/projects/orchestrator/src/a2ui-catalog/catalog.ts) 中完成。

```typescript
import {Catalog, DEFAULT_CATALOG} from '@a2ui/angular';
import {inputBinding} from '@angular/core';

export const RIZZ_CHARTS_CATALOG = {
  ...DEFAULT_CATALOG,
  Chart: {
    type: () => import('./chart').then(r => r.Chart),
    bindings: ({properties}) => [
      inputBinding('type', () => ('type' in properties && properties['type']) || undefined),
      inputBinding('title', () => ('title' in properties && properties['title']) || undefined),
      inputBinding(
        'chartData',
        () => ('chartData' in properties && properties['chartData']) || undefined,
      ),
    ],
  },
} as Catalog;
```

注册要点：

- **懒加载**：使用 `import()` 懒加载组件代码。
- **Input 绑定**：使用 `inputBinding` 将 schema 属性映射到 Angular input。

---

## 4. 从 Agent 调用

要使用自定义组件，需要用 A2UI SDK 中理解你的 catalog 的工具初始化 agent。SDK 会负责解析 catalog 并向模型提供示例。

整体流程如下：

### 4.1 会话准备（Executor）

执行层（例如 `RizzchartsAgentExecutor`）会拦截传入消息，检测 A2UI 是否启用以及客户端支持哪些 catalog。它会解析 catalog 并保存到 session state。

```python
# In agent_executor.py

use_ui = try_activate_a2ui_extension(context)
if use_ui:
    # Resolve catalog based on client capabilities
    a2ui_catalog = self.schema_manager.get_selected_catalog(
        client_ui_capabilities=capabilities
    )
    examples = self.schema_manager.load_examples(a2ui_catalog, validate=True)

    # Save to session (Event contains state_delta)
    await runner.session_service.append_event(
        session,
        Event(
            actions=EventActions(
                state_delta={
                    _A2UI_ENABLED_KEY: True,
                    _A2UI_CATALOG_KEY: a2ui_catalog,
                    _A2UI_EXAMPLES_KEY: examples,
                }
            ),
        ),
    )
```

### 4.2 Agent 工具设置

Agent 使用 [SendA2uiToClientToolset](../../../agent_sdks/python/a2ui_agent/src/a2ui/adk/send_a2ui_to_client_toolset.py) 获得一个可用于向客户端发送 A2UI 的工具。

```python
from a2ui.adk.send_a2ui_to_client_toolset import SendA2uiToClientToolset

a2ui_catalog = self.schema_manager.get_selected_catalog(
    client_ui_capabilities=capabilities
)
agent.tools = [
    SendA2uiToClientToolset(
        a2ui_catalog=a2ui_catalog,
        a2ui_enabled=True,
    )
]
```

### 4.3 工具执行

LLM 对 [SendA2uiToClientToolset](../../../agent_sdks/python/a2ui_agent/src/a2ui/adk/send_a2ui_to_client_toolset.py) 中工具的调用，会在 A2A Agent Executor 中通过 [A2uiEventConverter](../../../agent_sdks/python/a2ui_agent/src/a2ui/adk/a2a/event_converter.py) 被拦截。这会自动把工具调用转换为带有 A2UI payload 的 A2A DataPart。

```python
from a2ui.adk.a2a.event_converter import (
    A2uiEventConverter,
)

config = A2aAgentExecutorConfig(event_converter=A2uiEventConverter())
```
