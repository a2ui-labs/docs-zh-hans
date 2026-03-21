# A2UI 客户端到服务端动作

A2UI 的交互依赖一个双向通信回路。智能体通过流式发送组件和数据更新来驱动 UI，而客户端则通过 **动作（Actions）** 和 **数据模型同步** 把用户意图传回智能体。

## 动作架构

动作允许 UI 组件触发行为。它们定义在 [`common_types.json`](../specification/v0_9/json/common_types.json) 的 `Action` schema 中，并有两种形式：

1.  **服务端事件**：发送给智能体处理，例如点击“提交”。
2.  **本地函数调用**：完全在客户端执行，例如打开 URL。

### 在 schema 中连接动作

像 `Button` 这样的组件会暴露 `action` 属性。下面是一个服务端事件的连接方式：

```json
{
  "id": "submit-btn",
  "component": "Button",
  "child": "btn-text",
  "action": {
    "event": {
      "name": "submit_reservation",
      "context": {
        "time": { "path": "/reservationTime" },
        "size": { "path": "/partySize" }
      }
    }
  }
}
```

- **`name`**：供智能体做分支判断的稳定标识符。
- **`context`**：键值对映射。值可以是字面量，也可以使用 `path` 从当前数据模型状态中取值。

> [!NOTE]
> **Context 与 Data Model**：数据模型表示一个 surface 的完整状态树，而动作里的 `context` 实际上更像是从这棵树里挑选出来的一个 **“视图”** 或子集。这样可以让智能体只拿到特定事件所需的值，而不必去遍历一个可能非常大且复杂的数据模型。

### 本地动作 vs 服务端事件

服务端事件是与智能体交互的主要方式，但 **本地动作** 允许客户端直接立即响应，而不需要网络往返。这对于需要快速反馈的 UI 模式非常重要。

```json
{
  "id": "help-btn",
  "component": "Button",
  "child": "help-text",
  "action": {
    "functionCall": {
      "call": "openUrl",
      "args": { "url": "https://a2ui.org/help" }
    }
  }
}
```

本地动作的常见用途包括：

- **校验**：在提交表单之前校验输入。
- **格式化**：使用 `formatString` 格式化本地显示值。

### 基础 Catalog 的动作校验（Checks）

基础 catalog 定义了一组可在客户端执行的有限检查。交互式组件可以定义 `checks` 列表（使用 [`common_types.json`](../specification/v0_9/json/common_types.json) 中的 `Checkable` schema）。对于 `Button`，如果任一检查失败，按钮会在客户端 **自动禁用**。

- **UX 重点**：动作检查用于管理 **UI 状态（用户体验）**，在无效交互发生之前就将其阻止。它们不能替代 **数据完整性** 检查，后者仍然必须在服务端执行。

这样，UI 就能在用户还没真正提交之前，先强制满足一些要求，比如字段不能为空。

```json
{
  "id": "submit-button",
  "component": "Button",
  "child": "submit-text",
  "checks": [
    {
      "condition": {
        "call": "required",
        "args": { "value": { "path": "/partySize" } }
      },
      "message": "Party size is required"
    }
  ],
  "action": { "event": { "name": "submit_booking" } }
}
```

## 本地状态更新与“写入”契约

在动作真正派发之前，客户端其实已经在本地管理 UI 状态了。A2UI 为所有输入组件（例如 `TextField`、`CheckBox`、`Slider`）定义了一个 **读/写契约（Read/Write Contract）**。

1.  **读（Model → View）**：组件渲染时，会从绑定的 `path` 中读取值。
2.  **写（View → Model）**：一旦用户交互发生（例如输入字符或勾选复选框），客户端会 **立刻** 把新值写回本地数据模型。

这意味着，本地模型始终是 UI 当前状态的真相来源。只有当用户动作（例如按钮点击）被触发时，这个状态才会同步回服务端。

> [!IMPORTANT]
> **同步更新**：本地模型更新是 **同步** 的。这能保证在任何动作解析其 `context` 路径或封装 `DataModelSync` 载荷之前，数据模型都已经完全更新。输入与点击之间不会有竞态条件；“写入”总是先提交。

这种本地优先的方式带来了显著的 **性能收益**。由于同步是即时且本地的，开发者不需要在用户输入 `TextField` 时实现网络防抖，也不用担心延迟抖动。网络在用户准备好正式派发动作之前，完全不会被“UI 噪音”（例如每个按键）打扰。

### 表单提交模式

这种分离方式可以实现一个稳健的表单提交模式：

- `TextField` 绑定到 `/reservationTime`。
- 用户输入 `"7:00 PM"`。`/reservationTime` 的本地模型会立即更新。
- 用户点击 `"Book"` 按钮。按钮的动作会从本地模型中解析 `path: "/reservationTime"`，并把当前值发送给服务端。

## 用户交互流程

当用户与组件交互时（例如点击按钮）：

1.  **解析**：客户端根据本地 **数据模型** 解析 `context` 中的所有 `path` 引用。
2.  **构造**：客户端构建一个符合 [`client_to_server.json`](../specification/v0_9/json/client_to_server.json) 的 `action` 载荷。
3.  **派发**：该载荷通过选定的传输层（例如 A2A、WebSockets）发送出去。

### 示例：动作载荷（v0.9）

如果用户在包含 `{"reservationTime": "7:00 PM", "partySize": 4}` 的数据模型下点击按钮，客户端会使用 `action` 键发送如下消息：

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

> [!IMPORTANT]
> **版本说明（v0.8 vs v0.9）**：在 v0.8 中，顶层载荷键是 `userAction`（例如 `{"userAction": {...}}`）。v0.9 改为上面更简洁的 `action` 键。标准协议解析器会根据载荷中声明的版本，去期待相应的键。

## 智能体处理

智能体（或编排器）接收这个事件并进行处理。在 agentic 系统中，这通常意味着把该事件转成一个隐藏的 LLM 用户查询。

**智能体处理示例（Python）：**

```python
if action_name == "submit_reservation":
    time = context.get("time")
    size = context.get("size")
    # 将其输入给 LLM
    query = f"User submitted a reservation for {size} people at {time}."
    response = await llm.generate(query)
```

## 客户端到服务端错误报告

除了用户动作之外，客户端还可以通过 [`client_to_server.json`](../specification/v0_9/json/client_to_server.json) 中定义的 `error` 载荷，把系统级错误报告回服务端。

### 校验失败

如果智能体发送了违反 catalog schema 或协议规则的 A2UI JSON，客户端会发送一个 `VALIDATION_FAILED` 错误。这是 agentic 系统里非常关键的反馈回路：

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

智能体可以捕获这个错误，进行道歉（或内部自我修正），然后重新发送修正后的 UI。

## 数据模型同步（v0.9）

在 A2UI v0.9 中，我们引入了一个强大的“无状态（stateless）”同步功能。它允许客户端自动把某个 surface 的 **整个数据模型** 包含到其发送给服务端的每条消息元数据中。

### 启用同步

同步是在 surface 初始化阶段由智能体请求的。只要在 `createSurface` 消息中设置 `sendDataModel: true`，智能体就会要求客户端启动同步循环。

### 在线路上同步

启用同步后，客户端不会把数据模型作为单独消息发送，而是把它作为 **metadata** 附加到传出的传输封套中，例如 A2A 消息。

在 A2A（Agent-to-Agent）绑定中，数据模型会放在封套 `metadata` 字段里的 `a2uiClientDataModel` 对象中。

**A2A 带同步的封套示例：**

```json
{
  "parts": [{ "text": "Submit the reservation" }],
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

### 为什么要使用数据模型同步？

- **连接更简单**：你不需要手动把每个输入字段映射到按钮的 `context` 属性。
- **无状态智能体**：智能体无需为每个用户会话维护本地状态；它会在每次交互中拿到完整上下文。
- **口头快捷方式**：允许用户通过语音或文本触发动作，例如“好，提交”，即使没有点具体按钮也行。由于智能体会收到更新后的数据模型以及文本消息，它可以立刻处理请求。

## 客户端元数据与能力

在智能体安全发送 UI 之前，客户端必须先声明它支持哪些 component catalog。这由 `a2uiClientCapabilities` 对象处理。

### 能力声明

客户端会在发给服务端的消息 metadata 中包含 `a2uiClientCapabilities` 对象。

```json
{
  "v0.9": {
    "supportedCatalogIds": [
      "https://a2ui.org/specification/v0_9/basic_catalog.json",
      "https://my-company.com/catalogs/v1/custom.json"
    ],
    "inlineCatalogs": []
  }
}
```

- **`supportedCatalogIds`**：客户端可以渲染的 catalog URI 数组。
- **`inlineCatalogs`**：可选，用于开发或特定环境，允许把完整 catalog schema 直接内联发送。

没有这个握手，智能体就无法确定 renderer 是否真的能处理它发送的特定组件。

## 传输与编码

A2UI 不绑定具体传输层，但最常见的是通过 **A2A（Agent-to-Agent）** 或 WebSockets 使用它。理解载荷如何被封装，对实现非常关键。

### A2A 编码

在标准 A2A 绑定中，A2UI 消息会被编码为一个 A2A **DataPart**。为了把它识别为 A2UI 载荷，part 必须附带特定 metadata：

- **mimeType**: `application/json+a2ui`

DataPart 的 `data` 字段包含一 **列表** A2UI 消息。这允许把多个更新（例如 `createSurface` 后接 `updateComponents`）放在一个网络包中发送。

> [!NOTE]
> **A2A 版本说明**：`data` 字段中使用 **列表** 的方式是在 **A2A v1.0** 中引入的。更早版本的 A2A 协议期望 `data` 字段只包含一个 JSON 对象。

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

## 安全考虑

A2UI 的核心原则之一是安全、沙箱化通信。由于协议依赖通过网络传递用户状态和交互触发条件，因此它会对数据可见性和执行范围施加严格边界。

### 沙箱执行

A2UI 的一个核心卖点是通过限制来换取安全。由于禁止智能体执行任意代码（例如注入原始 JavaScript），A2UI 保证智能体只能触发预先注册的客户端行为。`functionCall` 机制提供了一种安全、沙箱化的方式，让智能体与客户端环境交互，而不会让用户暴露给恶意脚本。

### 数据模型隔离与编排器路由

启用 `sendDataModel: true` 后，客户端会把 surface 的整个数据模型包含到发出的消息中。开发者必须理解这些数据的可见范围：

- **点对点可见性**：只有接收传输封套的后端（创建该 surface 的智能体，或中间编排器）能够读取该载荷。
- **编排器的责任**：在多智能体架构中，中心编排器通常会把用户意图路由给专门的子智能体。编排器必须强制执行 **数据隔离**。它要负责解析 `a2uiClientDataModel`，识别 `surfaceId`，并确保数据模型只被传给拥有该 surface 的特定子智能体。一个智能体 surface 的数据绝不能泄漏给另一个智能体。

## 编排与路由

在多智能体系统中，中心 **Orchestrator** 往往负责管理用户与多个专门子智能体之间的交互。一个关键挑战是：如何确保来自客户端的 `action` 消息被路由回生成该 UI surface 的那个特定子智能体。

### Surface 所有权模式

为了解决这个问题，编排器必须维护 `surfaceId` 到其所有者子智能体的映射。这通常存储在 **Session State** 中。

#### 1. 映射所有权

当某个子智能体发出 `createSurface` 消息时，编排器会拦截它并记录所有权。

```python
# Simplified Orchestrator Logic: Record Ownership
def on_surface_created(surface_id, agent_name, session):
    # Store the mapping in the orchestrator's session state
    session.state.update({f"owner_of_{surface_id}": agent_name})
```

#### 2. 路由用户动作

当客户端把 `action` 发回编排器时，编排器会查找 `surfaceId`，并把请求转交给正确的子智能体。
