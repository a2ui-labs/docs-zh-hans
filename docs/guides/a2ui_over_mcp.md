# 通过 Model Context Protocol（MCP）使用 A2UI

本指南说明如何使用 **A2UI** 的声明式语法，基于 **Model Context Protocol（MCP）**，借助 Tools 和 Resources 构建丰富、可交互的界面。

示例请参见 [MCP Samples](../../samples/agent/mcp)。

## 目录协商

在服务器向客户端发送 A2UI 之前，双方必须先确认彼此都支持该协议，并确定可用的 catalog。根据系统架构的不同，这种能力协商可以通过两种方式完成：在初始连接握手时完成，或者在每条消息级别完成。

### 选项 A：在 MCP 初始化期间进行目录握手

由于 MCP 是有状态会话协议，最有效的方式是在建立连接时一次性声明能力。客户端会在标准 `initialize` 请求的 `capabilities` 对象中声明自己的 A2UI 支持（通常放在实验性或自定义键下）。服务器会在整个会话期间保存这份状态。

初始化请求示例：

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
          "v0.10": {
            "supportedCatalogIds": [
              "https://a2ui.org/specification/v0_10/basic_catalog.json"
            ]
          }
        }
      }
    }
  }
}
```

### 选项 B：在每条 MCP 消息上进行目录握手（适用于无状态服务器）

如果你的架构要求 MCP Server 完全保持无状态，那么客户端可以在每一次 tool 调用请求的 `_meta` 字段中传递自己的 A2UI 版本和 catalog 支持信息。服务器会实时读取这些元数据，以决定响应 UI 应该使用哪个 catalog。

调用请求元数据示例：

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-123",
  "params": {
    "name": "generate_report",
    "arguments": { "date": "2026-03-01" },
    "_meta": {
      "a2ui": {
        "clientCapabilities": {
          "v0.10": {
            "supportedCatalogIds": [
              "https://a2ui.org/specification/v0_10/basic_catalog.json"
            ],
            "inlineCatalogs": []
          }
        }
      }
    }
  }
}
```

## 将 A2UI 内容作为嵌入式资源返回

Embedded Resources 允许某个 Tool 直接返回与该次响应绑定的 UI 布局，而不需要服务端额外存储或跟踪。

- **URI**：必须使用 `a2ui://` 前缀，并带有描述性的名称标识符（例如 `a2ui://training-plan-page`）。
- **MIME Type**：必须使用 `application/json+a2ui`。这样 MCP 客户端才会把负载交给 A2UI 渲染器，而不是把原始 JSON 直接展示给用户。

#### Python 实现示例

```python
import mcp.types as types

@self.tool()
def get_hello_world_ui():
    a2ui_payload = [
        {
            "version": "v0.10",
            "createSurface": {
                "surfaceId": "default",
                "catalogId": "https://a2ui.org/specification/v0_10/basic_catalog.json"
            }
        },
        {
            "version": "v0.10",
            "updateComponents": {
                "surfaceId": "default",
                "components": [
                    {
                        "id": "root",
                        "component": "Text",
                        "text": "Hello World!"
                    }
                ]
            }
        }
    ]

    # 将 A2UI 封装为 Embedded Resource
    a2ui_resource = types.EmbeddedResource(
        type="resource",
        resource=types.TextResourceContents(
            uri="a2ui://training-plan-page",
            mimeType="application/json+a2ui",
            text=json.dumps(a2ui_payload),
        )
    )

    text_content = types.TextContent(
        type="text",
        text="Here is your generated training plan summary..."
    )

    return types.CallToolResult(content=[text_content, a2ui_resource])
```

## 处理用户动作

交互式组件（例如 `Button`）可以把 `actions` 回传给服务器。

#### 1. 带有动作的 A2UI JSON

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

#### 2. A2UI 动作 MCP 负载

当按钮被点击时，客户端会把任何绝对或相对路径模型（例如 `/dates/start` 或 `/dates/end`）相对于 surface 的绑定状态进行解析，并将其转换成 MCP tool 调用参数。

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-456",
  "params": {
    "name": "action",
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

#### 3. 动作处理器 MCP Server Tool

MCP server 接收到 tool 调用后，会执行对应的处理逻辑。

```python
@self.tool()
async def action(action_payload: Dict[str, Any]) -> Dict[str, Any]:
    if action_payload["name"] == "confirm_booking":
        return {"response": f"Booking confirmed for {action_payload['context']['start']} to {action_payload['context']['end']}."}
    raise ValueError(f"Unknown action: {action_payload['name']}")
```

## 错误处理

和处理用户交互类似，MCP server 也可以接收来自客户端的错误。

#### 1. A2UI 错误 MCP 负载

当客户端在处理 A2UI 负载时遇到错误，可以向服务器发送 error MCP 负载。

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-789",
  "params": {
    "name": "error",
    "arguments": {
      "code": "INVALID_JSON",
      "message": "Failed to parse A2UI payload.",
      "surfaceId": "default",
    }
  }
}
```

#### 2. 错误处理器 MCP Server Tool

MCP server 接收到 tool 调用后，会执行对应的处理逻辑。

```python
@self.tool()
async def error(error_payload: Dict[str, Any]) -> Dict[str, Any]:
    return {"response": f"Received A2UI error: {error_payload['error']}."}
```

## 口头化与可见性控制

你可以使用 MCP **Resource Annotations** 控制后续 assistant 回合是否能够“读取”或解释后端负载。

```python
a2ui_resource = types.EmbeddedResource(
    type="resource",
    resource=types.TextResourceContents(
        uri="a2ui://training-plan-page",
        mimeType="application/json+a2ui",
        text=json.dumps(a2ui_payload)
    ),
    # 对 LLM 隐藏原始 JSON，但仍向用户展示 UI
    annotations=types.Annotations(audience=["user"])
)
```

- **空受众**：元素对用户和 LLM 模型都可见。
- **受众 `user`**：在视图界面上渲染该项所必需。
- **受众 `assistant`**：允许在连续回合中触发内容口头化并作为提示输入。禁用 assistant 会限制代理的上下文解析，但仍保留离散且安全的数据暴露。
