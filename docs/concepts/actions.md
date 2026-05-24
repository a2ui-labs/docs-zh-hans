# 处理用户 Actions

本指南说明 A2UI 如何处理用户交互。组件使用 `action` 属性来触发本地 **Functions**（在 renderer 上执行）或 **Events**（派发给 agent）。此外，**Data Model Synchronization** 可确保 agent 始终能够访问完整 UI 状态，从而支持语音命令等顺畅的多模态交互。这个设计在保持安全、受限环境的同时，也能实现高响应性的界面。

## Action 架构

Action 允许 UI 组件触发 [`common_types.json`](../../specification/v0_9/json/common_types.json#L271-L313) 中 [`Action`](../../specification/v0_9/json/common_types.json#L271-L313) schema 定义的行为。Action 可以触发：

1.  **Events**：派发给 Agent 处理（在 Agent 上执行，例如点击“Submit”）。
2.  **Functions**：使用 [`FunctionCall`](../../specification/v0_9/json/common_types.json#L200-L242) 完全在 renderer 上执行（在 Renderer 上执行，例如打开 URL）。

### 1. Functions（本地）

Function 会在 renderer 上立即执行行为，不需要网络往返。Agent 不会收到本地 function call 的通知。它们使用 `functionCall` 关键字。

```json
{
  "id": "help-btn",
  "component": "Button",
  "child": "help-text",
  "action": {
    "functionCall": {
      "call": "openUrl",
      "args": {"url": "https://a2ui.org/help"}
    }
  }
}
```

Function 的常见用途包括：

- **导航**：打开 URL 或切换标签页。
- **校验**：提交前检查输入（见下方 Checks）。

### 2. Events（Agent）

Event 会把数据发送给 agent 处理。它们使用 `event` 关键字。

`Button` 这样的组件会暴露 `action` 属性。下面是一个 Event 的连接方式：

```json
{
  "id": "submit-btn",
  "component": "Button",
  "child": "btn-text",
  "action": {
    "event": {
      "name": "submit_reservation",
      "context": {
        "time": {"path": "/reservationTime"},
        "size": {"path": "/partySize"}
      }
    }
  }
}
```

- **`name`**：供 agent 分支处理的稳定标识符。
- **`context`**：键值对映射。值可以是字面量，也可以用 `path` 从 data model 当前状态中取值。

NOTE: **Context 与 Data Model**：Data Model 表示某个 surface 的完整状态树，而 action 中的 `context` 实际上是从该状态中手动挑选出来的 **“视图”** 或子集。这能为特定 event 精确提供所需值，让 Agent 不必遍历可能很大且复杂的 data model。

### Basic Catalog Function Validation（Checks）

Basic catalog 定义了一组可在 renderer 上执行的有限 checks。交互组件可以定义 `checks` 列表（使用 `common_types.json` 中的 [`Checkable`](../../specification/v0_9/json/common_types.json#L258-L270) schema）。对于 `Button`，如果任一 check 失败，按钮会在 renderer 上 **自动禁用**。

- **UX 重点**：Validation check 用来管理 **UI State（用户体验）**，在无效交互发生前阻止它们。它们不能替代 **Data Integrity** check；后者仍必须在 agent 上执行。

这样 UI 就能在用户尝试提交之前强制满足要求，例如字段不能为空。

```json
{
  "id": "submit-button",
  "component": "Button",
  "child": "submit-text",
  "checks": [
    {
      "condition": {
        "call": "required",
        "args": {"value": {"path": "/partySize"}}
      },
      "message": "Party size is required"
    }
  ],
  "action": {"event": {"name": "submit_booking"}}
}
```

## 本地状态更新与 “Write” 契约

在 Event 被派发之前，renderer 就已经在本地管理 UI 状态。A2UI 为所有输入组件（如 `TextField`、`CheckBox` 或 `Slider`）定义了 **Read/Write Contract**。

1.  **Read（Model → View）**：组件渲染时，从 Data Model 中绑定的 `path` 拉取值。
2.  **Write（View → Model）**：用户一发生交互（例如输入字符或点击复选框），renderer 就会 **立即** 将新值写入本地 Data Model。

这意味着本地 model **始终** 是 UI 当前状态的事实来源。这种 “View-to-Model” 同步完全在 renderer 上发生。Data model 只会在 event 发生时（例如点击 Button）发送给 agent。

IMPORTANT: **同步更新**：本地 model 更新是 **同步** 的。这保证在任何 Event 解析其 `context` path 或封装 `DataModelSync` payload 前，Data Model 都已经完全更新。输入和点击之间不存在竞态条件；“Write” 总是先提交。

这种本地优先方式带来显著的 **性能收益**。因为同步即时且本地完成，开发者不需要在用户输入 `TextField` 时实现网络防抖，也不用担心延迟抖动。网络会完全避免 “UI noise”（如逐个按键），直到用户准备好派发正式 Event。

### 表单提交模式

这种分离支持稳健的表单提交模式：

- **Binding**：`TextField` 绑定到 `/reservationTime`。
- **Interaction**：用户输入 “7:00 PM”。本地 model 中的 `/reservationTime` 会立即更新。
- **Submission**：用户点击 “Book” 按钮。按钮的 Event 会从本地 model 解析 `path: "/reservationTime"`，并把当前值发送给 agent。

## 用户交互流程

当用户与组件交互时（例如点击按钮）：

1.  **Resolve**：renderer 根据本地 **Data Model** 解析 `context` 中的所有 `path` 引用。
2.  **Construct**：renderer 构建符合 [`client_to_server.json`](../../specification/v0_9/json/client_to_server.json) 的 `action` payload。
3.  **Dispatch**：payload 通过所选传输层发送（例如 A2A、WebSockets）。

### 示例：Action Payload（v0.9）

如果用户点击上面的按钮，而 data model 包含 `{"reservationTime": "7:00 PM", "partySize": 4}`，renderer 会用 `action` key 发送消息：

```json
{
  "version": "v0.9",
  "action": {
    "name": "submit_reservation",
    "surfaceId": "booking-surface",
    "sourceComponentId": "submit-btn",
    "timestamp": "2026-02-25T10:40:00Z",
    "context": {
      "time": "7:00 PM",
      "size": 4
    }
  }
}
```

IMPORTANT: **版本说明（v0.8 vs v0.9）**：在 v0.8 中，顶层 payload key 是 `userAction`（例如 `{"userAction": {...}}`）。v0.9 转为上面更简洁的 `action` key。标准协议 parser 会期待与 payload 声明版本对应的 key。

## Agent 处理

Agent（或 Orchestrator）收到该 event 并采取行动。在 agentic 系统中，agent 通常会把 event 转换为给 LLM 的隐藏用户查询。

**Agent 处理示例（Python）：**

```python
if action_name == "submit_reservation":
    time = context.get("time")
    size = context.get("size")
    # Feed this to the LLM
    query = f"User submitted a reservation for {size} people at {time}."
    response = await llm.generate(query)
```

## Renderer 到 Agent 的错误报告

除了用户触发的 Event 之外，renderer 还可以使用 [`client_to_server.json`](../../specification/v0_9/json/client_to_server.json) 中定义的 `error` payload，将系统级错误报告给 agent。

### 校验失败

如果 agent 发送的 A2UI JSON 违反 catalog schema 或协议规则，renderer 会发送 `VALIDATION_FAILED` 错误。这是 agentic 系统中的关键反馈回路：

```json
{
  "version": "v0.9",
  "error": {
    "code": "VALIDATION_FAILED",
    "surfaceId": "booking-surface",
    "path": "/components/0/children",
    "message": "Expected array of strings, got null."
  }
}
```

Agent 可以捕获该错误、道歉（或在内部自我修正），然后重新发送修正后的 UI。

## Data Model Sync（v0.9）

A2UI v0.9 引入了强大的“无状态”同步功能。它允许 renderer 自动把某个 surface 的 **完整 data model** 包含到它发送给 agent 的每条消息的 metadata 中。

### 启用同步

同步由 agent 在 surface 初始化期间请求。通过在 `createSurface` 消息中设置 `sendDataModel: true`，agent 会指示 renderer 启动同步循环。

```json
{
  "version": "v0.9",
  "createSurface": {
    "surfaceId": "booking-surface",
    "catalogId": "https://a2ui.org/catalogs/v1/basic.json",
    "sendDataModel": true
  }
}
```

### 线上传输中的同步

启用同步后，renderer 不会把 data model 作为单独消息发送。相反，它会把 data model 作为 **metadata** 附加到外发的传输 envelope（例如 A2A message）上。

在 A2A（Agent-to-Agent）绑定中，data model 会放在 envelope `metadata` 字段内的 `a2uiClientDataModel` 对象中。

**带同步的 A2A Envelope 示例：**

```json
{
  "parts": [{"text": "Submit the reservation"}],
  "metadata": {
    "a2uiClientDataModel": {
      "version": "v0.9",
      "surfaces": {
        "booking-surface": {
          "reservationTime": "7:00 PM",
          "partySize": 4,
          "notes": "Window seat preferred"
        }
      }
    }
  }
}
```

### 为什么使用 Data Model Sync？

- **连接更简单**：无需手动把每个输入字段映射到按钮的 `context` 属性。Agent 可以直接检查 metadata，看到所有字段的当前状态。
- **无状态 Agent**：Agent 不需要为每个用户 session 维护本地状态；它在每次交互中都会收到完整的当前上下文。
- **语音快捷方式**：允许用户通过语音或文本触发 Event（例如“好的，提交”），即使没有点击特定按钮也可以。由于 agent 会随文本消息一起收到更新后的 data model，因此可以立即处理请求。

## Renderer Metadata 与 Capabilities

Agent 在安全发送 UI 之前，renderer 必须声明它支持哪些 component catalog。这通过 `a2uiClientCapabilities` 对象处理。

### 声明能力

Renderer 会在发给 agent 的消息 **metadata** 中包含 `a2uiClientCapabilities` 对象（例如 A2A envelope 的 `metadata` 字段）。

```json
{
  "v0.9": {
    "supportedCatalogIds": [
      "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json",
      "https://my-company.com/catalogs/v1/custom.json"
    ],
    "inlineCatalogs": []
  }
}
```

- **`supportedCatalogIds`**：renderer 可以渲染的 catalog URI 数组。
- **`inlineCatalogs`**：（可选）用于开发或专门环境，允许内联发送完整 catalog schema。

没有这个握手，agent 无法确定 renderer 是否能处理要发送的特定组件。

## 传输与编码

A2UI 与具体传输层无关，但最常通过 **A2A（Agent-to-Agent）** 或 WebSockets 使用。理解 payload 如何被包裹对实现至关重要。

### A2A 编码

在标准 A2A 绑定中，A2UI 消息编码为 A2A **DataPart**。为了将其识别为 A2UI payload，part 必须用特定 metadata 包裹：

- **mimeType**：`application/json+a2ui`

`DataPart` 的 `data` 字段包含 A2UI 消息 **列表**。这允许在单个网络包中发送多个更新（例如 `createSurface` 后接 `updateComponents`）。

NOTE: **A2A 版本说明**：`data` 字段中使用 **列表** 是在 **A2A v1.0** 中引入的。更早版本的 A2A 协议期望 `data` 字段包含单个 JSON object。

```json
{
  "kind": "data",
  "metadata": {
    "mimeType": "application/json+a2ui"
  },
  "data": [
    {
      "version": "v0.9",
      "action": { ... }
    }
  ]
}
```

## 安全注意事项

A2UI 把安全、沙箱化通信作为核心原则。由于协议依赖通过网络传递用户状态和交互触发，它会对数据可见性和执行施加严格边界。

### 沙箱化执行

A2UI 的一个核心卖点是通过限制获得安全性。通过禁止 agent 执行任意代码（例如注入原始 JavaScript），A2UI 确保 agent 只能触发预注册行为。`functionCall` 机制提供了一种安全、沙箱化的方式，让 agent 与 renderer 环境交互，而不会让用户暴露于恶意脚本。

### Data Model 隔离与 Orchestrator 路由

启用 `sendDataModel: true` 后，renderer 会在外发消息中包含 surface 的完整 data model。开发者必须理解这些数据的可见范围：

- **点对点可见性**：只有接收传输 envelope 的后端（创建该 surface 的 Agent，或中间 Orchestrator）可以读取此 payload。
- **Orchestrator 的责任**：在多 agent 架构中，中央 Orchestrator 通常会把用户意图路由到专门 sub-agent。Orchestrator 必须强制执行 **数据隔离**。它负责解析 `a2uiClientDataModel`、识别 `surfaceId`，并确保 data model 只传给拥有该 surface 的特定 sub-agent。一个 agent surface 的数据绝不能泄漏给另一个 agent。

## 编排与路由

在多 agent 系统中，中央 **Orchestrator** 通常管理用户与多个专门 sub-agent 之间的交互。一个关键挑战是确保来自 renderer 的 `action` 消息被路由回生成该 UI surface 的特定 sub-agent。

### Surface 所有权模式

为处理这个问题，orchestrator 必须维护 `surfaceId` 到其所属 sub-agent 的映射。这通常存储在 **Session State** 中。

#### 1. 映射所有权

当 sub-agent 发出 `createSurface` 消息时，orchestrator 会拦截它并记录所有权。

```python
# Simplified Orchestrator Logic: Record Ownership
def on_surface_created(surface_id, agent_name, session):
    # Store the mapping in the orchestrator's session state
    session.state.update({f"owner_of_{surface_id}": agent_name})
```

#### 2. 路由 Event

当 renderer 把 `action` 发回 orchestrator 时，orchestrator 会查找 `surfaceId`，并把请求转交给正确的 sub-agent。

```python
# Simplified Orchestrator Logic: Route Event
async def handle_incoming_action(payload, session):
    action = payload.get("action")
    surface_id = action.get("surfaceId")

    # Lookup the owning agent
    target_agent = session.state.get(f"owner_of_{surface_id}")

    if target_agent:
        # Programmatically route the request to the sub-agent
        return transfer_to(target_agent)
```

该模式确保即使在复杂的多 agent 环境中，双向通信循环仍能为每个功能区域保持完整且有状态。

### 通过 Metadata Stripping 防止数据泄漏

在多 agent 环境中，`a2uiClientDataModel` 可能包含多个由不同 agent 拥有的 surface 状态。为防止敏感数据泄漏，orchestrator 必须 **strip** data model metadata，只保留被调用的特定 sub-agent 所拥有的 surface。

这最好在 outbound metadata interceptor 中实现：

```python
# Simplified Orchestrator Interceptor: Strip Data Model
async def intercept(self, request_payload, target_agent, session):
    message = request_payload["params"]["message"]
    data_model = message.get("metadata", {}).get("a2uiClientDataModel")

    if data_model:
        # Filter surfaces to only those owned by the target_agent
        filtered_surfaces = {
            surface_id: state for surface_id, state in data_model["surfaces"].items()
            if session.state.get(f"owner_of_{surface_id}") == target_agent.name
        }

        # Replace with the stripped data model
        message["metadata"]["a2uiClientDataModel"]["surfaces"] = filtered_surfaces

    return request_payload
```

通过剥离 metadata，orchestrator 可以确保 sub-agent 只收到自己有权查看的 data model 部分。

CAUTION: **安全风险：状态抓取**：如果 Orchestrator 未剥离 `a2uiClientDataModel`，恶意或遭入侵的 sub-agent 可能会“抓取”其他 active surface 的状态。例如，如果 orchestrator 泄漏了整个多 surface data model，一个天气 sub-agent 就可能读取银行 surface 的敏感数据。剥离是多 agent 系统中的强制安全要求。

---

## 综合示例

### 1. Button Submit（显式 Context）

此示例展示一个按钮如何显式收集需要发送的数据。

**组件定义：**

```json
{
  "id": "submit-button",
  "component": "Button",
  "child": "submit-text",
  "action": {
    "event": {
      "name": "submit_booking",
      "context": {
        "partySize": {"path": "/partySize"},
        "reservationTime": {"path": "/reservationTime"}
      }
    }
  }
}
```

**生成的 Action Payload：**
Agent 会收到一个 `action` 对象，其中 `partySize` 和 `reservationTime` 直接位于 `context` 字段中。

### 2. 语音提交（Data Model Sync）

在这个场景中，用户没有点击按钮，而是说“Okay, submit the form.”

**初始化：**
Agent 用 `sendDataModel: true` 创建了 surface：

```json
{
  "version": "v0.9",
  "createSurface": {
    "surfaceId": "booking-surface",
    "catalogId": "...",
    "sendDataModel": true
  }
}
```

**Renderer 传输：**
Renderer 发送一条 A2A 消息，其中包含用户文本以及 metadata 中的 data model：

```json
{
  "parts": [{"text": "Okay, submit the form"}],
  "metadata": {
    "a2uiClientDataModel": {
      "version": "v0.9",
      "surfaces": {
        "booking-surface": {
          "partySize": 4,
          "reservationTime": "7:00 PM"
        }
      }
    }
  }
}
```

**Agent 处理：**
Agent 看到用户意图（“submit”），并查看 `metadata` 找到 `partySize` 和 `reservationTime` 的当前值，从而无需进一步澄清即可完成任务。
