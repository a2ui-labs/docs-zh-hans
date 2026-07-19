# 通过 Model Context Protocol（MCP）使用 A2UI

本指南将展示如何借助 Tools 和 Embedded Resources，从 **MCP server** 提供**丰富、可交互的 A2UI 界面**。读完本指南后，你将拥有一个可正常工作的 MCP server，它能够向任何兼容 MCP 的客户端返回 A2UI 组件。

<video width="100%" height="auto" controls playsinline style="display: block; aspect-ratio: 16/9; object-fit: cover; border-radius: 8px; margin-bottom: 24px;">
  <source src="https://raw.githubusercontent.com/a2ui-project/a2ui/main/docs/public/assets/guides-a2ui-over-mcp-tour.mp4" type="video/mp4">
  你的浏览器不支持 video 标签。
</video>

## 前置条件

在开始之前，请确保已安装以下内容：

- **Python**（3.10 或更高版本）。
- **[uv](https://docs.astral.sh/uv/)**，用于快速的 Python 包管理。
- **Node.js**（18 或更高版本），用于运行 MCP Inspector。

## 快速开始：运行示例

在深入了解协议细节之前，先跑通一个可用的示例。A2UI 仓库中包含一个开箱即用的 MCP 配方（recipe）演示。

```bash
# 克隆仓库（如果还没有克隆）
git clone https://github.com/a2ui-project/a2ui.git
cd a2ui/samples/mcp/a2ui-over-mcp-recipe

# 启动 MCP server（SSE transport，端口 8000）
uv run .
```

### 选项 A：通过 MCP Inspector 进行交互

在另一个终端中，启动 [MCP Inspector](https://github.com/modelcontextprotocol/inspector) 来与服务器交互：

```bash
npx @modelcontextprotocol/inspector
```

在 Inspector 中：

1. 将 **Transport Type** 设置为 `SSE`
2. 连接到 `http://localhost:8000/sse`
3. 点击 **List Resources** → 你会看到名为 "Recipe Form" 的 resource。
4. 读取 `a2ui://recipe-form` resource → 该 resource 的内容就是渲染出这个简单表单的 A2UI JSON。
5. 点击 **List Tools** → 你会看到 `get_recipe_a2ui`
6. 运行该 tool → 响应中包含渲染出配方卡片的 A2UI JSON

> NOTE: 说明
>
> 该示例通过本地路径引用 A2UI Agent SDK。在你自己的项目中，请从 PyPI 安装：
>
> ```bash
> pip install a2ui-agent-sdk
> ```

### 选项 B：运行 Recipe 客户端 Web 应用

如果想获得能直观展示 A2UI over MCP 效果的完整渲染交互体验，可以运行内置的 Web 应用：

> [!NOTE]
> **包管理器的使用：** 在 A2UI 仓库内运行内置示例应用需要使用 Yarn（`yarn install` / `yarn dev`），这是由 Corepack workspaces 配置决定的。如果是你自己日常使用或本仓库之外的独立项目，可以使用你偏好的任意包管理器（例如 npm、pnpm）。

1. 在新的终端窗口中，进入客户端目录：
    ```bash
    cd client
    ```
2. 安装 Node.js 依赖：
    ```bash
    yarn install
    ```
3. 启动 Vite 开发服务器：
    ```bash
    yarn dev
    ```
4. 在浏览器中打开终端里显示的 URL（通常是 `http://localhost:5173`）。

你会看到一个精美、具有响应式布局的双栏界面：左侧栏从 MCP Resource（`a2ui://recipe-form`）渲染出选择表单。选择选项并点击 **"Get Recipe"** 会执行 MCP Tool（`get_recipe_a2ui`），并在右侧栏动态渲染出返回的定制化 A2UI 配方卡片。

![Dynamic Recipe Studio demo showing selection form on the left and dynamic recipe card generation on the right](../assets/recipe_sample.gif)

完整示例请参见 [`samples/community/mcp/`](https://github.com/a2ui-project/a2ui/tree/main/samples/community/mcp)。

## 工作原理

MCP server 向客户端投递 A2UI 内容主要有以下两种方式：

1. **通过读取 Resource（`resources/read`）**：客户端直接读取一个 MCP resource（例如 `a2ui://recipe-form`），服务器直接返回 A2UI JSON 负载。
2. **通过调用 Tool（`tools/call`）**：客户端调用一个 MCP tool（例如 `get_recipe_a2ui`），服务器将 A2UI JSON 负载包装为 tool 响应内的一个 **Embedded Resource** 返回。

无论采用哪种方式，客户端都会检测 `application/a2ui+json` MIME 类型，并将该负载路由给 A2UI renderer。

> [!IMPORTANT]
> **MIME 类型的一致性**
> 无论通过哪种投递渠道（无论是作为 Resource 直接获取，还是包装在 Tool 的 `CallToolResult` 中返回），A2UI JSON 负载始终以 `application/a2ui+json` MIME 类型标识。在 Tool 的响应中，负载必须被包装在携带该 MIME 类型的 `EmbeddedResource` 内。这种统一的标识方式，使客户端中间件能够无缝地拦截静态 resource 和动态 tool 响应，并将两者都路由给 A2UI。

### 1. 基于 Resource 的交付流程（`resources/read`）

```
Client → resources/read → MCP Server
                             ↓
                 Retrieve A2UI JSON
                             ↓
Client ← ResourceContents ← MCP Server
          (application/a2ui+json)
   ↓
A2UI Renderer displays UI
```

### 2. 基于 Tool 的交付流程（`tools/call`）

```
Client → tools/call → MCP Server
                         ↓
              Generate A2UI JSON
                         ↓
         Wrap as EmbeddedResource
              (application/a2ui+json)
                         ↓
Client ← CallToolResult ← MCP Server
   ↓
A2UI Renderer displays UI
```

## Resources 与 Tools：职责分工

在设计基于 MCP 的 A2UI 集成时，应根据 UI 负载是静态还是动态，在 **Resources** 与 **Tools** 之间做出选择。

### 1. 通过 MCP Resources 提供静态 UI（`resources/read`）

对于不依赖用户提示输入或对话历史的简单静态用户界面，应直接以 MCP Resource 的形式提供 A2UI。

- **概念**：客户端使用标准的 resource URI（例如 `a2ui://recipe-form`）读取一个预先定义好的 A2UI resource。
- **使用场景**：非常适合静态配置表单、选择界面、设置仪表盘或固定布局。
- **优势**：实现极其简单、开销低，并且不需要 LLM/智能体 额外发起一次 tool 调用来获取结构。

**Python 服务端示例：**

```python
@app.list_resources()
async def list_resources() -> list[types.Resource]:
    return [
        types.Resource(
            uri="a2ui://recipe-form",
            name="Recipe Form",
            mimeType="application/a2ui+json",
            description="Static form allowing users to pick options.",
        )
    ]

@app.read_resource()
async def read_resource(uri: str) -> list[ReadResourceContents]:
    if uri == "a2ui://recipe-form":
        return [
            ReadResourceContents(
                content=json.dumps(recipe_form_json),
                mime_type="application/a2ui+json",
            )
        ]
    raise ValueError(f"Unknown resource: {uri}")
```

### 2. 通过 MCP Tools 提供动态 UI（`tools/call`）

对于需要根据对话上下文、用户参数或实时数据动态生成的用户界面，应将 A2UI 放在 MCP Tool 的响应中提供。

- **概念**：客户端/智能体 携带特定参数（例如所选食材、偏好设置）调用一个 tool，服务器返回一个定制化的 A2UI JSON，并将其包装在 `CallToolResult` 中的 `EmbeddedResource` 内。
- **使用场景**：非常适合依赖实时数据库查询、此前的输入、交互式分步向导状态，或个性化推荐（例如定制化的配方卡片）的内容。
- **优势**：最大化灵活性和上下文感知能力，并支持高度动态的流程。
- **最佳实践（回退文本）**：始终在 `CallToolResult` 中的 `EmbeddedResource` 旁附带一个 `TextContent`。不支持 A2UI 的客户端会回退显示这段文本给用户。

**Python 服务端示例：**

```python
@app.call_tool()
async def handle_call_tool(name: str, arguments: dict[str, Any]) -> types.CallToolResult:
    if name == "get_recipe_a2ui":
        # 从客户端参数中解析动态选择
        style = arguments.get("cookingStyle", "Baked")
        protein = arguments.get("protein", "Salmon")

        # 获取定制化的配方数据库条目
        recipe_data = RECIPES.get((style, protein))

        # 动态定制基础 A2UI schema
        custom_recipe_json = copy.deepcopy(recipe_a2ui_json)
        custom_recipe_json[1]["updateComponents"]["components"][0]["text"] = recipe_data["title"]

        # 将定制化的配方卡片作为 EmbeddedResource 返回
        return types.CallToolResult(content=[
            types.EmbeddedResource(
                type="resource",
                resource=types.TextResourceContents(
                    uri="a2ui://recipe-card",
                    mimeType="application/a2ui+json",
                    text=json.dumps(custom_recipe_json),
                )
            )
        ])
```

## 目录协商

在服务器能够向客户端发送 A2UI 之前，双方必须先确定可用的 catalog。根据具体架构的不同，这可以通过以下两种方式之一完成。

### 选项 A：在 MCP 初始化时完成（推荐）

由于 MCP 是有状态会话协议，最有效的方式是在建立连接时一次性声明能力。客户端会在 `capabilities` 下声明自己的 A2UI 支持：

```json
{
  "jsonrpc": "2.0",
  "method": "initialize",
  "id": "init-123",
  "params": {
    "protocolVersion": "2025-11-25",
    "clientInfo": {
      "name": "a2ui-enabled-client",
      "version": "1.0.0"
    },
    "capabilities": {
      "a2ui": {
        "clientCapabilities": {
          "v0.9": {
            "supportedCatalogIds": [
              "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json"
            ]
          }
        }
      }
    }
  }
}
```

服务器会在整个会话期间保存这份状态。

### 选项 B：基于每条消息的元数据（适用于无状态服务器）

如果你的服务器必须保持无状态，客户端可以在每次 tool 调用的 `_meta` 字段中传递 A2UI 能力：

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-123",
  "params": {
    "name": "generate_report",
    "arguments": {"date": "2026-03-01"},
    "_meta": {
      "a2ui": {
        "clientCapabilities": {
          "v0.9": {
            "supportedCatalogIds": [
              "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json"
            ],
            "inlineCatalogs": []
          }
        }
      }
    }
  }
}
```

## 处理用户动作

像 `Button` 这样的交互式组件可以触发 Action，并将其作为 MCP tool 调用发送回服务器。

### 1. 定义一个带有 Action 的 Button

在你的 A2UI JSON 中，为组件添加一个 `action`：

```json
{
  "id": "confirm-button",
  "component": {
    "Button": {
      "child": "confirm-button-text",
      "action": {
        "event": {
          "name": "confirm_booking",
          "context": {
            "start": "/dates/start",
            "end": "/dates/end"
          }
        }
      }
    }
  }
}
```

### 2. 客户端将 Action 作为 Tool 调用发送

当用户点击按钮时，客户端会依据 surface 状态解析数据绑定（例如 `/dates/start`），并发送一个 tool 调用：

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-456",
  "params": {
    "name": "a2ui_action",
    "arguments": {
      "name": "confirm_booking",
      "context": {
        "start": "2026-03-20",
        "end": "2026-03-25"
      }
    }
  }
}
```

### 3. 在服务器端处理 Action

```python
@self.tool()
async def a2ui_action(name: str, context: dict) -> types.CallToolResult:
    """处理 A2UI 用户 Action。"""
    if name == "confirm_booking":
        # 处理预订，然后返回确认 UI
        return types.CallToolResult(content=[
            types.TextContent(
                type="text",
                text=f"Booking confirmed: {context['start']} to {context['end']}"
            )
        ])
    raise ValueError(f"Unknown action: {name}")
```

## 错误处理

客户端可以通过 tool 调用，将 A2UI 渲染错误报告回服务器：

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-789",
  "params": {
    "name": "a2ui_error",
    "arguments": {
      "code": "INVALID_JSON",
      "message": "Failed to parse A2UI payload.",
      "surfaceId": "default"
    }
  }
}
```

在服务器端处理：

```python
@self.tool()
async def a2ui_error(code: str, message: str, surfaceId: str = "") -> types.CallToolResult:
    """处理 A2UI 客户端错误。"""
    # 记录错误、重试，或发送回退 UI
    return types.CallToolResult(content=[
        types.TextContent(
            type="text",
            text=f"Acknowledged error {code}: {message}"
        )
    ])
```

## 口头化与可见性控制

使用 MCP **Resource Annotations**，可以控制 LLM 在后续对话轮次中是否能够"读取" A2UI 负载：

```python
a2ui_resource = types.EmbeddedResource(
    type="resource",
    resource=types.TextResourceContents(
        uri="a2ui://training-plan-page",
        mimeType="application/a2ui+json",
        text=json.dumps(a2ui_payload)
    ),
    # 向用户展示 UI，同时对 LLM 隐藏原始 JSON
    annotations=types.Annotations(audience=["user"])
)
```

| 受众             | 行为                                     |
| ---------------- | ---------------------------------------- |
| _(空)_           | 对用户和 LLM 均可见                       |
| `["user"]`       | 渲染给用户；对 LLM 上下文隐藏             |
| `["assistant"]`  | 可供 LLM 用于后续推理；不会被渲染         |

## 使用 A2UI Agent SDK

在生产环境中，**A2UI Agent SDK** 会为你处理 schema 管理、校验，以及提示词生成：

```bash
pip install a2ui-agent-sdk
```

```python
from a2ui.strategies.schema import A2uiSchemaManager
from a2ui.basic_catalog.provider import BasicCatalog

# 使用 basic catalog 初始化 schema manager
schema_manager = A2uiSchemaManager(
    catalogs=[BasicCatalog.get_config()],
)

# 在发送前校验 A2UI 输出
selected_catalog = schema_manager.get_selected_catalog()
selected_catalog.validator.validate(a2ui_payload)
```

关于 schema 管理、动态 catalog 和流式传输的更多细节，请参阅完整的 [代理开发指南](agent-development.md)。

## 下一步

- [A2UI 规范](../specification/v0.9-a2ui.md) — 完整协议参考
- [组件画廊](../reference/components.md) — 浏览可用组件
- [在 A2UI Surface 中集成 MCP Apps](mcp-apps-in-a2ui.md) — 在 A2UI 中嵌入基于 HTML 的 MCP apps
- [客户端设置指南](client-setup.md) — 构建一个能够显示 A2UI 的 renderer
